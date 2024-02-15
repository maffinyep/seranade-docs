# Serenade Docs

After a deep evaluation, we decided to follow `V0` data model, specifically:

- We will not enforce implementations that avoid ID disclosure
- We will perform server-side encryption with TDE

Previous solutions have been discarded because they overachieve the security needs of the problem, and add excessive complexity to the project, especially in terms of maintainability.

We still report `V2` as a byproduct of this analysis, and eventually for future extension of this project.

## Index

0. [Stack](#stack)
1. [Data Model](./model/)
   - [ID Aliases](./model/hashing/)
2. [Authentication and Access Control](./auth-ac/)
3. [Data Encryption](./encryption/)
4. [Data Flows](./flow/)

## Stack

![Serenade Architectural Flow](./media/ark.png "Serenade Architectural Flow")
