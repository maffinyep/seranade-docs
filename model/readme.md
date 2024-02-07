# Data Model

```mermaid
erDiagram
    SensitiveData {
        str DateTime
        obj Values
    }
    Patient {
        str CodiceFiscale
        str Name
        str Surname
        str Address
        str PhoneNo
        str MedicalNotes
    }
    Ticket {
        ts DateTime
        str Status
        str Notes
    }
    Patient ||--o{ SensitiveData : "PRODUCE"
    Patient ||--o{ Ticket : "NEED"

```

## Data Identification

Let's introduce `PID` and `I_NO`, that are respectively `Patient` and `Installation` primary keys.
Then `DataNick` and `HouseNick` are human-friendly aliases respectively for `PID` and `I_NO`.
These aliases can be used in communication contexts among users who do not share the same data views but need to refer to the same objects.
Precisely, `DataNick` is used to refer the same `Patient`, `HouseNick` is used to refer the same `HouseNick`.

`Nick` generation can be:

1. **Random**: in this scenario `Nick` is randomly drawn at `Patient` creation
2. **Hashing**: a *derivation function* can be designed to map primary keys to `Nick`, see [here](./hashing/)

From now one we will omit `Nick` when not necesssary, we will point out when presence is relevant.

## Baseline Model (V0)

### ER Diagram

```mermaid
erDiagram
    SensitiveData {
        str DateTime
        str PID FK
        obj Values
    }
    Patient {
        str PID PK
        str I_NO UK "Installation Number (I#)"
        str CodiceFiscale UK
        str Name
        str Surname
        str Address
        str[] PhoneNo
        str[] MedicalNotes
    }
    Ticket {
        ts DateTime
        str I_NO FK
        str Status
        str Notes
    }
    Patient ||--o{ SensitiveData : "PRODUCE"
    Patient ||--o{ Ticket : "NEED"

```

Where `SensitiveData` is a generalization for `D1`, `D2`, `D3` and `D4` tables.

### ER Diagram - RBAC view

```mermaid
erDiagram
    SensitiveData {
        str DateTime
        str PID FK
        obj Values
    }
    Patient {
        str PID FK
        str I_NO FK
    }
    PatientDetail {
        str PID PK
        str CodiceFiscale
        str MedicalNotes
    }
    PatientGeneral {
        str I_NO PK
        str Name
        str Surname
        str Address
        str PhoneNo
    }
    Ticket {
        ts DateTime
        str I_NO FK
        str Status
        str Notes
    }
    Patient ||--o{ SensitiveData : "PRODUCE"
    Patient ||--|| PatientDetail : ""
    Patient ||--|| PatientGeneral : ""
    Ticket }o--|| PatientGeneral : "NEED"
```

### Logic Model

```js
Patient(PID, I_NO) // Lookup Table

PatientGeneral(I_NO, Name, Surname, Address, PhoneNo)
PatientDetail(PID, CodiceFiscale, MedicalNotes)

Ticket(DateTime, I_NO, Status, Notes)

SensitiveData(DateTime, PID, Values)
```

Where `PatientDetail` and `PatientGeneral` are encrypted, and `SensitiveData` is stored to the existing InfluxDB.

## What if `ID` disclosure is an issue

In the V0 Model, `monitor`, `dottori` and `analista` share `PID` knowledge.
This can be see as a weakness in separation of duty: basically the same `ID` is used to identify both sensible attributes and identifiers.
Table encryption will for sure mitigate reidentification risk, until tables will remain encrypted.
In case of data disclosure or leakage, a better implementation should obstruct joins between tables.

Let change `PID` to `XID` in `PatientDetail`, and remove the relationships between entities and the `Patient` table (that in the previous ER is used as a lookup table).

```mermaid
erDiagram
    SensitiveData {
        ts DateTime
        str PID
        float Measure1
        float Measure2
        __ __
        float MeasureN
    }
    PatientDetail {
        str XID
        str CodiceFiscale
        str MedicalNotes
        bool HasDementia
        __ __
        bool HasAnimals
    }
    PatientGeneral {
        str I_NO
        str Name
        str Surname
        str Address
        str PhoneNo
    }
    Ticket {
        ts DateTime
        str Status
        str Notes
    }
```

However now each user group has to be able to reference the same `Patient` even if they don't share the same view of the data.

### Aliasing

In this solution, two channels are aliased:

1. `dottori` ↔️ `monitor`/`analista` is called `DataNick`
2. `tecnico` ↔️ `monitor` is called `HouseNick`

This solution requires to define all lookup operation:

0. `XID` ➡️ `DataNick`
0. `PID` ➡️ `DataNick`
0. `DataNick` ➡️ `XID`
0. `DataNick` ➡️ `PID`
0. `HouseNick` ➡️ `PID`
0. `HouseNick` ➡️ `XID`
0. `HouseNick` ➡️ `I_NO`
0. `PID` ➡️ `HouseNick`
0. `XID` ➡️ `HouseNick`
0. `I_NO` ➡️ `HouseNick`

Then, for each operation, it shoud be defined which user groups need to access it.

```mermaid
erDiagram
    SensitiveData {
        str DateTime
        str PID FK
        obj Values
    }
    Patient {
        str DataNick PK
        str XID FK
    }
    Installation {
        str HouseNick PK
        str I_NO FK
    }
    Stream {
        str PID PK
        str DataNick FK
        str HouseNick FK
    }
    PatientDetail {
        str XID PK
        str CodiceFiscale
        str MedicalNotes
    }
    PatientGeneral {
        str I_NO PK
        str Name
        str Surname
        str Address
        str PhoneNo
    }
    Ticket {
        ts DateTime
        str HouseNick FK
        str Status
        str Notes
    }
    Stream ||--|| Installation : "channel"
    Installation ||--|| PatientGeneral : "alias"
    Patient ||--|| PatientDetail : "alias"
    Installation ||--o{ Ticket : "NEED"
    Stream ||--o{ Ticket : "NEED"
    Stream ||--o{ SensitiveData : "PRODUCE"
    Patient ||--|| Stream : "channel"
```

### Why?
<!-- Double Key explanatiomn -->
In an implemetation overview, you don't have to specify the capabilities for all lookup operation, anyone who already can decrypt a lookup table is already authorized a priori to perform any combination of lookup operation inside the same lookup table. More details [here](../auth-ac).

### Logical Model

```js
Patient(XID, DataNick)
Installation(I_NO, HouseNick)
Stream(PID, HouseNick, DataNick)

PatientGeneral(I_NO, Surname, Name, Address, PhoneNo)
PatientDetail(XID, CodiceFiscale, MedicalNotes)
SensitiveData(DateTime, PID, Values)

Ticket(DateTime, HouseNick, Status, Notes)
```

Where `Patient`, `Installation` and `Stream` are basically encrypted lookup tables, `PatientDetail` and `PatientGeneral` are also encrypted, and `SensitiveData` is stored to the existing InfluxDB.
