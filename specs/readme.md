# Specs

> An authentication and access control system for the participating entities.
> The system will control who can access which portion of data.

A Keyclock instance will be configured. A middleware API will manage interactions among client requests, Keycloak and Database.

> Locally compute a PID for each patient

No consistent advantages have been evaluated for client-side `PID` generation.
Server-side generation will avoid `PID` collisions.

A `POST /api/patient` method will be implemented.

> Use a cryptographic technique that gives same-length PIDs (possibly readable).
> Ensures difficult reversing and low probability of collision.

`PID` will be randomly generated (specifically PCG or MT algorithms).
A random `PID` will be impossible to reverse except for data disclosure.

> Consider hashing (with SHA-256 or better) on the whole string of identifying data (D0) with a salt.

No consistent advantages have been evaluated for hashing an identifier.

> Since PIDs will likely be long, a shortcut may be used for convenience considering for example the first or last n chars ensuring that no collision occurs

An S-Table will be implemented, similar to project name generators used in common websites.

> Store in a very secure encrypted form the association between the PID and the patient identifying data (including identifying attributes in D0 but also any internal ID used by HOS to identify the patient in their systems)

Lookup tables will be stored in a DB featured by TDE

> The initialization and intervention protocols explained above

API will manage protocol phases.

> An API that accepts requests by an authorized entity that includes a PID, converts the PID in the patient's corresponding identifier

A `GET /api/lookup?PID={0}` method will be implemented. Authentication will be required.

> Performs actions like sending data to another entity that is authorized to access identified data

Any record should be sent or stored with `PID`, and **ONLY** `HOS` FrontEnd should be allowed to perform `PID` lookup.

> The API can be called by sending pseudo-identified data resulting from data analysis and producing an identified dashboard widget for the clinicians.

We instead expect that `UniMI` will upload analyses possibly on an external independent DB or data storage. An `HOS` FrontEnd will follow these steps:

1. Select a `Patient`
2. Obtain his/her `PID`
3. Fetch external DB (or service) by querying on `PID`
