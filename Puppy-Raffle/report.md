### [I-1] pragma solidity ^0.7.6 Invalid Version of the contract! Current one is 0.8.30!

## Description:
The contract specifies a Solidity compiler version pragma of ^0.7.6, which is outdated. The current project standard version is 0.8.30. Using an older compiler version can introduce compatibility issues, miss important language features, and potentially expose the contract to known bugs or security vulnerabilities fixed in later Solidity releases.

## Impact:

Potential incompatibility with other contracts or tooling expecting Solidity 0.8.x.

Loss of newer compiler safety checks and language improvements introduced after 0.7.x.

Increased risk of bugs or security vulnerabilities that have been addressed in Solidity 0.8.30.

May cause build or deployment failures if the environment expects a consistent compiler version.

## Proof of Concept:
In the contract file, the pragma line:

```javascript
   pragma solidity ^0.7.6;
``` 
differs from the project's standard:

```javascript
pragma solidity ^0.8.30;
```
Attempting to compile with 0.8.30 the contract will either fail or produce warnings about incompatible syntax or behavior.

## Recommended Mitigation:

Update the pragma statement in the contract source code to match the project standard:

```diff
+ pragma solidity ^0.8.30;
```
Test the contract thoroughly after upgrading the compiler version to ensure compatibility.

Adjust code as necessary to comply with Solidity 0.8.x breaking changes and new best practices.

### [S-#] TITLE (Root Cause + Impact)

**Description:** 

**Impact:** 

**Proof of Concept:**

**Recommended Mitigation:** 