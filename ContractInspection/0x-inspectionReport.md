# A Report of Contract Issues of 0x protocol 

### Abstract 

As a beginner of code review, I was very excited to read this file for the first time. Because its code format of my first glance was very neat and well organized, the dataflow is logically compact and elegant. Through the code review practice, I have learned quite much from this file. The contracts file in this report are subject to 0x protocol, which facilitates the exchange of Ethereum-based assets. Those significant issues in Ethereum history could have been reflected in this contracts files. Therefore, this report will draw lessons from past problems and summarize existing issues of this contracts file. The specific issues will be divided into three types: the maintainability if the contract, interactions, and existing bugs/errors.


### Code to be inspected

The [contracts file](https://github.com/0xProject/0x-monorepo/tree/development/packages/contracts/src/contracts/current) which is subject to 0x protocol project is given on April 23. Up to this report sent, the newest [modification](https://github.com/0xProject/0x-monorepo/commit/13299158d1e22d1af1cd36434fc403a74743ecb1) of the contract file was on March 4. The contracts file include 5 original folders: multisig, protocol, test, tokens and tutorials, and a third-party folder which includes SafeMath and Ownable file.


### Inspection methods

Due to lack of code audit experience, I am not familiar with the standard specification and procedures of code audit. Therefore the report was written in my own style and the inspection is going through dataflow within modifiers and functions. In this way, I tried my best to cover all functions and modifiers and their dataflow. In order to keep report neat, some issues within any contract inherited from its parent contract will not be discussed again.   


### Issues discussion

#### Type 1: Lack of maintainability

The code regarding security should be first ensured about its maintainability. Whichever the contract would be updated or multi-contributed, a good format and illustration should be ensured all through the code file. The code format through the contracts is quite unified, while some contracts lack good illustration: **ERC20Token** in tokens, **MaliciousToken, DummyToken, Mintable** in the test and **EtherDelta** in tutorials. Even the test implementation should be illustrated well in order for following contributors to follow. While bad (or no) illustration will result in confusion of other people, as well the errors or bugs which could not be easily found. 

The test is also very significant to the maintainability. While the test in this contracts file seems has not covered all range of functions and resulted dataflow. Further, the test seems incomplete as it has not yet covered all circumstances to the functionalities of the whole contract files. 

Apart from above, small issues like the clean structure of code may not be necessarily addressed. A good structure of code may talk about the position of declarations of variables and event which are better seated in the front of the contract, in order for future maintenance. So **ERC20Token and Token** in tokens should be modified. 

#### Type 2: Inefficient interactions

The term 'interactions' I have drawn here include the consideration of necessary event logs and gas use of repeated functions. I think some important procedure of token transactions should be event-logged as there would be outside attacks, for example for **Token** folder. Good event log on abnormal transactions will help track any root of bugs and eliminate loss. 


#### Type 3: Bugs/Errors 

Bugs/Errors type issues can be separated into 3 classes: 
- **Better be modified**: no vulnerability but the modification of some functions could make function better
- **Code incomplete**: the vulnerability of a function probably results from its incompleteness.
- **Should be modified**: there is any crucial vulnerability if the function not been modified.
Details are shown below:

Contract | Function/Modifier | Class |Problem | Suggestion
:- | :- | :- | :- | :-
MultiSigWallet | `notNull` | BetterModified | Though it pre-check the destination is not 0, the format of a valid address is not assured. The parsed address missing or adding a digit becomes invalid, even though it could be copied and pasted. | At least to make sure the address is valid by importing third-party library like `wallet-address-validator` or validator API from web3.
MultiSigWallet | `MultiSigWallet` | BetterModified | If one owner address exists in the last wallet, it will throw. One address cannot rule several wallets. | It is better to give a notification that the specific address has been added before in order to check out this address or validate the address to be used in a next wallet.
MultiSigWallet | `isConfirmed` | BetterModified | It loops all the owners to check their status, inefficient in gas use. | It can be modified as the function directly return false by breaking the loop if the loop has checked out someone's status is not confirmed.
MultiSigWallet | `getTransactionIds` | BetterModified | The returned array may contain several 0 values and it would not be efficient (use more gas) to loop through all the `transactionCount`. | The function could be modified as it gives out the length of `_transactionIds` by `to` and `from`, before looping through `transactionCount` and filling results into the array.
MultiSigWallet | `validRequirement` | BetterModified | The `MAX_OWNER_COUNT` is constant set to be 50, while the wallet could require more owners to participate. | Add a function to change the `MAX_OWNER_COUNT`. 
MultiSigWallet | `addOwner`	<br>`removeOwner` | BetterModified | It seems that when the `removeOwner` is invocated, the `addOwner` cannot be used as the `required` has been changed to exactly the length of `owners` length. | Remove the `changeRequirement(owners.length)` from `removeOwner`, or add `changeRequirement(owners.length + 1)` to `addOwner`.
MultiSigWallet | `executeTransaction` | ShouldModified | The call function seems only return a boolean in this circumstance and it may leave rest gas which cannot be consumed up by fallback | change `call` to `send` or `transfer`.
MultiSigWallet | fallback | Incomplete 	<br>/BetterModified | The fallback function only contains one circumstance: `msg.value > 0`, or the fallback too complicated| add `if msg.value = 0 throw` and even log to consume gas left by external functions./ Make fallback as simple as possible, for example only set an event log. 
ERC20Token 	<br>WETH9| `transfer`	<br>`transferFrom`| BetterModified | Although it prevents the uint256 overflow problem to have happened if the `_value` parsed is very large, a problem may happen if future update allows another significant value to be added to the `_value`. |  Using safeMath always goes right.
WETH9 | `transfer`	<br>`transferFrom`| BetterModified | Although it prevents the uint256 overflow problem to have happened if the `_value` parsed is very large, a problem may happen if future update allows another significant value to be added to the `_value`. |  Using safeMath always goes right.
AccountLevels | `accountLevel` | Incomplete | only for regular users
Arbitrage | `Arbitrage` | ShouldModified | the number of parsed in params doesn't match in `Exchange` and `EtherDelta` | 


Apart from lists above, there should also be vulnerability issues related to ether forced to be transferred to the contract. In this way, attackers may invocate external function like `selfdestruct`. The visibility of variables in this contracts file seems very good while some functions don't. 





