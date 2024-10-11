# Analysis - 20%

Overall looks safe, maybe need to triple check the tStore, tLoad flow

---

Recovery Spells are deployed deterministically

They must be added on deploy

Technically they can be added later (as module via Timelock)

Adding them doesn't influence the address, but they can only be added by the Hot Signer
There doesn't seem to be any enforcement on the owners of the spell vs the values

Owners imo should be from the safe at that time
Having the owners hardcoded looks dangerous


# To check

- Sanity on Owners vs Threshold (Is this actually enforced?)
- Owner relation to real owners is unclear | undefined and overlapping



Owner gas optimizations
Also prob doesn't need to use tStore -> Can just read and cache in memory once


# QA

## Technically there's a risk of clash - S

But zero advantage with deploying the Spell

I think deploying the spells makes them more deterministic as it would enforce the settings

Deploying them latest could open up to abusing a clash

## The spell can be reverse engineered

Maybe


# TODO | No module guard
https://github.com/safe-global/safe-smart-account/blob/main/contracts/Safe.sol


 *      - Guards: Guards are contracts that can execute pre- and post- transaction checks. There are two types of guards:
 *          1. Transaction Guard: managed in `GuardManager` for transactions executed with `execTransaction`.
 *          2. Module Guard: managed in `ModuleManager` for transactions executed with `execTransactionFromModule`
*      



# Gas - Set it to 0 to save gas due to refund - S

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/RecoverySpell.sol#L302

```solidity
        recoveryInitiated = type(uint256).max;
```
There is no zero timestamp so this is a safe change



# New Notes

## Only applies to benign spells - S

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/RecoverySpell.sol#L112-L117

```solidity
        /// no checks on parameters as all valid recovery spells are
        /// deployed from the factory which will not allow a recovery
        /// spell to be created that does not have valid parameters.
        /// A recovery spell can only be created by the factory if the Safe has
        /// already been created on the chain the RecoverySpell is being
        /// deployed on.
```

Malicious spells can be deployed anywhere by the compromised hot signer


