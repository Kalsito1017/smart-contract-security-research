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


### [M-1] Looping through players array to check for duplicates in `PuppyRaffle::enterRaffle` is a potential denial of service (DoS) attack, incrementing gas costs for future entrants.


**Description:** The PuppyRaffle::enterRaffle function loops through the players array to check for duplicates. However, the longer the `PuppyRaffle::players` array is, the more checks a new player will have to make. This means the gas costs for players who enter right when the raffle stats will be dramatically lower than those who enter later. Every additional address in the `players` array, is an additional check the loop will have to make.

```javascript
 // Check for duplicates
        for (uint256 i = 0; i < players.length - 1; i++) {
            for (uint256 j = i + 1; j < players.length; j++) {
                require(
                    players[i] != players[j],
                    "PuppyRaffle: Duplicate player"
                );
            }
        }

```

**Impact:** The gas costs for raffle entrants will greatly increase as more players enter the raffle. Discouraging later users from entering, and causing a rush at the start of a raffle to be one of the first entrants in the queue.

An attacker might make the PuppyRaffle::entrants array so big, that no one else enters, guaranteeing themselves the win.

**Proof of Concept:** 
If we have 2 sets of 100 players enter, the gas costs will be as such:

1st 100 players: ~6252048 gas

2nd 100 players: ~18068138 gas

This is more than 3x more expensive for the second 100 players.
<details>
<summary>PoC</summary>

Place the following test in `PuppyRaffleTest.t.sol`:

```solidity
function test_denialOfService() public {
    vm.txGasPrice(1);
    // Let's enter 100 players
    uint256 playersNum = 100;
    address[] memory players = new address[](playersNum);
    for (uint256 i = 0; i < playersNum; i++) {
        players[i] = address(uint160(i + 10)); // addresses 10 to 109
    }
    // see how much gas it costs
    uint256 gasStart = gasleft();
    puppyRaffle.enterRaffle{value: entranceFee * players.length}(players);
    uint256 gasEnd = gasleft();
    uint256 gasUsedFirst = (gasStart - gasEnd) * tx.gasprice;
    console.log("Gas cost of the first 100 players: ", gasUsedFirst);

    // now for the 2nd 100 players - non-overlapping addresses (110 to 209)
    address[] memory playersTwo = new address[](playersNum);
    for (uint256 i = 0; i < playersNum; i++) {
        playersTwo[i] = address(uint160(i + 110));
    }
    // see how much gas it costs
    uint256 gasStartSecond = gasleft();
    puppyRaffle.enterRaffle{value: entranceFee * playersTwo.length}(
        playersTwo
    );
    uint256 gasEndSecond = gasleft();
    uint256 gasUsedSecond = (gasStartSecond - gasEndSecond) * tx.gasprice;
    console.log("Gas cost of the second 100 players: ", gasUsedSecond);

    assert(gasUsedFirst < gasUsedSecond);
}
```
</details> 


**Recommended Mitigation:**   There are a few recommended mitigations.
Consider allowing duplicates. Users can make new wallet addresses anyways, so a duplicate check doesn’t prevent the same person from entering multiple times, only the same wallet address.

Consider using a mapping to check duplicates. This would allow you to check for duplicates in constant time, rather than linear time. You could have each raffle have a uint256 id, and the mapping would be a player address mapped to the raffle id.

```diff
+ mapping(address => uint256) public addressToRaffleId;
+ uint256 public raffleId = 0;
.
.
.
function enterRaffle(address[] memory newPlayers) public payable {
    require(msg.value == entranceFee * newPlayers.length, "
        PuppyRaffle: Must send enough to enter raffle");
    for (uint256 i = 0; i < newPlayers.length; i++) {
      +    players.push(newPlayers[i]);
+    addressToRaffleId[newPlayers[i]] = raffleId;
+}
+
+// Check for duplicates
+// Check for duplicates only from the new players
+for (uint256 i = 0; i < newPlayers.length; i++) {
+    require(addressToRaffleId[newPlayers[i]] != raffleId, "PuppyRaffle: Duplicate player");
+}
 for (uint256 i = 0; i < players.length; i++) {
     for (uint256 j = i + 1; j < players.length; j++) {
-    require(players[i] != players[j], "PuppyRaffle: Duplicate player");
+    require(players[i] != players[j], "PuppyRaffle: Duplicate player");
 }
 }
 emit RaffleEnter(newPlayers);
 }

 function selectWinner() external {
+    raffleId = raffleId + 1;
     require(block.timestamp >= raffleStartTime + raffleDuration, "PuppyRaffle: Raffle not over");
```