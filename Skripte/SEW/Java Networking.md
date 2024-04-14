# Kommunikation über Sockets
- Socket - Endpunkt einer bidirektionalen Kommunikationsverbindung zwischen zwei Programmen
- Client-Socket: initiiert Verbindung und sendet Daten
- Server-Socket: lauscht auf eingehende Verbindungen

## Client - Verbindung zum Server
- Klasse `java.net.Socket`
- Konstruktor `Socket(String host, int port)`

Verbindung zu Port 2000:
```
Socket serverSocket = new Socket("localhost", 2000);
```
## Datenübertragung mit Streams
- Sobald die Verbindung aufrecht ist, geht man mit Daten genauso um, als wenn man Dateien schrieben/lesen würde.
-  `Socket`-Klasse bietet `getInputStream()`,`getOutputStream()
  - Damit Reader- und Writer-Objekte erzeugen
## Verbindung beenden
- Methode `close()` initiiert Ende der Verbindung und legt die Ports frei
- Wird meistens mit `try with resources` oder `finally`verwendet

## Server
- Server auf Port 2000 aktivieren
  - `ServerSocket serverSocket = new ServerSocket(2000);`
  - _Jetzt können sich die Clients verbinden_
- Server akzeptiert Verbindung
  - `Socket client = serverSocket.accept();`
  - _Schreiben/Lesen zwischen Client/Server ist jetzt möglich._

Server:
```
Socket clientSocket = null;

try(ServerSocket serverSocket = new ServerSocket(2000)){
    clientSocket = serverSocket.accept();
    try(PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);
        Scanner in = new Scanner(clientSocket.getInputStream())) {
    
        //TODO communication
        
    }
}
catch (IOException ex) {
    System.out.println("Server: Communication Error!");
}
finally {
    if (clientSocket != null)
        try { clientSocket.close(); } catch ( IOException e ) { }
}
```
Client:
```
try( Socket serverSocket = new Socket("localhost", 2000);
    Scanner in = new Scanner(serverSocket.getInputStream());
    PrintWriter out = new PrintWriter(serverSocket.getOutputStream(), true)) {
     
    System.out.println("Client: Verbindung zu Server wurde hergestellt");
    
    //TODO communication
    
} catch (IOException e) {
    System.out.println("Client: Kommunikationsfehler!");
}
```

# Multithreading
- Server ist in der Lage, mit mehreren Clients zu kommunizieren
- Implementierung mit `ThreadPool`

Client:
```
ExecutorService executorService = Executors.newCachedThreadPool();
try( ServerSocket serverSocket = new ServerSocket(2000) ){
    while(true) {
        Socket clientSocket = serverSocket.accept();
    executorService.execute(new ClientHandler(clientSocket));
    }
}
catch(...) {
    ...
}
```
Server:
```
public class ClientHandler implements Runnable {
    public ClientHandler(Socket clientSocket, ...) {
        ...
    }
    @Override
    public void run() {
        //Communication is done here!
        ...
    }
}
```
# Streams - `DataInputStream`- `DataOutputStream`
- Wird verwendet, um primitive Datentypen zu senden/empfangen
```
...
DataInputStream in = new DataInputStream(socket.getInputStream());
DataOutputStream out = new DataOutputStream(socket.getOutputStream());
...
out.writeDouble(1.0);
double v = in.readDouble();
...
```

# ObjectStreams - `ObjectInputStream` - `ObjectOutputStream`
- Wird verwendet, um Objekte der eigenen Klasse zu senden/empfangen
```
...
ObjectInputStream in = new ObjectInputStream(socket.getInputStream());
ObjectOutputStream out = new ObjectOutputStream(socket.getOutputStream());
...
Request req = new Request();
out.writeObject(req);
Response response = (Response) ois.readObject();
...
```

- Da Server und Client übergebene Klasse kennen müssen, erstellt man package `common`, welche die Klassen hält.
- Nun kann man `Request` und `Response`- Klassen erstellen
- 
