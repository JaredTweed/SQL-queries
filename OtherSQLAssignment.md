# SQL Project

## Airport Schema

The schema for the airport database is as follows. Primary key attributes are underlined and foreign keys are noted in superscript.

- *Airport* = {*iata*, airportName, city}
- *Flight* = {*flightCode*, distance, departureIata<sup>Airport</sup>, arrivalIata<sup>Airport</sup>}
- *FlightInstance* = {*flightCode<sup>Flight</sup>*, *departs*, gate, aircraftID}
- *Passenger* = {*passengerID*, firstName, lastName, miles}
- *Flies* = {*flightCode<sup>FlightInstance</sup>*, *departs<sup>FlightInstance</sup>*, *passengerID<sup>Passenger</sup>*}

#### Attribute Descriptions

- *iata* – Three-character code that identifies an airport.
- *distance* – Distance between the departure and arrival airports.
- *departs* – Departure date.
- *passengerID* – Integer that identifies a passenger.
- *miles* – Total distance traveled by a passenger on all flights.

#### Attribute Domains

- *iata, departureIata, arrivalIata, gate* – Fixed length character string of length 3.
- *flightCode* – Fixed length character string of length 6.
- *airportName, city, firstName, lastName* – Variable length character string of length 30.
- *distance, passengerID, miles* - Integer.
- *aircraftID* - Tinyint.
- *departs* - Date.

#### Primary Key Constraints

- *Airport* – iata.
- *Flight* – flightCode.
- *FlightInstance* – flightCode, departs.
- *Passenger* – passengerID.
- *Flies* - flightCode, departs, passengerID.

#### Foreign Key Constraints

##### Flight
- *departureIata, arrivalIata* (references Airport), prevent deletion or update of referenced records.

##### FlightInstance
- *flightCode* (references Flight), cascade deletion and updates of referenced records.

##### Flies
- *flightCode, departs* (references FlightInstance), prevent deletion of referenced records, but cascade updates.
- *passengerID* (references Passenger), prevent deletion or update of referenced records.

#### Other Constraints

##### Airport
- *airportName* and *city* should not be null.
- *airportName* is a candidate key.

##### Flight
- *departureIata* and *arrivalIata* should not be null.
- *departureIata* and *arrivalIata* must be different.

##### Passenger
- *firstName* and *lastName* should not be null.

##### Flight Instance
- *aircraftID* should only appear in one flight instance per day.

### My Code

```sql
-- Create Airport table
CREATE TABLE Airport (
  iata CHAR(3) PRIMARY KEY,
  airportName VARCHAR(30) NOT NULL,
  city VARCHAR(30) NOT NULL,
  UNIQUE (airportName)
);

-- Create Flight table
CREATE TABLE Flight (
  flightCode CHAR(6) PRIMARY KEY,
  distance INTEGER,
  departureIata CHAR(3) NOT NULL,
  arrivalIata CHAR(3) NOT NULL,
  FOREIGN KEY (departureIata) REFERENCES Airport(iata) ON DELETE NO ACTION ON UPDATE NO ACTION,
  FOREIGN KEY (arrivalIata) REFERENCES Airport(iata) ON DELETE NO ACTION ON UPDATE NO ACTION,
  CHECK (departureIata <> arrivalIata)
);

-- Create FlightInstance table
CREATE TABLE FlightInstance (
  flightCode CHAR(6),
  departs DATE,
  gate VARCHAR(3),
  aircraftID TINYINT,
  PRIMARY KEY (flightCode, departs),
  FOREIGN KEY (flightCode) REFERENCES Flight(flightCode) ON DELETE CASCADE ON UPDATE CASCADE,
  CHECK (aircraftID NOT IN (SELECT aircraftID FROM FlightInstance WHERE departs = FlightInstance.departs AND flightCode <> FlightInstance.flightCode))
);

-- Create Passenger table
CREATE TABLE Passenger (
  passengerID INTEGER PRIMARY KEY,
  firstName VARCHAR(30) NOT NULL,
  lastName VARCHAR(30) NOT NULL,
  miles INTEGER
);

-- Create Flies table
CREATE TABLE Flies (
  flightCode CHAR(6),
  departs DATE,
  passengerID INTEGER,
  PRIMARY KEY (flightCode, departs, passengerID),
  FOREIGN KEY (flightCode, departs) REFERENCES FlightInstance(flightCode, departs) ON DELETE NO ACTION ON UPDATE CASCADE,
  FOREIGN KEY (passengerID) REFERENCES Passenger(passengerID) ON DELETE NO ACTION ON UPDATE NO ACTION
);
```

