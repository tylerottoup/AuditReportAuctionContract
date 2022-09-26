# Audit Report

##  Summary
An updated version of Solidity was used to develop the token auction contract. For the auction process to work, users must have tokens, **USDT**, or **BNB** in their possession to participate in the auction within the owner&#39;s predetermined time range and selling tokens while owner is also setting minimum value to check the auction passing status at the end of an auction. The individual who bids may then claim their token if the auction gets passed or refund their investment in case of auction failure when the auction is over.

## In scope

Auction.sol has been audited ,it is not been verified on the explorer as the contract has some severe issues. 

## Findings

According to the project&#39;s scope, there are several medium and low severity concerns with the code flow.

- constructor : start time and end time has been set without the use of **block.timestamp** . 
- refundAll() : the loop is transferring tokens to the users of an auction, it should be bounded
- refundAll() : one externally owned account has to call this function in order to refund all the participants , this function must be called individually by the every participants.
- msg.sender : must be converted into _msgSender() according to openzeppelin context.sol.
- depositAuction() : this Function is not checking if the user has deposited its BNB or not.
- claimTokens() : reentrancy is not handled.

## x Error (  `severity`  )

Severity assigning:
- high : depositAuction() function is not checking if the user has deposited its BNB or not this will lead to fund shortages in case of auction failure and fund stealing in case of success the deposit amount of BNB must be checked at the time of function call.
- high : refundAll() function is using a loop for transferring tokens to the users of an auction, it should be bounded otherwise is the otherwise, your contract could get stuck if the loop iteration is hitting the block's gas limit. If a loop is consuming more gas than the block's gas limit, that transaction will not be added to the blockchain; in turn, the transaction would revert. Hence, there is a transaction failure. Hence the the funds will be stuck in the contract forever.
- high : depositAuction() function must be handling reentrancy calls since the untrusted contract can make recursive calls back to the original function in an attempt to increase its investment value.
- high : claimTokens() function must be handling reentrancy calls since the untrusted contract can make recursive calls back to the original function in an attempt to drain the contract funds.
- medium : constructor is setting start time and end time without the use of **block.timestamp** which will lead to contract failure if the deployer set incorrect values start time less than endTime and doesn't set enough time to call the setCaps function then the contract auction will start without having offering token in supply.
- low : msg.sender must be converted into _msgSender() according to openzeppelin context.sol. this will prevent against the delegate call attacks.
owner privileges : none

#### Code snippet

```
constructor(address _offeringToken, uint256 _startTime, uint256 _endTime) 
    {
        offeringToken = _offeringToken;
        startTime = _startTime;
        endTime = _endTime;
    }
```


```
function refundAll() external {
        require(isClosed, "Auction not closed");
        require(!isSuccess, "Should distribute tokens");

        for (uint256 i = 0; i < addressList.length; i++) {
            UserInfo storage user = userInfo[addressList[i]];
            if(user.bnbAmount != 0){
                payable(addressList[i]).transfer(user.bnbAmount);
                user.bnbAmount = 0;
            }
            if(user.usdtAmount != 0){
                IERC20(USDT_TOKEN).transfer(addressList[i], user.usdtAmount);
                user.usdtAmount = 0;
            }            
        }
    }
```
```
function claimTokens() external {
```

```
function depositAuction(uint256 _amount, bool _isUSDT) external payable {
        checkConditions(_amount);
        UserInfo storage user = userInfo[msg.sender];
        if (user.bnbAmount == 0 && user.usdtAmount == 0) { // add new user to the list
            addressList.push(msg.sender);
        }

        if (_isUSDT) {
            IERC20(USDT_TOKEN).transferFrom(msg.sender, address(this), _amount);
            totalUSDT += _amount;
            user.usdtAmount += _amount;
        } else {
            totalBNB += _amount;
            user.bnbAmount += _amount;
        }
    }
```
    
## Conclusion
The above mentioned security concerns must be handled properly if not the funds of the contract are venerable to fund steal and reentrancy attacks and also fund collection functionality must be handled otherwise contract is not holding funds.
