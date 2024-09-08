## 1. jacqueline
commit version : a5981c8

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
commit version : 1671f2f

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
commit version : d99cfb5

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
commit version : 14fadc4

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
## 5. Ella
commit version : 1487df3

https://github.com/skskgus/Dex_solidity/blob/main/src/Dex.sol
<br></br>
### 1. Re-entrancy attack

**설명**

Dex.sol/transferFrom:111-135
```solidity
function swap(uint256 amount0Out, uint256 amount1Out, uint256 minOutput) external returns (uint256 amountOut) {
    require(amount0Out == 0 || amount1Out == 0, "Invalid input");

    bool isToken0 = amount1Out == 0;
    (IERC20 tokenIn, IERC20 tokenOut, uint256 reserveIn, uint256 reserveOut) = isToken0
        ? (token0, token1, reserve0, reserve1)
        : (token1, token0, reserve1, reserve0);

    uint256 amountIn = isToken0 ? amount0Out : amount1Out;

    uint256 balanceBefore = tokenIn.balanceOf(address(this));

    tokenIn.transferFrom(msg.sender, address(this), amountIn);

    uint256 actualAmountIn = tokenIn.balanceOf(address(this)) - balanceBefore;

    uint256 amountInWithFee = (actualAmountIn * 999) / 1000;
    amountOut = (reserveOut * amountInWithFee) / (reserveIn + amountInWithFee);

    require(amountOut >= minOutput, "Insufficient output amount");

    tokenOut.transfer(msg.sender, amountOut);

    _update();
}
```

다른 함수들과 달리 `swap` 함수는 맨 마지막에만 `_update` 함수를 호출한다. 여기서는 ERC20만을 사용하였지만 Cream finance와 같이 `transfer`함수를 통해 ERC777의 `tokenReceived` 함수와 같은 callback함수가 호출된다면 update되지 않은 reserve0, reserve1의 값으로 `swap` 함수를 다시 호출할 수 있다.
<br></br>
**파급력**

Informational
<br></br>
**해결방안**

당장의 취약점은 아니지만 나중에 문제가 될 수 있으므로 `_update` 함수를 `transfer` 함수를 호출하기 전에 호출하는 것을 추천한다.
<br></br>
## 6. Mia
commit version : 3dc4e18

https://github.com/ooMia/Upside_DEX_solidity/blob/main/src/Dex.sol
<br></br>
### 1. 전역변수 update 미흡

**설명**

Dex.sol/swapX:120-127
```solidity
function swapX(uint256 amountIn) internal returns (uint256 amount) {
    // in case of swap X -> Y
    amount = balanceY - (balanceX * balanceY) / (balanceX + amountIn);
    amount = (amount * 999) / 1000;

    tokenX.transferFrom(msg.sender, address(this), amountIn);
    tokenY.transfer(msg.sender, amount);
}
```

swap을 한 뒤, 전역변수인 balanceX를 update하지 않는다. 따라서 swap을 여러 번 해도 같은 양의 토큰을 얻을 수 있으므로 일방적인 이득을 볼 수 있다.
<br></br>
**파급력**

Critical
<br></br>
**해결방안**

`swapX` 함수에서 balanceX를 swap한 양만큼 update해줘야 한다. 같은 맥락으로 `swapY` 함수에서 balanceY를 update해줘야 한다.

<br></br>
## 7. kaymin
commit version : 1b37751

https://github.com/kaymin128/Dex_solidity/blob/main/src/Dex.sol
<br></br>
### 1. 전역변수 update 미흡

**설명**
Dex.sol/addLiquidity:27
```solidity
function addLiquidity(uint amount_x, uint amount_y, uint min_liquidity) external returns (uint liquidity) {
    require(amount_x > 0 && amount_y > 0, "Invalid input amounts");
    require(token_x.allowance(msg.sender, address(this)) >= amount_x, "ERC20: insufficient allowance");
    require(token_y.allowance(msg.sender, address(this)) >= amount_y, "ERC20: insufficient allowance");
    require(token_x.balanceOf(msg.sender) >= amount_x, "ERC20: transfer amount exceeds balance");
    require(token_y.balanceOf(msg.sender) >= amount_y, "ERC20: transfer amount exceeds balance");
    
    reserve_x = token_x.balanceOf(address(this));

    if (total_supply == 0) {
        liquidity = sqrt(amount_x * amount_y);
    } else {
        liquidity = min(amount_x * total_supply / reserve_x, amount_y * total_supply / reserve_y);
    }

    require(liquidity >= min_liquidity, "Dex: insufficient liquidity minted");

    balanceOf[msg.sender] += liquidity; 
    total_supply += liquidity; 
    
    reserve_x += amount_x; 
    reserve_y += amount_y; 

    require(token_x.transferFrom(msg.sender, address(this), amount_x), "Dex: transfer of token_x failed");
    require(token_y.transferFrom(msg.sender, address(this), amount_y), "Dex: transfer of token_y failed");
    return liquidity;
}
```

