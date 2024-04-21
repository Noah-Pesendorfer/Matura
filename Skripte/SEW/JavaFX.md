# JavaFX
## Keyfacts: Zu beachten

- **M**odel, **V**iew, **C**ontroller Klassen erstellen
- View.fxml erstellen, SceneBuilder starten und View.fxml Datei im SceneBuilder öffnen
-

## Tips von Helml zur Matura
- Meistens JavaFX Anwendung (**Konsolenanwendung sehr selten**)
- Schnittstelle zu Datenbank (**Immer HSQLDB**)
- HSQLDB **Treiber einbinden** und alles was dazu gehört
- APIs sind immer vollständig da (Auf **D-Laufwerk**)
- Viel **Sourcecode** im D-Laufwerk
- **Datenbanken** Abfragen
- **Bindings und Nebenläufigkeit** (Concurrency)
  - Progressbar
  - Hintergrundaktivitäten
  - Concurrency auch im Zusammenhang mit JavaFX
- Producer/Consumer noch nie gekommen
- **Client/Server** oft gekommen
- **CSV-Datei** auslesen
- **Collections** immer
- **Binärdateien** immer
- Komplette Synchronized Methode? -> Schwerer Fehler
- Bei **HSQLDB immer SingleUser**
- **Prepared Statements**

## SceneBuilder mit IntelliJ verknüpfen
- IDE Settings öffnen (Strg+Alt+S)
- → Languages & Framework
- → JavaFX: Pfad zum SceneBuilder angeben
  - Der Pfad soll die ausführbare Datei sein.


## Properties und Bindings
+ Benutzeroberfläche
  + stellt Zustand von Datenobjekten dar
  + gibt Benutzer Möglichkeit, den Zustand zu ändern
+ Bsp.: Schieberegler für Breite von Rechteck
  + Werte auslesen
  + width des Models aktualisieren
  + Berechnung Rechtecksfläche
  + zeichnen

Einfaches Beispiel:
```java
public class MyBean {
   private StringProperty sample = new SimpleStringProperty();
   public String getSample() {
      return sample.get();     
   } 
   
   public void setSample(String value) {
      sample.set(value);
   } 
   
   public StringProperty sampleProperty() {
      return sample;
   } 
}
```
### Einfache Properties (abstrakte Klassen):
- BooleanProperty
- DoubleProperty
- FloatProperty
- IntegerProperty
- LongProperty
- StringProperty

Erzeugen von Properties über konkrete Klassen (maximale Parameter):
```java
BooleanProperty booleanProperty = new SimpleBooleanProperty(true, „b″, this);
DoubleProperty doubleProperty = new SimpleDoubleProperty(1.5, „d″, this);
FloatProperty floatProperty = new SimpleFloatProperty(1.5f, „f″, this);
IntegerProperty integerProperty = new SimpleIntegerProperty(123, „i″, this);
LongProperty longProperty = new SimpleLongProperty(1234567899l, „l″, this);
StringProperty stringProperty = new SimpleStringProperty("hallo", „s″, this);
```

### Object Properties
- speichern beliebige Objekte

```java
ObjectProperty<Image> objectProperty = new SimpleObjectProperty<>();
```

### Bindings
- verknüpfen Properties mit Werten
```java
label.textProperty().bind(myBean.sampleProperty());
```
- Binding lösen
```java
label.textProperty().unbind();
```

- Bindings werden **im Controller** erstellt
- 2 Möglichkeiten:
  - `Interface Initializable`
    - ```java
      public class Controller implements Initializable {
      
        @Override 
        public void initialize(URL location, ResourceBundle resources) { 
            // Bindings here 
        }
      }
      ```
  - `@FXML initialize`
    - ```java
      public class Controller {
         @FXML public void initialize() {
             // Bindings here 
         } 
      }
      ```
Übung: [1simpleBinding](..%2F..%2F%C3%9Cbungen%2FSEW%2FJavaFX%2F1simpleBinding)

#### Calculated Bindings
- High-Level-API
  - Bindings-Klasse
    - ```java
      DoubleProperty number1 = new SimpleDoubleProperty(1);
      DoubleProperty number2 = new SimpleDoubleProperty(2); 
      DoubleProperty number3 = new SimpleDoubleProperty(3); 
      
      NumberBinding calculated = Bindings.add( number1, Bindings.multiply(number2,number3));
      ```
  - Fluent-API
    - ```java
      DoubleProperty number1 = new SimpleDoubleProperty(1); 
      DoubleProperty number2 = new SimpleDoubleProperty(2); 
      DoubleProperty number3 = new SimpleDoubleProperty(3); 
      
      NumberBinding calculated = number1.add(number2.multiply(number3));
      ```
- Low-Level-API
  - ```java
    DoubleProperty number1 = new SimpleDoubleProperty(1); 
    DoubleProperty number2 = new SimpleDoubleProperty(2); 
    DoubleProperty number3 = new SimpleDoubleProperty(3); 
    
    NumberBinding calculated = new DoubleBinding() { 
        {      
            super.bind(number1, number2, number3);
        }
       @Override
       protected double computeValue() {
         return number1.get() + (number2.get() * number3.get()); 
        } 
    };
    ```

Berechning mit numerischen Bindings:
- `.add` / `.substract`
- `.multiply` / `.divide`
- `.negate`
- `.min` / `.max`

