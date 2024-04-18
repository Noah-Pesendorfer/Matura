# JDBC
- **J**ava **D**ata**B**ase **C**onnectivity
- Schnittstelle für Zu
- 
- griff auf relationale DB
  - Mittels SQL und Java
## Grundgerüst
### Verbindungsaufbau
- externes **Property File für Verbindungsdaten**
  - `dbconnect.properties`

`dbconnect.properties`:
```
driver=org.hsqldb.jdbcDriver
url=jdbc:hsqldb:file:../../../db/jdbctryDb/
username=app
password=app
```
Laden der Eigenschaften aus Property-File:
```
try (FileInputStream in = new FileInputStream("dbconnect.properties");) {
  Properties prop = new Properties();
  // Properties laden prop.load(in);
  
  String driver = prop.getProperty("driver");
  String url = prop.getProperty("url"); 
  String user = prop.getProperty("user"); 
  String pwd = prop.getProperty("password"); 
} 
catch ...
```
### Datenbankverbindung aufbauen
```
Connection con = DriverManager.getConnection(url, user, pwd);
```
- url, user und pwd von ausgelesenem Property-File benutzen
- Zugriff auf DB erfolgt über Objekt `con`

Übung:
[1jdbctryEnhanced](..%2F..%2F%C3%9Cbungen%2FSEW%2FJDBC%2F1jdbctryEnhanced)

### HSQDLDB in IntelliJ einbinden
- Projekt anlegen
- -> Project Structure
- -> Settings - Libraries
- -> +
- -> hsqldb.jar einbinden

### Datentypen
![Datentypen.png](..%2FZ_Images%2FDatentypen.png)

Übung: [bikesProps3Base3](..%2F..%2F%C3%9Cbungen%2FSEW%2FJDBC%2FbikesProps3Base3)

### Datenbankänderungen
- Änderungen der DB-Struktur
  - z.B. `CREATE TABLE`
- Änderungsbefehle
  - z.B. `INSERT`, `UPDATE`, `DELETE`
  - Rückgabewert: Anzahl geänderter Datensätze
```
int executeUpdate(String sql) throws SQLException
```

### Statements
- Schnittstelle:
  - `java.sql.Statements`
- Statement Objekt erzeugen:
  - `Statement stmt = con.createStatement();`
- SQL-Anweisung ausführen
  - `ResultSet rs = stmt.executeQuery("SELECT * FROM CUSTOMER);`
- Ergebnis der SQL-Abfrage: `ResultSet`
  - = Ergebniscursor (=Position in Ergebnismenge)
    - Zeilenwiese vorwärts mittels `.next()`
    - Anfang: `.first()`
    - Ende: `.last()`
  - Spaltenzugriff über `getXXX()` Methoden
    - Spaltenindex bzw. Spaltenname als Parameter
    - Beispiel:
      - `byte getByte(...)`
      - `int getInt()`
      - `short getShort()`

Beispiel 1:
```
ResultSet rs = stmt.executeQuery( "SELECT * FROM CUSTOMER" );
while ( rs.next() )
{
  System.out.printf( "%s, %s, %s, %s, %s \n",
    rs.getString(1),
    rs.getString(2),
    rs.getString(3),
    rs.getString(4),
    rs.getString(5)
  );
}
```

Beispiel 2:
```
rs = stmt.executeQuery( "SELECT LASTNAME,
                      CITY FROM CUSTOMER);
while( rs.next() )
{
  System.out.printf( "%s, %s \n",
      rs.getString(1),
      rs.getString(2)
  );
}
```

Beispiel 3:
```
rs = stmt.executeQuery( "SELECT LASTNAME,
                      CITY FROM CUSTOMER);
while( rs.next() )
{
  System.out.printf( "%s \n",
      rs.getString("LASTNAME" )
  );
}
```

mit `close()` schließen ( try-with-resources )

### Prepared Statements
- Beispiel: Suchanfrage
  - Immer gleiche SQL-Anweisung im Hintergrund
  - lediglich Parameter ändert sich
- Lösung: Beschleunigung durch PreparedStatements
  - DB parst SQL-Statement
  - Statement wird vorkompiliert
  - nur mehr Parameter wird ausgetauscht
- `PreparedStatement prepareStatement(String sql)`
- Parameter löschen:`clearParameters()`
- Ausführung:
  - `ResultSet executeQuery() throws SQLException`
  - `int executeUpdate() throws SQLException`

Übung: [bikesProps4Prepared3](..%2F..%2F%C3%9Cbungen%2FSEW%2FJDBC%2FbikesProps4Prepared3)

### BLOBs
- **B**inary **L**arge **OB**jects
  - Streams zum Lesen und Schreiben von Daten

Blob in DB speichern:
```
void setBinaryStream (int parameterIndex,
                      InputStream x,
                      int length) throws SQLException
             
// parameterIndex ... Index des Platzhalters im SQL-String
// x ... InputStream (Lesen vom Datenträger: FileInputStream)
// length ... Größe der Datei in Byte
```

Blob aus DB lesen:
```
InputStream getBinaryStream
(int columnIndex) throws SQLException

InputStream getBinaryStream
(String columnName) throws SQLException
```