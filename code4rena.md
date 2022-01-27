# YETI Finance Findings
--------------------------------------------------------------------------------------------------------------------------------------------------------
C4 finding submitted:

# Handle

dalgarim


# Vulnerability details

## Impact
sYETIToken.sol mint funtion is public function and has no midifier or validr msg.sender check.
Anyone can mint an arbitrary amount of sYETI Tokens which in turn can impact the token economics.

## Proof of Concept
[mint function](https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/YETI/sYETIToken.sol#L174)
```
  function mint(uint256 amount) public returns (bool) {
      require(msg.sender != address(0), "Zero address");
      User memory user = users[msg.sender];

      uint256 shares = totalSupply == 0 ? amount : (amount * totalSupply) / effectiveYetiTokenBalance;
      user.balance += shares.to128();
      user.lockedUntil = (block.timestamp + LOCK_TIME).to128();
      users[msg.sender] = user;
      totalSupply += shares;

      yetiToken.sendToSYETI(msg.sender, amount);
      effectiveYetiTokenBalance = effectiveYetiTokenBalance.add(amount);

      emit Transfer(address(0), msg.sender, shares);
      return true;
  }
```

## Tools Used
Manual

## Recommended Mitigation Steps
Implement valid access control logic or make this function as internal.

--------------------------------------------------------------------------------------------------------------------------------------------------------

C4 finding submitted:

# Handle

dalgarim


# Vulnerability details

## Impact
The comment on the "StabilityPool.receiveCollateral" function states that this function should be called by ActivePool.
However this function doesn't implement access control which checks whether the caller is actually ActivePool or not.
As this function emit the "StabilityPoolBalancesUpdated" event, malicious user can contaminate events log by calling this function many times.

## Proof of Concept
[receiveCollateral](https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/StabilityPool.sol#L1143)
```
  // Should be called by ActivePool
  // __after__ collateral is transferred to this contract from Active Pool
  function receiveCollateral(address[] memory _tokens, uint256[] memory _amounts)
      external
      override
  {
      totalColl.amounts = _leftSumColls(totalColl, _tokens, _amounts);
      emit StabilityPoolBalancesUpdated(_tokens, _amounts);
  }
```

## Tools Used
Manual

## Recommended Mitigation Steps
Adding _requireCallerIsActivePool() on the function is required.
