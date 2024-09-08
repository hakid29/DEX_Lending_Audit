## 1. nullorm
commit version : ef1a322

https://github.com/Null0RM/Lending_solidity/blob/main/src/UpsideLending.sol
<br></br>
### 1. repay token 검증
UpsideLending.sol/repay:114

**설명**

_token이 usdc인지 검증하지 않는다. 따라서 임의로 발생한 아무 토큰으로 `repay` 함수를 호출하여 빌린 usdc를 갚을 수 있다.

```solidity
function repay(address _token, uint256 _amount) external {
    _debtUpdate(msg.sender);
    User memory user = user_info[msg.sender];

    require(user.borrow_usdc >= _amount, "EXCEEDS_AMOUNT_TOKEN_REPAY");
    require(IERC20(_token).balanceOf(msg.sender) >= _amount, "USER_INSUFFICIENT_TOKEN");
    require(IERC20(_token).allowance(msg.sender, address(this)) >= _amount, "INSUFFICIENT_ALLOWANCE");
    
    IERC20(_token).transferFrom(msg.sender, address(this), _amount);
    user.borrow_usdc -= _amount;
    total_borrowed -= _amount;

    user_info[msg.sender] = user;
    _depositUpdate();
}
```

<br></br>
**파급력**

Critical
<br></br>

**해결방안**

_token이 usdc인지 확인하는 로직이 추가되어야 한다.
<br></br>

## 2. kenny
commit version : 8c1d7cd

https://github.com/55hnnn/Lending_solidity/blob/main/src/DreamAcademyLending.sol
<br></br>
### 1. repay token 검증
DreamAcademyLending.sol/repay:77

**설명**

token이 usdc인지 검증하지 않는다. 따라서 임의로 발생한 아무 토큰으로 `repay` 함수를 호출하여 빌린 usdc를 갚을 수 있다.

```solidity
function repay(address token, uint256 amount) external {
    require(token != address(0), "Invalid token address");
    require(amount > 0, "Amount must be greater than zero");
    _updateLoanValue(token); // 이자율 반영

    uint256 debt = debts[msg.sender][token];
    require(debt > 0, "No outstanding debt");
    require(amount <= debt, "Repayment amount exceeds debt");

    ERC20(token).transferFrom(msg.sender, address(this), amount);
    debts[msg.sender][token] -= amount;
    
    totalDebt -= amount;
}
```