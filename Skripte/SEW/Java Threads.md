# Threads
## Theorie
- **Jeder Thread** muss Interface `Runnable` implementieren.
- Methode `run()` steht Code, welcher im Thread ausgeführt wird.
## Thread ausführen
### Variante 1
**`Runnable` Interface implementieren**
```
class TimePrinter implements Runnable{
@Override
public void run() {
    for (int i = 0; i < 20; i++)
    System.out.println("Time: " + System.currentTimeMillis());
    }
}
```
1. Objekt in Klasse Thread erzeugen (`Runnable` als Parameter):
```
Thread t1 = new Thread(new TimePrinter());
```
2. Methode `start()` aufrufen (Bei `run()`**keine** Nebenläufigkeit):
```
t1.start();
```
### Variante 2
**Von Thread Klasse ableiten**

Thread Klasse implementiert selbst `Runnable`
```
class TimePrinter extends Thread{
@Override
public void run() {
    for (int i = 0; i < 20; i++)
        System.out.println("Time: " + System.currentTimeMillis());
    }
}
```
```
new TimePrinter.start();
```
### Thread im Konstruktor ausführen
```
class TimePrinter extends Thread{

    public TimePrinter() {
        this.start();
    }
    
    @Override
    public void run() {
        ...
    }
}
...
new TimePrinter ();
```
## Eigenschaften und Zustände von Threads
### Eigenschaften:
- Zustand
- Priorität
- Namen

### Zugriff über Methode `currentThread()`:
```
System.out.println(Thread.currentThread().getPriority());
```

### Threads schlafen lassen
- Nur bei **aktuellem Thread** - nicht Fremde
- Methode `Thread.sleep()`

```
try {
    Thread.sleep( 2000 );
} catch ( InterruptedException e ) { }
```

### Auf Rechenzeit verzichten
```
Thread.yield();

/*
Ich setze diese Runde aus und mache weiter, wenn ich das
nächste Mal dran bin.
*/
```

### Threads als `Daemons`
- `Daemon`-Threads dienen den Benutzer-Threads, indem sie **Hintergrundaufgaben** erledigen, und haben keine andere Rolle als die **Unterstützung der Hauptausführung**.
- Wird von JVM (Java-Virtual-Machine) beendet, wenn **alle** Haupt-Threads beendet sind
```
class DaemonThread extends Thread {

    @Override
    public void run() {
        while(true)
        System.out.println( "Lauf, Thread, lauf" );
    }
    
    public static void main( String[] args ) {
        new DaemonThread().start();
    }
    
    public DaemonThread() {
        this.setDaemon(true);
    }
}
```

## Das Ende eines Threads
- `run()`-Methode wurde ohne Fehler beendet
- `RuntimeException` in der `run()`-Methode
- Abgebrochen durch `stop()`. 
  - Achtung: **Nicht verwenden**, veraltet, verwendet man nicht!

### Thread um Abbruch bitten
- `interrupt()` Setzt einem Thread ein internes Flag
- in der `run()`-Methode mittels `isInterrupted()` abfragen, ob enden soll
- `run()`-(Schleife) wird verlassen, wenn `isInterruped() = true`

### Schlafenden Thread um Abbruch bitten
- Wenn Thread im Zustand `sleep()` ist
  - und `interrupt()` aufgerufen wird
  - --> `InterruptedException` und Flag bleibt auf `false`
- (Erneuter) Aufruf im `catch`-Block **zwingend notwendig**!

## Executor
- Löst starke Bindung zw. Thread und `Runnable`
- --> Mehr Flexibilität
- Implementiert Methode: `void execute(Runnable command)`

### Konkrete Implementierungen:
- `ThreadPoolExecutor`
  - Sammlung von Threads - Ausführung übernimmt beliebiger, freier Thread
- `ScheduledThreadPoolExecutor`
  - Ausführung von Threads zu bestimmten Zeiten oder Wiederholungen

Konstruktoren der beiden Klassen nicht trivial
- --> Hilfsklasse `Executors`
```
ExecutorService es =
Executors.newCachedThreadPool();

ScheduledExecutorService ses =
Executors.newScheduledThreadPool(5);
```

### Beispiel:
```
Runnable r1 = new Runnable() {
    @Override public void run() {
        System.out.println( "A1 " + Thread.currentThread() );
        System.out.println( "A2 " + Thread.currentThread() );
    }
};

Runnable r2 = new Runnable() {
    @Override public void run() {
        System.out.println( "B1 " + Thread.currentThread() );
        System.out.println( "B2 " + Thread.currentThread() );
    }
};

ExecutorService executor = Executors.newCachedThreadPool();
executor.execute( r1 );
executor.execute( r2 );
Thread.sleep( 500 );
executor.execute( r1 );
executor.execute( r2 );
executor.shutdown();
```

