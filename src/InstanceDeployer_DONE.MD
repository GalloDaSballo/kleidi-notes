# Q - I guess you need to `safe.execFromModule`

To use the Timelock to change config?


# Recovery Spells not part of ID

But they can only be added at that time

It is made safe by Hot Signers
But Hot signers could become comprimised
This puts a risk on new deplyments
But seems safe for already deployed stuff

Unless:
- We break the Spell Matches


# Goal

## Can we grief it, or misconfigure it in some way?

Beside hash mining which is technically possible but low likelihood

-------------------------------

Seems like it can be front-run

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/InstanceDeployer.sol#L179-L182

```solidity
        /// timelock salt is the result of the all params, so no one can
        /// front-run creation of the timelock with the same address on other
        /// chains
        instance.timelockParams.salt = bytes32(creationSalt);
```


# QA

## Validation / Sanitization on instance deployer?

Timelock duration being crazy high, etc..



# Front-run

## Can prob be used to grant more power to the deployer


# QA
## Threshold can result in different hash but same config - S

`instance.threshold` unused and instead uses 1

For safes with 1 owner

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/InstanceDeployer.sol#L308-L329

```solidity
            for (uint256 i = 1; i < instance.owners.length - 1; i++) {
                calls3[index++].callData = abi.encodeWithSelector(
                    OwnerManager.addOwnerWithThreshold.selector,
                    instance.owners[i],
                    1
                );
            }

            /// if there is only one owner, the threshold is set to 1
            /// if there are more than one owner, add the final owner with the
            /// updated threshold
            if (instance.owners.length > 1) {
                /// add final owner with the updated threshold
                /// if threshold is greater than the number of owners, that
                /// will be caught in the addOwnerWithThreshold function with
                /// error "GS201"
                calls3[index++].callData = abi.encodeWithSelector(
                    OwnerManager.addOwnerWithThreshold.selector,
                    instance.owners[instance.owners.length - 1],
                    instance.threshold
                );
            }
```

# Niptpicks

## Could use CONSTANT - S

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/InstanceDeployer.sol#L294-L302

```solidity
            calls3[index++].callData = abi.encodeWithSelector(
                OwnerManager.swapOwner.selector,
                /// previous owner
                address(1),
                /// old owner (this address)
                address(this),
                /// new owner, the first owner the caller wants to add
                instance.owners[0]
            );
```