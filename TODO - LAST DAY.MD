# LAST THING

# Pokemon chart of Malicious Attacks

## Timelock can remove spells?
## Spells can remove timelock?
## Pauser ?

--------

Malicious Signers
-> Pause
-> Execute Recovery
-> If Pause is short and spell takes too long, signers are compromised, this is over
-> If pause + timelock > spell -> Spell wins -> good
-> If Pause + timelock < spell -> Timelock wins -> Bad

Malicious Spell
-> If Timelock > spell -> Spell wins -> Bad
-> If Timelock < spell -> Timelock wins -> Good

Malicious Pause
-> Cannot really do much, they need to collude with one or the other

ok lg

------



# TODO - DONE

## Can this flag cause issues? - Looks fine - SAFE
Fundamentally they iterate from the start of the list, popping one item at a time
They then rebuild the list via adding which ignores the SENTINEL, since it relies on it

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/RecoverySpell.sol#L262-L263

```solidity
                SENTINEL, /// @audit Is this valid or could this be made to disable the spells

```


# TO SEND THIS SHIT


# TODO - Aggregate the various edge cases - Arbitrary caller in `execute` may cause damage
## QA - Self Rekt with permissioneless `execute` - DONE

`expirationPeriod` very long

wrong value to remove a certain hash

that is pending

They add the check later

It get's removed

### Self-Rekt - Desynch remove of index due to `execute`

All these index based checks must be done in a batch


# Self-Rekt - TOSEND . DONE
## Self-Rekt removing all hashes = Can no longer perform checks

Add Wildcard
With a bunch of bullshit hashes
In the issues I think




