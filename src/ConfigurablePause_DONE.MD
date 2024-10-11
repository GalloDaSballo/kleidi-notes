# Analysis - DONE

Implicit invariant:
- When pause no normal operations can be performed
- You cannot queue new operations during pause

An operation can be executed at most after PAUSE_END_TIME + Delay from the timelock
With the exception of `revokeHotSigner`

Can remove hot signers:
https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/Timelock.sol#L810-L815

```solidity
    /// @notice function to revoke the hot signer role from an address
    /// can only be called by the timelock or the safe
    /// @param deprecatedHotSigner the address of the hot signer to revoke
    function revokeHotSigner(address deprecatedHotSigner) external onlySafe {
        _revokeRole(HOT_SIGNER_ROLE, deprecatedHotSigner);
    }
```

Removes guardian (can only pause once)

# Gas -S

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/ConfigurablePause.sol#L15-L23

```solidity

    /// @notice pause start time, starts at 0 so contract is unpaused
    uint128 public pauseStartTime;

    /// @notice pause duration
    uint128 public pauseDuration;

    /// @notice address of the pause guardian
    address public pauseGuardian;
```

u48 is plenty and will allow one slot for all 3 values


# Info

## Not sure it makes sense to index this - D

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/ConfigurablePause.sol#L46-L47

```solidity
    event PauseTimeUpdated(uint256 indexed newPauseStartTime);

```

That would be helpful to find all events at a time, but that's a pretty niche thing to do


Same here

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/ConfigurablePause.sol#L51-L54

```solidity
    event PauseDurationUpdated(
        uint256 indexed oldPauseDuration, uint256 newPauseDuration
    );

```

## Grant Guardian is setting to 0 as well, meaning it's also a revoke

`setGuardian` seems to be most appropriate