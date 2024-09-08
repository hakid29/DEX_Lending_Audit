## 1. jacqueline

https://github.com/je1att0/DEX_solidity/blob/main/src/DEX.sol
<br></br>
### 1. LP token burn

**설명**

Dex.sol/removeLiquidity:49-63

```solidity
function removeLiquidity(uint _lpReturn, uint _minAmountX, uint _minAmountY) public returns (uint _tx, uint _ty) {
    uint poolAmountX = tokenX.balanceOf(address(this));
    uint poolAmountY = tokenY.balanceOf(address(this));
    _tx = poolAmountX*_lpReturn/totalLP;
    _ty = poolAmountY*_lpReturn/totalLP;
    require(_tx >= _minAmountX && _ty >= _minAmountY, "Insufficient minimum amount");
    require(_lpReturn <= poolAmountX + poolAmountY, "Insufficient LP return");

    tokenX.transfer(msg.sender, _tx);
    tokenY.transfer(msg.sender, _ty);
    totalLP -= _lpReturn;

    return (_tx, _ty);

}
```

`removeLiquidity` 함수를 호출할 때, 유저가 없앨 LP token을 burn하지 않는다. 따라서 유저는 원하는 만큼 무한으로 LP token을 reuse할 수 있다.
<br></br>
**파급력**

Critical
<br></br>
**해결방안**

`removeLiquidity` 함수에 유저의 LP token을 burn하는 로직을 추가하면 된다. 
<br></br>
### 2. 불필요한 public visibility 사용

**설명**

Dex.sol/swapXtoY:74

Dex.sol/swapYtoX:85

```solidity
function swapXtoY(uint _amountX,  uint _amountY, uint _minOutput) public returns (uint swapReturn) {
```

```solidity
function swapYtoX(uint _amountX, uint _amountY, uint _minOutput) public returns (uint swapReturn) {
```
<br></br>
`swapXtoY` 함수와 `swapYtoX` 함수가 public 으로 선언되어 있다. 이는 `swap` 함수에서 했던 amountX, amountY에 대한 검사를 하지 않을 뿐더러 `swap` 함수가 존재할 필요가 없게 만든다. 
<br></br>
**파급력**

Low
<br></br>
**해결방안**

`swapXtoY` 함수와 `swapYtoX` 함수를 public을 internal로 선언하는 것을 추천한다.

<br></br>
## 2. damon

https://github.com/gloomydumber/DEX_solidity/blob/master/src/Dex.sol
<br></br>
### 1. 잘못된 external visibility 사용

**설명**

Dex.sol/mint:94

```solidity
function mint(address _to) external returns (uint256 liquidity) {
```

`mint` 함수가 external로 선언되어 외부에서 누구나 호출이 가능하다. 따라서 유저는 무한으로 LP token을 minting하여 이를 유동성 풀의 token으로 바꿀 수 있다.  
<br></br>
**파급력**

Critical
<br></br>
**해결방안**

`mint` 함수를 internal로 선언해야 한다. 같은 맥락으로 `burn` 함수도 internal로 선언하여 유저가 직접 호출하는 일이 없게 하는 것이 좋다.

<br></br>
### 2. 잘못된 public visibility 사용

**설명**

Dex.sol/update:138

```solidity
function update(uint256 _balanceX, uint256 _balanceY) public {
```

마찬가지로 public이므로 `update` 함수를 누구나 호출할 수 있다. 따라서 악의적인 목적으로 reserveX, reserveY를 모두 0으로 만드는 등의 조작을 매우 쉽게 할 수 있다.
<br></br>
**파급력**

Critical
<br></br>
**해결방안**

`update` 함수를 internal 또는 private으로 선언해야 한다.

<br></br>
## 3. Muang

https://github.com/GODMuang/DEX_solidity/blob/main/src/Dex.sol
<br></br>
### 1. 잘못된 external visibility 사용

**설명**

Dex.sol/mint:82

```solidity
function mint(address to)public lock returns (uint256 liquidity) {
```

`mint` 함수가 external로 선언되어 외부에서 누구나 호출이 가능하다. 따라서 유저는 무한으로 LP token을 minting하여 이를 유동성 풀의 token으로 바꿀 수 있다.  

<br></br>
**파급력**

Critical
<br></br>
**해결방안**

`mint` 함수를 internal로 선언해야 한다. 같은 맥락으로 `burn` 함수도 internal로 선언하여 유저가 직접 호출하는 일이 없게 하는 것이 좋다.

<br></br>
### 2. 잘못된 public visibility 사용 및 approve 검사 미흡

**설명**

Dex.sol/transferFrom:144

```solidity
function transferFrom(address from, address to, uint256 value) public override returns (bool) {
    if (to == address(this))
        _transfer(from, to, value);
    else
        super.transferFrom(from, to, value);
    return true;
}
```

마찬가지로 public이므로 `transferFrom` 함수를 누구나 호출할 수 있다. 또한, `transferFrom` 함수 내에서 `to == address(this)` 를 만족하는 경우, from과 to에 대한 allowance를 검사, 소비하지 않는다. 따라서 악의적인 목적으로 공격하려는 주소를 from, Dex contract를 to로 하여 남의 LP token을 Dex contract로 전송할 수 있다. (Dex contract가 의도치 않게 걸어둔 backdoor라고도 볼 수 있을 것 같다.)
<br></br>
**파급력**

Critical
<br></br>
**해결방안**

`transferFrom` 함수를 internal 또는 private으로 선언해야 한다.

<br></br>
## 4. Nullorm

https://github.com/Null0RM/DEX_solidity/blob/main/src/Dex.sol
<br></br>
### 1. 잘못된 public visibility 사용 및 approve 검사 미흡

**설명**

Dex.sol/transferFrom:111

```solidity
function transferFrom(address from, address to, uint256 value) public override returns (bool) {
    if (to == address(this))
        _transfer(from, to, value);
    else
        super.transferFrom(from, to, value);
    return true;
}
```

마찬가지로 public이므로 `transferFrom` 함수를 누구나 호출할 수 있다. 또한, `transferFrom` 함수 내에서 `to == address(this)` 를 만족하는 경우, from과 to에 대한 allowance를 검사, 소비하지 않는다. 따라서 악의적인 목적으로 공격하려는 주소를 from, Dex contract를 to로 하여 남의 LP token을 Dex contract로 전송할 수 있다. (Dex contract가 의도치 않게 걸어둔 backdoor라고도 볼 수 있을 것 같다.)
<br></br>
**파급력**

Critical
<br></br>
**해결방안**

`transferFrom` 함수를 internal 또는 private으로 선언해야 한다.
<br></br>