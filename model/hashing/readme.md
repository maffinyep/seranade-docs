# Alias derivation

In those scenarios we don't need anymore to store any `Nick` into the data model, `Nick` can be derived locally.

## S-Table Hashing

We can figure `Nick` aliases like hashing of their primary keys. Then an `S-table` will bind hashing to representation.

Let's see a toy example of hashing. Imagine a simple `S-table` like this:

| Byte | Word |
|---|---|
| 00 | Black |
| 01 | Blue |
| 10 | Red |
| 11 | Magenta |

The `Nick` derivation will follow this flow:

```mermaid
stateDiagram-v2
    %% direction LR
    [*] --> Hashing : PID=01010101
    Hashing --> S_Table : 1011
    S_Table --> [*] : Nick=Patient_Red_Magenta
```

## Fiat Shamir

## RSA Lehmer
