# Authentication and Access Control

## Table Access

### V0 - Tables

|                | Doctor | Monitor | Tech |
| -------------- | ------ | ------- | ---- |
| Patient        | ✅      | ✅       | ❌    |
| PatientGeneral | ✅      | ❌       | ✅    |
| PatientDetail  | ✅      | ❌       | ❌    |
| Ticket         | ⭕      | ✅       | ✅    |
| SensitiveData  | ⭕      | ✅       | ❌    |

<!-- Users of `monitor` group need to access `Patient` table to know, but `I_NO` is not a col -->

### V2 - Lookup Tables

|              | Doctor | Monitor | Tech |
| ------------ | ------ | ------- | ---- |
| Patient      | ✅      | ❌       | ❌    |
| Installation | ✅      | ❌       | ✅    |
| Stream       | ✅      | ✅       | ❌    |

### V2 -Other Tables

|                | Doctor | Monitor | Tech |
| -------------- | ------ | ------- | ---- |
| PatientGeneral | ✅      | ❌       | ✅    |
| PatientDetail  | ✅      | ❌       | ❌    |
| Ticket         | ⭕      | ✅       | ✅    |
| SensitiveData  | ⭕      | ✅       | ❌    |

## Authentication with KeyCloak

### Signup without IdP

```mermaid
sequenceDiagram
    actor Doctor
    participant App
    participant API
    participant KeyCloack
    actor Admin
    Doctor->>KeyCloack: Send (Email,Password)
    KeyCloack->>Doctor: OK
    KeyCloack-->>KeyCloack: Store
    Admin->>KeyCloack: Add capabilities
```

### Signup with IdP

```mermaid
sequenceDiagram
    participant IdP
    actor Doctor
    participant App
    participant KeyCloack
    Doctor->>KeyCloack: Send (IdP)
    KeyCloack->>Doctor: Ask (Token)
    Doctor->>IdP: Check (Email,Password)
    IdP->>Doctor: OK (Token)
    Doctor->>KeyCloack: Send (Token)
    KeyCloack->>IdP: Check (Token)
```

### Login

### Reset Password

## Access Control

### Remote Lookup

```mermaid
sequenceDiagram
    actor User
    User->>App: Lookup (ID, Resource)
    App->>API: Lookup (ID, Token)
    API->>KeyCloack: Check (Token, Resource)
    KeyCloack->>API: OK (Token, Resource)
    API->>DataBase: Query (Resource, ID)
    DataBase->>User: Result (Resource, ID)
```

### Local Lookup

```mermaid
sequenceDiagram
    actor Doctor
    participant App
    participant API
    participant KeyCloack
    actor Admin
    Doctor->>KeyCloack: Send (Email,Password)
    KeyCloack->>Doctor: OK
    KeyCloack-->>KeyCloack: Store
    Admin->>KeyCloack: Add capabilities
```

### Add `Patient`

```mermaid
sequenceDiagram
    actor Doctor
    Doctor->>App: Add (PatientGeneral, PatientDetail)
    App->> API: Add (PatientGeneral, PatientDetail, Token)
    API->>KeyCloack: Check (Token, Resource)
    KeyCloack->>API: OK (Token, Resource)
    API-->API: Generate: PID, I_NO
    API->>DataBase: Add Patien (, PatientGeneral, PatientDetail, )
```
