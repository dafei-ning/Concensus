# Independent Report of the Contract Issues

This report only assesses the vulnerabilities and design pattern flaws of the contract, rather than giving out specific code audit or modification, as the report writer may not fully comprehend the intent of the contract creator. However this report may have necessarily pointed out some to-be-improved or incompete issues.	


### Brief description of the dataflow

- Contract creator initiate eventCreditQuota, accessCreditQuota, eventCharge, tokenCurrency, tokenValue.
- Contract creator authorizes staff. (newly added)
- Potential clients buy tickets to yield credits for access to content.
- Check if there is enough credits for free access, otherwise the client is charged.
- Check if there is enough content accesses for free ticket, otherwise the client is charged.
- One can create a token redeeming request 
- The approved request of redeeming is executed by staff
- One can publish any content by sending a request
- The approved request of content published is executed 	


### Vulnerabilities and design pattern flaws

Function | Evaluation | Problem | Suggestion
:- |:- |:- |:-
CustomContract	|Safe		
clientCreate	|Controversial |Strangers can call this function and create vicious storage issue.|	Better to change it to a private function, or there should be at least a function to determine whether the 'Client' is valid.
setEventCreditQuota	|Safe		
getEventCreditQuota	|Safe		
setAccessCreditQuota|	Safe		
getAccessCreditQuota|	Safe		
setEventCharge	|Safe		
getEventCharge	|Safe		
getAccessCount	|Controversial|	If considering privacy, strangers can access the info of clients.|	Make a list which includes the addresses of valid clients and company staff. Therefore there should also be a function which is called to authorize any address to the list.
getEventCount	|Controversial	|If considering privacy, strangers can access the info of clients.|	Make a list which includes the addresses of valid clients and company staff. Therefore there should also be a function which is called to authorize any address to the list.
setLicenseValue	|Safe		
buyEventTicket	|Unsafe	|This function is incomplete. If parameter 'Number' is very large, the total eventCount may exceed the limit of uint256, and become 0.	|Use SafeMath operation method, like those from library of Zeppelin.
runAccessPre|	Unsafe|	This function is incomplete and everyone can call this function to change his eventCount number.|	Complete the function by completing the charge process and set it to be a private function to ensure parameters passed in are valid when called.
runAccessPost	|Safe	|Seems incomplete	
runFinalize	Safe	|Seems incomplete	
publishContent	|Unsafe|	Incomplete and vicious strangers can call this function to publish content without valid verification.|	The signiture passed in should not only be a string. It should be structured with more confidential or detailed information such as address of msg.sender, publishing time, and event authorization code.
getPendingApprovalRequest|	Safe		
approveContentExecuted|	Safe		
redeemTokenRequest|	Error|	token number = msg.value|	numToken should be calcuated as numToken = RedeemValue / tokenValue. And there should be a checking function make sure the tokenValue is set to be non-zero before redeem.  
getPendingRedeemRequest	|Safe		
redeemTokenExecuted	|Unsafe| the redeem process can be called by anyone with no valid proof. |Set the function to be called only by authorized staff address.There should be a staff list on which any staff is authorized by the contract owner. There should also be a function checking if the msg.sender is a staff on the list.
redeemDbg |Unsafe |The bug problem is public rather than confidential. | Set the function to be called only by authorized staff address.
setTokenValue |Unsafe |Token value can be set by strangers.	| Set the function to be called only by authorized staff address.
getTokenValue|Safe		
payCredit|Unsafe|Payment can be called by strangers |Set the function to be private that only able to be called within this contract.	
	



### New functions suggested

- First of all, modifiers like `onlyCreater`, `onlyStaff`, `onlyValidClient` can be used to reduce cost of calling functions.
- Regarding clients engaged,`clientValid` function should used to check if the client is valid in order to reduce cost of performing operations on vicious strangers.
- A list of `validClient` is used to determine if the client is among those valid before accessing any private information.
- `changeClientStatus` function is used to modify the client to be valid or not to get rid of vicious clients.
- There should be a `tokenTransfer` function let the token redeeming automatically processed. 
- Any staff rather than only the contract creator can be authorized by `authorizeStaff` to execute some process.  
- A good and complete `testBuyTicket` function in which at least ticket number to buy should not be exactly 1.



### Limitation and lost

I don't know what these function do exactly:
- `runAccessPost` 
- `runFinalize`
- `setLicenseValue`	
	
I don't understand whether there is a `approvalRequestsLength` even the `approvalRequests.length` is the same value, in the `publishContent`. Thus I don't know why `approvalRequestsLength` could be unequal to `approvalRequests.length` and the detecting bug was based on this.	
Same confusion applies in `redeemTokenRequest` function.






