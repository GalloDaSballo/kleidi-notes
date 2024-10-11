# Analysis - DONE

Fundamentally enforces that no delegate call can be called from the safe

It can still be called via the Modules and MAYBE via the fallback module

Seems to be upgrade resistant and I don't believe upgrades are "intended"

You cannot guarantee the safe will be empty
But the behaviour of the safe is pretty much normal when it comes to executing operations


This seems to remove the ability to batch operations because batches are done via delegatecalls

# QA

## Admin sidestep

Delegatecall could still be performed by setting up modules

AFAICT this is still allowd by the Timelock

IMO this is fine, however, signers should be warned that adding a module could cause a full sidestep of the security guarantees of the project

The same can happen via a fallback handler

# Hunch - Takeover via malicious fallback handler?

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/InstanceDeployer.sol#L155-L156

```solidity
            /// no fallback handler allowed by Guard
            address(0),
```

-> Front-run
-> Deploy the guard
-> Shrekt?

# NITS?

## Safe flow for owners, and other configs


## Technically cannot guarantee this

Some funds could be there and may even be used or active

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/Guard.sol#L29-L30

```solidity
/// Refund receiver and gas params are not checked because the Safe itself
/// does not hold funds or tokens.
```

## Fallback missing

Likelihood of this being an issue is extremely low
Also since upgrades are not contemplated this seems fine as is



# Notes

## Self Calls via

https://github.com/safe-global/safe-smart-account/blob/0142ec8a4a05f03167daba9e7231b7e858aabd32/contracts/base/ModuleManager.sol#L154-L163

```solidity
    function execTransactionFromModule(
        address to,
        uint256 value,
        bytes memory data,
        Enum.Operation operation
    ) external override returns (bool success) {
        (address guard, bytes32 guardHash) = preModuleExecution(to, value, data, operation);
        success = execute(to, value, data, operation, type(uint256).max);
        postModuleExecution(guard, guardHash, success);
    }
```

So timelock can still change settings since it's a module


# Pokemon chart of types

## Timelock can remove spells?
## Spells can remove timelock?
## Pauser ?