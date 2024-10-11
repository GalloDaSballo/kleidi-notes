# Areas of Concern



# Big concern tied to calldata manips

- Managing the calldata that needed to be stored in the timelock

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/Timelock.sol#L1061-L1069

```solidity

    function _addCalldataCheck(
        address contractAddress,
        bytes4 selector,
        uint16 startIndex,
        uint16 endIndex,
        bytes[] memory data,
        bool[] memory isSelfAddressCheck
    ) private {
```



# Hot signers
- Only predefined actions, except on construction
- Scoped down

# Cold signers
- Control the safe
- Usual safe stuff

Adds Guard
- Guard prevents from changing selfcalls and delegatecalls

The assets go on the timelock

Hot signers can do actions
Cold signers can do anything via timelock + config what hot can do


--------

# Emergency withdrawal scenario

Tap -> Locking

Locking Hot signers

Lock is on the timelock

Emergency is called

How do they withdraw?

Cold -> Go to Safe -> Send to Timelock -> Wait for Delay -> Then anyone can execute

# Alternative Scenario | False alarm scenario

Signers say "we need to withdraw"

Then emergency is cancelled

Need to undo / Remove -> Cancel -> Safe Signers

--------

# Fast Withdrawal scenario

- Allow withdrawal for hot signers
- Ensure to the stuff


# Known issue
-> Loss due to slippage / emergency withdrawal



# You can only pause
Once the duration has gone through it auto-unpause

This is to prevent / delay compromises


Pause Guardian = Pause all proposals
10 day wait period (or w/e)


RecoverySpell -> Notifs
-> Could be vetoed

---
Trust signers
vs
Trust pauses
---
You need to trust one of them