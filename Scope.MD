- SafeProxyFactory

- /Deploy - 60% OK

- /Interface - SKIP

- /Utils - Create2 Stuff is fine (Checked against OZ) - FVd with Halmos, is good

- /View - TODO: Prob 1 hour

- BytesHelper
  - Maybe the slicing has some weird edge case I don't know about
  - Not much else IMO


---------------

# One more
- ConfigurablePause - 50%

# One more
- Guard - 50%


## Still thinking about front-runs
- InstanceDeployer


## 2 more gos
- RecoverySpell - 30%
- RecoverySpellFactory


- Timelock - 10%
- TimelockFactory


---------------


# Last Go

- RecoverySpellFactory - One quick last pass
- InstanceDeployer - One quick last pass

Overall I don't think there's any clash in InstanceDeployer, barring for the risk of hash mining (10 BLN dollars)

# TODO: THESE 2 One more time deep

- RecoverySpell
  Fundamental weakness is if they get disabled module
  Gas for owners
  Validation of owner (address(0))

- Timelock

Then one final wrap up
----

# TO RE CHECK

CREATE2 uses:   Salt + Contract Bytecode + Constructor Arguments

Meaning that if salt changes but arguments are the same (Gnosis Safe), you can have clashes on matching salt
But on the Timelock and the RecoverySpell, you can have clashes only if you also get the same args

-------------------------------------

Gnosis Safe Repo
- Check for ability to multicall
- Check for ability to set fallback module and behaviour
- 
  

# Reference Guard?
https://github.com/gnosisguild/zodiac-guard-scope/tree/main



Calldata manip
Ability to get bytes past the intended one?
Not exact match on length of data
Not sure if this causes issues