`addLiquidity` 함수를 호출할 때, reserve_y를 가지고 오지 않는다. 하지만 reserve_y가 정상적으로 update가 되기 때문에 큰 문제가 되지 않는다.

<br></br>
**파급력**

Informational
<br></br>

**해결방안**

reserve_x를 가져오는 부분을 제거하거나 reserve_y를 가져오는 로직을 추가하는 것을 추천한다.

<br></br>
## 8. bob
commit version : 973f72b

https://github.com/choihs0457/DEX_solidity/blob/main/src/Dex.sol
<br></br>
### 1. 외부 transfer 고려

**설명**
Dex.sol/updateLiquidity:20-27
```solidity
modifier updateLiquidity() {
    uint256 currentBalanceX = tokenX.balanceOf(address(this));
    uint256 currentBalanceY = tokenY.balanceOf(address(this));
    
    liquidityX = currentBalanceX;
    liquidityY = currentBalanceY;
    _;
}
```

`updateLiquidity` modifier에서 전역변수인 liquidityX, liquidityY를 update하는 과정에서 외부 transfer가 발생하면 기대했던 값과 다르게 update되어 두 토큰 간의 비율이 깨질 수 있다.

<br></br>
**파급력**

Medium
<br></br>

**해결방안**

외부 transfer을 검사하는 로직이 추가되어야 한다. (다른 함수로 인해 변화한 값을 저장하는 변수를 선언하여 currentBalanceX와 일치하는지 확인하는 등)

<br></br>
## 9. jeremy
commit version : 3a5de86

https://github.com/WOOSIK-jeremy/DEX_solidity/blob/main/src/Dex.sol
<br></br>
### 1. 조기 검사

**설명**
Dex.sol/swap:89
```solidity
require(currentX == 0 || currentY == 0);
```

만약 currentX, currentY가 모두 0이면 바로 revert를 시키지 않고 나머지 로직을 모두 수행한다.

<br></br>
**파급력**

Informational
<br></br>

**해결방안**

currentX, currentY가 모두 0이면 바로 revert시키는 로직을 추가하는 것을 추천한다.
<br></br>
### 2. 외부 transfer 고려

**설명**

`swap`, `addLiquidity`, `removeLiquidity` 함수를 호출할 때 두 토큰의 양을 balanceOf로 가져온다. 외부 transfer가 발생하면 기대했던 값과 다르게 가져와져서 두 토큰 간의 비율이 깨질 수 있다.

<br></br>
**파급력**

Medium
<br></br>

**해결방안**

외부 transfer을 검사하는 로직이 추가되어야 한다. (다른 함수로 인해 변화한 값을 저장하는 변수를 선언하는 등)

<br></br>
## 10. teddy
commit version : 3488bc3

https://github.com/gdh8230/DEX_solidity/blob/main/src/Dex.sol
<br></br>
### 1. 외부 transfer 고려

**설명**

두 토큰의 양은 balanceOf, LP token의 양은 `totalSupply` 함수를 호출하여 가져온다. 외부 transfer가 발생하면 기대했던 값과 다르게 가져와져서 두 토큰 간의 비율이 깨질 수 있다. 또한, 많은 양의 LP token을 누군가 transfer하면 다른 유저들이 `addLiquidity`, `removeLiquidity` 함수를 호출할 때 손해를 보게 된다.

<br></br>
**파급력**

Medium
<br></br>

**해결방안**

외부 transfer을 검사하는 로직이 추가되어야 한다. (다른 함수로 인해 변화한 값을 저장하는 변수를 선언하는 등)
