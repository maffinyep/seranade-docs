# Authentication and Access Control

## Table Access

We specify for each user group which table can be accessed.

- ✅ is granted
- ❌ is not granted
- ⭕ grant it's not needed

See [here](../model/) to review the logical model of tables.

### `V0` - All Tables

| | Doctor | Monitor | Tech |
| - | - | - | - |
| Patient | ✅ | ✅ | ❌ |
| PatientGeneral | ✅ | ❌ | ✅ |
| PatientDetail  | ✅ | ❌ | ❌ |
| Ticket | ⭕ | ✅ | ✅ |
| SensitiveData  | ⭕ | ✅ | ❌ |

### `V2` - Lookup Tables

| | Doctor | Monitor | Tech |
| - | - | - | - |
| Patient | ✅ | ❌ | ❌ |
| Installation | ✅ | ❌ | ✅ |
| Stream | ✅ | ✅ | ❌ |

### `V2` -Other Tables

| | Doctor | Monitor | Tech |
| - | - | - | - |
| PatientGeneral | ✅ | ❌ | ✅ |
| PatientDetail  | ✅ | ❌ | ❌ |
| Ticket | ⭕ | ✅ | ✅ |
| SensitiveData  | ⭕ | ✅ | ❌ |

## Authentication with Keycloak

Users can register or manage their accounts from `keycloack.domain/realms/serenade/account/`.

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

#### User signup

![Serenade SignUp](../media/signup.png "Serenade SignUp")

![Serenade Verify](../media/verify.png "Serenade Verify")

#### Admin actions

![Serenade Users List](../media/admin.png "Serenade Users List")

![Serenade Group Grant](../media/grant.png "Serenade Group Grant")

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

#### ![Dottori FE Oauth](../media/dottori-fe-login.png "Doctor FE Oauth")

#### ![Keycloack Login](../media/login.png "Keycloack Login")

#### ![Dottori FE Home](../media/dottori-fe-home.png "Home")

#### Forgot Password

 ![Keycloak Forgot Password](../media/forgot.png "Keycloak Forgot Password")

## Access Control

### Grant Actions

Any action over the `Database` must be granted by `Keycloak`.
We can generalize all actions like the following sequence diagram:

```mermaid
sequenceDiagram
    %%actor User
    %%User->>App: Action (ID, Resource)
     App->>API: ResourceAction <br> (ID | Resource, Token)
    alt GRANT ACTION
        API->>KeyCloack: Check (Token, Resource)
        KeyCloack->>API: OK (Token, Resource)
        API->>Database: Query (Resource, ID)
        Database->>API: Result (Resource, ID)
    end
    API->>App: Result (Resource | ID)

```

Where `ResourceAction` can be:

- A `Lookup` query on an `ID` or `Nick`
- `Get` an existing `Patient` or `Ticket`
- `Post` a new `Patient` or `Ticket`
- `Pull` an update for a `Patient` or a `Ticket`