#### Bidirektionale Bindings
- 2 Properties gegenseitig gebunden
```java
DoubleProperty number1 = new SimpleDoubleProperty(1);
DoubleProperty number2 = new SimpleDoubleProperty(2); 

number1.bindBidirectional(number2);
```
- Dann können auch Properties gesetzt werden:
```java
number2.setValue(3); 
number1.setValue(4); 
System.out.println(„number2 hat Wert: „ + number2.getValue()); // Wert 4
```

Übung: [2cylinder](..%2F..%2F%C3%9Cbungen%2FSEW%2FJavaFX%2F2cylinder)

#### Object Bindings
- Beliebige Objekte an Properties binden
- Vorgehensweise:
  - eigene Klasse ableiten von `ObjectBinding<T>`
  - im Konstruktor passendes Property annehmen (muss nicht Typ T sein)
  - Property als Attribut speichern
  - `T computeValue()` Methode implementieren, die Objekt returnt, welches an Property gebunden wurde

Übung: [5instantViewer1](..%2F..%2F%C3%9Cbungen%2FSEW%2FJavaFX%2F5instantViewer1)

#### Boolean Bindings
- Logische Verknüpfung von Boolean Propertys


- .greaterThan / .greaterThanOrEqualTo
- .isEqualTo / .isNotEqualTo
- .lessThan / .lessThanOrEqualTo
- .and / .or
- isNotEmpty
- ...


Beispiel: In einem Textfield überprüfen, ob Eingabe mind. 3 Zeichen:
```java
BooleanBinding textFieldEntered = 
    textField.textProperty() 
    .isNotEmpty() 
    .and(textField.textProperty().length().greaterThan(3));
```
Button deaktivieren, wenn Textfield leer ist:
```java
button.disableProperty().bind(textFieldEntered.not());
```

Übung: [6bikesProps1Serial4](..%2F..%2F%C3%9Cbungen%2FSEW%2FJavaFX%2F6bikesProps1Serial4)

## Serialisierung von Properties
- Properties können NICHT serialisiert werden
  - daher mit transient kennzeichenen
  - werden trotzdem nicht gespeichert!

```java
public class Sample implements Serializable {
    private transient StringProperty sample = new SimpleStringProperty(); 
    
    public String getSample() {
        return sample.get();     
    } 
    
    public void setSample(String value) {         
        sample.set(value);     
    } 
    
    public StringProperty sampleProperty() { 
        return sample;     
    } 
}
```

## Erzeugen + Befüllen einer ListView
```java
// Definiere ListProperty -> es sollen Strings gespeichert werden 
ListProperty<String> listProperty = new SimpleListProperty<>(); 

// Erstelle Collection mit Inhalt 
List<String> iosList = new ArrayList<>(); 

iosList.add("iPhone 4"); 
iosList.add("iPhone 5"); 
iosList.add("iPhone 6"); 
iosList.add("iPhone SE"); 
iosList.add("iPhone 8"); 

// setzen der Werte durch umwandeln einer ArrayList in eine observableList 
listProperty.set(FXCollections.observableArrayList(iosList)); 

// binde die ListView Items an die ListProperty 
myListView.itemsProperty().bind(listProperty); 

// ändere Elemente in der listProperty 
listProperty.add("iPhone11"); 
listProperty.remove(0);
```
- Ausgewähltes Item: `myListView.getSelectionModel().getSelectedItem()`
- Property des Items (für Bindings): `myListView.getSelectionModel().selectedItemProperty()`

Übung: [4formPersList4](..%2F..%2F%C3%9Cbungen%2FSEW%2FJavaFX%2F4formPersList4)

## Concurrency
- JavaFX Thread darf **niemals** blockiert werden
  -  -> eingeforene Anwendung
  -  -> Ereignisbehandlung reagiert nicht mehr
- rechenintensive Tätigkeiten -> Threads auslagern
- für JavaFX gibt es eigene Klassen/Interfaces:


- `Interface Worker`
    - Interface (Basis für alle Concurrent Klassen)
    - Worker Objekte erledigen Arbeiten im Hintergrund-Thread
  - Worker
    - Zustand des Threads kann abgefragt werden:
      - `Worker.State.READY`
      - `Worker.State.SCHEDULED`
      - `Worker.State.RUNNING`
      - `Worker.State.CANCELLED`
      - `Worker.State.SUCCEEDED`
      - `Worker.State.FAILED`
    - Fortschritt über Properties beobachtbar:
      - `totalWork`: DoubleProperty, Maximalwert für die Arbeit
      - `workDone`: DoubleProperty, Anteil an erledigter Arbeit
      - `progress`: Wert zw. 0, 1 
- Klasse Task
  - implementiert `Worker`-Interface
  - einmalige Hintergrundberechnungen
  - Task impl. Runnable -> Start über Executor möglich
  - Ergebnis des Tasks mit Methode `getValue()`
    - Ergebnis noch nicht Fertig: `getValue()` blockiert
  - in `call()`-Methode wird arbeit verrichtet und ggf. Properties aktualisiert
  - Fortschritt aktualiseren:
    - in `call()` wird Methode `updateProgress()` aufgerufen
    - Mit `task.progressProperty()` Binding auf z.B. Progressbar möglich
  - Task unterbrechen:
    - im Controller mit: `myTask.cancel()`
    - bei Tasks: `isCancelled()` prüfen
  - Aktionen nach beendigung von Tasks:
    - ```java
        task.setOnSucceeded((WorkerStateEvent event) -> {
         Object value = task.getValue();
           // do anything with the result
         updateTheUI(value); 
        }); 
        // setOnFailed 
        // setOnScheduled 
        // setOnCanceled
        // setOnRunning
      ```