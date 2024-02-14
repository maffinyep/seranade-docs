# Serenade Docs

After a deep evaluation, we decided to follow `V0` data model, specifically:

- We will not enforce implementations that avoid ID disclosure
- We will perform server-side encryption with TDE

Previous solutions have been discarded because they will overachieve the security needs of the problem, adding excessive complexity to the project, especially in terms of maintainability.

We still report `V2` as a byproduct of this analysis, and eventually for future extension of this project.

## Stack

![Serenade Architectural Flow](./media/ark.png "Serenade Architectural Flow")
