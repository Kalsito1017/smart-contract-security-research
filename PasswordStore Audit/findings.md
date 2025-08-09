### [S-#] Storing the password on-chain makes it visable to anyone, and no longer private

**Description:** All data stored on-chain is visible to anyone, and can be read directly from the blockchain. The `PasswordStore::_password` variable is intended to be a private variable and only accessed through the `PasswordStore::getPassword` function, which is intended to be only called by the owner of the contract.

We show one such method of reading any data off chain below.

**Impact:** Anyone can read the private password, severly breaking the functionality of the protocol.

**Proof of Concept:**  

The following steps demonstrate how anyone can read the `s_password` variable directly from the blockchain storage:

1. Start a local blockchain instance  
   
   
2.Deploy the contract
(Ensure it’s deployed to the local node started by Anvil.)

3.Read the storage slot
The `s_password` variable is stored in slot 1. Use cast to read it:

cast storage <CONTRACT_ADDRESS> 1 --rpc-url http://127.0.0.1:8545
Example output:

0x6d7970617373776f726400000000000000000000000000000000000000000000

4.Parse the storage value into a string

cast parse-bytes32-string 0x6d7970617373776f726400000000000000000000000000000000000000000000
Output:

 `myPassword`

This confirms that the private password can be retrieved directly from on-chain storage without calling the contract’s getter function.

**Recommended Mitigation:** Due to this, the overall architecture of the contract should be rethought. One could encrypt the password off-chain, and then store the encrypted password on-chain. This would require the user to remember another password off-chain to decrypt the password. However, you'd also likely want to remove the view function as you wouldn't want the user to accidentally send a transaction with the password that decrypts your password.