## Triggers

You should create two triggers. Make sure that your triggers work correctly when multiple records are affected.

### Deleted Passenger Trigger

Create a trigger that prints the number of *Flies* records associated with any passengers that are attempted to be deleted, as an error message. The behavior of the trigger is:

1. If the passenger is not associated with any *Flies* records no value should be printed and the passenger should be deleted as normal.
2. If the passenger is associated with at least one *Flies* record the total number of such records should be printed, and the passenger should not be deleted.
3. If multiple passengers are deleted as one operation and at least one of the passengers is associated with a *Flies* record then (a) the total number of *Flies* records for all the deleted passengers should be printed and (b) none of the passengers should be deleted, including those with no *Flies* records.

Hint – an after trigger is not going to work …

#### My Code

```sql
CREATE TRIGGER tr_deleted_passenger
ON Passenger
BEFORE DELETE 
AS
BEGIN
    IF EXISTS(SELECT * FROM Flies WHERE Flies.passengerID = old.passengerID)
    --print total number of records
    --do not delete passenger
   IF EXISTS(SELECT COUNT(*) INTO num_flies FROM Flies WHERE passengerID = OLD.passengerID AND num_flies >= 2)
END
```

### Miles Update Trigger

Create a trigger that correctly updates the *miles* attribute of the passenger table when records of the *Flies* table are inserted, deleted or updated. The *miles* attribute should store the total number of miles traveled by a passenger. The trigger(s) must use the contents of the inserted and deleted tables and you are not permitted to solve this by recalculating the values of the *miles* attribute of all passengers.

## Stored Procedures

In addition to the above you should write the following three stored procedures, which run SQL queries, with variables.

The basic syntax to create a store procedure is:

```sql
CREATE PROCEDURE test
AS
BEGIN
select * from song
END
```

You can then run it by calling *EXEC test*.

### Flights to and from Airports

This stored procedure must be named *spFlightsToFrom*.

Return *flight code*, *departure date* and *gate* of flights that depart from and arrive at particular airports, where the two airport's IATAs are variables passed to the procedure - departure IATA first and arrival IATA second. The result should be sorted by *flight code* and *departure date* (in ascending order).

#### My Code

```sql
CREATE PROCEDURE spFlightsToFrom
  @departureIata CHAR(3),
  @arrivalIata CHAR(3)
AS
BEGIN
  SELECT fi.flightCode, fi.departs, fi.gate
  FROM FlightInstance AS fi
    INNER JOIN Flight AS f ON fi.flightCode = f.flightCode
  WHERE f.departureIata = @departureIata AND f.arrivalIata = @arrivalIata
  ORDER BY fi.flightCode ASC, fi.departs ASC
END
```

### Select Passengers

#### My Code
```sql
CREATE PROCEDURE spPassenger
  @lastName VARCHAR(30)
AS
BEGIN
  SELECT DISTINCT p.passengerID, p.firstName, p.lastName, a.airportName
  FROM Passenger AS p
    INNER JOIN Flies AS fl ON p.passengerID = fl.passengerID
    INNER JOIN Flight AS f ON fl.flightCode = f.flightCode
    INNER JOIN Airport AS a ON f.arrivalIata = a.iata OR f.departureIata = a.iata
  WHERE p.lastName LIKE '%' + @lastName + '%'
  ORDER BY p.passengerID ASC
END
```

This stored procedure must be named *spPassenger*.

Return *passengerID, first name, last name* and *airport names* of passengers whose last name contains a string that is passed to the procedure as a variable. The *airport name* column should contain the names (not IATAs) of airports the passenger has either arrived at or departed from. Note that the name criterion, and SQL LIKE, are not case sensitive.

### Airport Statistics

This stored procedure must be named *spAirport* and returns statistics relating to the airport identified by the airport IATA passed to the procedure as a variable. The result should have a single row with four, named, columns.

Return *count of flight instances departed* from the airport, *count of passengers departed* from the airport, *count of flight instances arrived* at the airport and *count of passengers arrived* at the airport.