```
ScheduledExecutorService executor = Executors.newScheduledThreadPool(5);
executor.schedule(r1, 10, TimeUnit.SECONDS);
executor.execute( r2 );
executor.shutdown();
```

### Wichtige Methoden
- `shutdown()`
  - Fährt Thread-Pool herunter. 
  - Laufende Threads werden nicht abgebrochen
  - Keine neuen werden gestartet
  - Rückgabe: void
- `isShutdown()`
  - Wurde der Executor schon heruntergefahren?
  - Rückgabe: `true` - `false`
- `shutdownNow()`
  - Momentan ausführende Aufgaben werden versucht zu stoppen
  - Rückgabe: Liste der zu beendenden Threads

## Synchronisation über kritische Abschnitte
- Grundsätzlich verwalten Threads ihre eigenen Daten (lokale Daten)
- Können sich auch Daten "teilen"
  - über statische Variablen/Klassen
  - über Objektreferenzen, von mehreren Threads gleichzeitig genutzt
- Problem: Scheduler schaltet zu unbekannten Zeitpunkten zwischen Ausführung der Threads um.


**Das bedeutet:**
-  Leseoperationen von gemeinsamen Daten: unproblematisch!
-  **Schreibeoperationen**: problematisch!
  - "kritische Abschnitte"
- Wenn man "kritische Abschnitte" **nicht sauber** löst, ist Programm **nicht "thread-safe"**

## Java Konstrukte zum Schutz der kritischen Abschnitte
- Sicherstellen, dass nur **EIN** Thread den kritischen Abschnitt bearbeitet
- Konzept: **`synchronized`**
```
lock.lock();
{
  statement1
  statement2
}
lock.unlock();
```

### ReentrantLock
- Objekt der Klasse `ReentrantLock` kümmert sich um Zugriffsberechtigungen in dem kritischen Abschnitt

```
Lock lock = new ReentrantLock(); // Objekt der Klasse ReentrantLock
...
lock.lock();
...
lock.unlock();
```

- Dasselbe Objekt der `ReentrantLock`-Klasse muss geteilt werden
```
Lock lock = new ReentrantLock();
...
RandomPoint rp1 = new RandomPoint(p, lock);
RandomPoint rp2 = new RandomPoint(p, lock);
...
```
Warum nicht alle Methoden synchronisieren (mit Locks versehen)?
- Je mehr gesperrter Code --> schlechtere Performance
- Wahrscheinlichkeit für Deadlocks steigt.

## Deadlock
- 2 Threads sperren sich gegenseitig
- Beispiel:
```
class T1 extends Thread {
  @Override
  public void run() {
    lock1.lock();
    System.out.println( "T1:Got lock1!" );
    try { TimeUnit.SECONDS.sleep( 1 ); } catch ( InterruptedException e ) { }
    lock2.lock();
    System.out.println( "T1:Got lock2!" );
    lock2.unlock();
    lock1.unlock();
  }
}
class T2 extends Thread {
  @Override
  public void run() {
    lock2.lock();
    System.out.println( "T2:Got lock2!" );
    lock1.lock();
    System.out.println( "T2:Got lock1!" );
    lock1.unlock();
    lock2.unlock();
  }
}
```
Beschreibung: In diesem Fall blockiert T1 die Freigabe (den `unlock`) von `lock1`, während T2 darauf wartet, `lock2` zu erhalten. Gleichzeitig blockiert T2 die Freigabe von `lock2`, während T1 darauf wartet, `lock2` zu erhalten. 

## Lock Freigabe im Fall von Exceptions
_Sperre bleibt bestehen_
```
ReentrantLock lock = new ReentrantLock();
try {
  lock.lock();
  //critical code
  System.out.println(lock.getHoldCount()); // 1
  System.out.println(12 / 0);             // Exception-Auslöser
  lock.unlock();
}
catch (Exception e) {
  System.out.println(e.getMessage()); // divided by zero
}
System.out.println(lock.getHoldCount()); // 1!
```
Lock freigeben mit `finally`:
```
ReentrantLock lock = new ReentrantLock();
try {
  lock.lock();
  //critical code
  System.out.println(lock.getHoldCount()); // 1
  System.out.println(12 / 0);
}
catch (Exception e) {
  System.out.println(e.getMessage()); // / by zero
}
finally {
  lock.unlock();
}
System.out.println(lock.getHoldCount()); // 0
```
Finally wird immer am Ende eines try-catch Blocks ausgeführt, egal ob eine Exception geworfen wurde oder nicht.



