### [H-1] The password variable stored on-chain storage is visible to anyone

**Description:** All data stored on-chainis visible to anyone, and can be read directly from the blockchain. The `PasswordStore::s_password` variable is intended to be private for only the owner of the contract.

We show one such emthod of reading any off chain below

**Impact** Anyone can read the private password, severely breaking the functionality of the protocol

**Proof of Concept:** (Proof of Code)
Below shows how anyone can read contract storage off the blockchain

1. ```bash
   $ make anvil
   ```
2. Deploy the contract to the chain.
   `make deploy`
3. Run the storage tool
   We use `1` because that is the storage slot of `PasswordStore::s_password` in the contract

```
cast storage <CONTRACT_ADDRESS> --rpc-url http://127.0.0.1:8545
```

it will return bytes32 data:

```
0x6d7950617373776f726400000000000000000000000000000000000000000014
```

now convert the bytes32 data to a string

```
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```

This will give us an output of

```
myPassword
```

**Recommended Mitigation:**
This is an architectual error storing non-encrypted data on-chain. One solution is to encrypt the password off chain with a secret key, then put the encrypted key on-chain. Also, the getPassword function should be removed to prevent accidently displaying your secret key. However, this solution will make the owner store a secret key off-chain.

## Likelihood and Impact:

- Impact: High
- likelihood: High
- Severity: High

### [H-2] `PasswordStore::setPassword` has no acces controls which mean anyone can change the password

**Descripion:**
Anyone can call the function `PasswordStore::setPassword` with a string which will be set as the new password. Also, the natspec of the contract says `This allows only the owner to retrieve the password.`

```javascript
function setPassword(string memory newPassword) external {
        s_password = newPassword;
@>      // @audit - there are no access controls
        emit SetNetPassword();
    }

```

**Impact** Anyone can change the password of the contract

**Proof of Concept:** Add the following to `PasswordStore.t.sol` test file.

<details>
<summary>code</summary>

```javascript

function test_non_owner_reading_password_reverts() public {
        vm.startPrank(address(1));
        vm.expectRevert(PasswordStore.PasswordStore__NotOwner.selector);
        passwordStore.getPassword();
}

```

</details>

**Recommended Mitigation:**
Add an access control like the following

```javascript
if(msg.sender != s_owner){
    revert PassWordStore__NotOwner();
}
```

## Likelihood and Impact:

- Impact: High
- likelihood: High
- Severity: High

### [I-1] The `PasswordStore::getPassword` natspec indicates there shoould be a parameter that doesn't exist, natspec is incorrect

**Descripion:** From the natspec documentation `@param newPassword The new password to set.` However, there is no param newPassord that exists in the function.

**Impact** The natspec is incorrect

**Recommended Mitigation:**

```diff
- * @param newPassword The new password to set.

```

## Likelihood and Impact:

- Impact: None
- likelihood: Low
- Severity: Informational /Gas/Non-crits
