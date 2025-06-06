<!DOCTYPE html>
<html>
<head>
<style>
    .full-page {
        width:  100%;
        height:  100vh; /* This will make the div take up the full viewport height */
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }
    .full-page img {
        max-width:  200;
        max-height:  200;
        margin-bottom: 5rem;
    }
    .full-page div{
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }
</style>
</head>
<body>

<div class="full-page">
    <img src="./logo.jpeg" alt="Logo">
    <div>
    <h1>Protocol Audit Report</h1>
    <h3>Prepared by: Daniel Simanullang</h3>
    </div>
</div>

</body>
</html>

<!-- Your report starts here! -->

# Table of Contents
- [Table of Contents](#table-of-contents)
- [Protocol Summary](#protocol-summary)
- [Disclaimer](#disclaimer)
- [Risk Classification](#risk-classification)
- [Audit Details](#audit-details)
  - [Scope](#scope)
  - [Roles](#roles)
- [Executive Summary](#executive-summary)
  - [Issues found](#issues-found)
- [Findings](#findings)
- [High](#high)
- [Informational](#informational)

# Protocol Summary

The `PasswordStore` protocol is a single-user protocol that enables the owner to store their password on-chain. The protocol does not support multiple users usage.

# Disclaimer

The Daniel Simanullang team makes all effort to find as many vulnerabilities in the code in the given time period, but holds no responsibilities for the findings provided in this document. A security audit by the team is not an endorsement of the underlying business or product. The audit was time-boxed and the review of the code was solely on the security aspects of the Solidity implementation of the contracts.

# Risk Classification

|            |        | Impact |        |     |
| ---------- | ------ | ------ | ------ | --- |
|            |        | High   | Medium | Low |
|            | High   | H      | H/M    | M   |
| Likelihood | Medium | H/M    | M      | M/L |
|            | Low    | M      | M/L    | L   |

We use the [CodeHawks](https://docs.codehawks.com/hawks-auditors/how-to-evaluate-a-finding-severity) severity matrix to determine severity. See the documentation for more details.

# Audit Details 
## Scope 
```
./src/
└── PasswordStore.sol
```
## Roles
# Executive Summary
## Issues found
# Findings
High: 2

Informational: 1
# High
### [H-1] the `s_password` variable is visible on-chain

**Description**
Variables that are stored on-chain, despite them being `private`, is accessible to anyone that wish to view them. The intended way
to view the `s_password` variable is trough the `getPassword` function.
**Impact**
Anyone can view the passwords stored on-chain.
**Proof of Concepts**
1. Create a locally running chain
```bash
    make anvil
```
2. Deploy the contract
```bash
    make deploy
```
3. cast the storage tool
```bash
    cast storage
```
4. cast the string parser to retrieve the password
```bash
    cast parse-bytes32-string <hex>
```
**Recommended mitigation**
Encrypt the password off-chain, and only store the encrypted password on-chain. Make sure to also not accidentally trigger a function
that decrypts the user password

### [H-2] Non-owner can set the owner's password through `setPassword`

**Description**
The `setPassword` function lacks accessibility control, therefore making practically anyone to set everyone's password.

```javascript
    //  @audit only the owner has to be able to set the password!
    function setPassword(string memory newPassword) external {
        s_password = newPassword;
        emit SetNewPassword();
    }
```

**Impact**
Non-owners can set/change the password of the contract, severly breaking the contract intended functionality. 

**Proof of Concepts**
```javascript
    function test_non_owner_can_set_password(address randomAddress) public{
            vm.assume(owner!=randomAddress);
            vm.prank(randomAddress);
            string memory expectedPassword = "myPassword";
            passwordStore.setPassword(expectedPassword);

            vm.prank(owner);
            string memory actualPassword = passwordStore.getPassword();
            assertEq(actualPassword, expectedPassword);

        }
```

**Recommended mitigation**
Add access control conditional to the `setPassword` function.
```javascript
    if(msg.sender != s_owner) {
        revert PasswordStore_notOwner();
    }
```

# Informational
### [I-1] Incorrect natspec in `getPassword`

**Description**
The `function getPassword()` has no parameter, while the natspec indicates that it should be `function getPassword(string)`

**Recommended mitigation**
Remove the following comment:
```diff
- * @param newPassword The new password to set.
```

