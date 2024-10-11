# Analysis

You can revoke the hot signer during pause and outside of pause


Other roles can be given but technically they don't do much

You may be able to grant role admin to someone else, not super sure

But you can always `_revokeRole` the hot signers even once you lost the admin role

# Hunch

## TODO - Aggregate the various edge cases - Arbitrary caller in `execute` may cause damage




# Analysis

My understanding of `checkCalldata`

It requires that a check exist
The check may do nothing (same indexes = skip)

It's effectively both a whitelist and a validation tool

## How does this work?

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/Timelock.sol#L506-L514

```solidity
            require( /// @audit TODO?? Example?
                calldataCheck.dataHashes.contains(
                    data.getSlicedBytesHash(
                        calldataCheck.startIndex, calldataCheck.endIndex
                    )
                ),
                "CalldataList: Calldata does not match expected value"
            );
        }
```

# QA 

## Ignoring fallback

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/Timelock.sol#L1074-L1077

```solidity
        require(selector != bytes4(0), "CalldataList: Selector cannot be empty");
        require(
            startIndex >= 4, "CalldataList: Start index must be greater than 3"
        );
```

## Event Data

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/Timelock.sol#L1198-L1199

```solidity
            indexes[targetIndex].dataHashes.values() /// @audit technically adding all, also technically will emit a hash not the actual value

```

# QA - Re-exec not possible but reentrancy is - TO SEND

Because of isReadyTS

Which prevents re-exec


However, reentrancy is possible for other calls

This should generally be safe barring doing MEV risky operations




# QA

## This should be `DONE_TIMESTAMP` - S

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/Timelock.sol#L430-L431

```solidity
        require(timestamp != 1, "Timelock: operation already executed");

```

## Typo -S

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/Timelock.sol#L1104-L1105

```solidity
                "CalldataList: End index eqauls start index only when 4"

```


## Can be removed and done off chain for simplicity - S

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/Timelock.sol#L1166-L1177

```solidity
            if (isSelfAddressCheck[i]) {
                /// self address check, data must be empty
                require(
                    data[i].length == 0,
                    "CalldataList: Data must be empty for self address check"
                );
                require(
                    endIndex - startIndex == 20,
                    "CalldataList: Self address check must be 20 bytes"
                );
                dataHash = ADDRESS_THIS_HASH;
            } else {
```

The hash is known at time of setup, so there's no advantage in having this code


# Refactoring Analysis -S
 
Fundamentally:
You either accept all (wildcard)
Or you check some
And checking some is tied to exact matches on some pieces

Each bytes check has a list of values afaict

So there is prob some risk there
But also not sure if it can be avoidable


In TS
Check {
    isWildcard: boolean,
    actualChecks: Check[]
}

Where each check is the segmented check


Start
End
Self Address Check
Data


# Fundamentaly Property

Hash(data)

Hashing concatenation is consistent and can be specified given
Bytes + data + length
Meaning it doesn't matter where the word is, the word hash will be exclusively defined by the length of the bytes and the data










# Gas Crap - S

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/Timelock.sol#L469-L472

```solidity
        Index[] storage indexes = _calldataList[contractAddress][selector];

        indexDatas = new IndexData[](indexes.length);
        for (uint256 i = 0; i < indexes.length; i++) {
```

indexes.length -> As memory var

Maybe no longer necessary


# Encoding (UI risk)  - S

This test fails

```solidity
    bytes wrongCheck = "a54D3c09E34aC96807c1CC397404bF2B98DC4eFb";
    bytes rightCheck = "a54d3c09E34aC96807c1CC397404bF2B98DC4eFb";

    function test_bytes_checksum() public {
        bytes32 kak1 = keccak256(wrongCheck);
        bytes32 kak2 = keccak256(rightCheck);

        assertEq(kak1, kak2, "same res");
    }
```

This doesn't
```solidity
    bytes wrongCheck = hex"a54D3c09E34aC96807c1CC397404bF2B98DC4eFb";
    bytes rightCheck = hex"a54d3c09E34aC96807c1CC397404bF2B98DC4eFb";

    function test_bytes_checksum() public {
        bytes32 kak1 = keccak256(wrongCheck);
        bytes32 kak2 = keccak256(rightCheck);

        assertEq(kak1, kak2, "same res");
    }
```

Because the first one is converting the bytes to literals
While the second one is converting them from hex, which is consistent with encodePacked values


# Calldata check removal could remove wildcard by mistake - S

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/Timelock.sol#L1062-L1112

```solidity
    function _addCalldataCheck(
        address contractAddress,
        bytes4 selector,
        uint16 startIndex,
        uint16 endIndex,
        bytes[] memory data,
        bool[] memory isSelfAddressCheck
    ) private {
        require(
            contractAddress != address(0),
            "CalldataList: Address cannot be zero"
        );
        require(selector != bytes4(0), "CalldataList: Selector cannot be empty");
        require(
            startIndex >= 4, "CalldataList: Start index must be greater than 3"
        );
        require(
            data.length == isSelfAddressCheck.length,
            "CalldataList: Array lengths must be equal"
        );
        /// prevent misconfiguration where a hot signer could change timelock
        /// or safe parameters
        require(
            contractAddress != address(this),
            "CalldataList: Address cannot be this"
        );
        require(contractAddress != safe, "CalldataList: Address cannot be safe");

        Index[] storage calldataChecks =
            _calldataList[contractAddress][selector];
        uint256 listLength = calldataChecks.length;
        /// @audit If you add wildcard, you need to remove it before adding something else
        if (listLength == 1) {
            require(
                calldataChecks[0].startIndex != calldataChecks[0].endIndex,
                "CalldataList: Cannot add check with wildcard"
            );
        }

        if (startIndex == endIndex) {
            require(
                startIndex == 4,
                "CalldataList: End index eqauls start index only when 4"
            );
            require(
                listLength == 0,
                "CalldataList: Add wildcard only if no existing check"
            );

            /// @audit Add `data.length == 0` so you ensure this check is correct
        } else {
```

A Wildcard Could be added with Data by mistakes due to a lack of this check

In removing the dataHash, the wildcard would also be removed by `removeCalldataCheckDatahash`:

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/Timelock.sol#L903-L908

```solidity
    function removeCalldataCheckDatahash(
        address contractAddress,
        bytes4 selector,
        uint256 index,
        bytes32 dataHash
    ) external onlyTimelock {
```

### Mitigation

Prevent wildcard checks to have dataHashes
Prevent removing dataHashes from wildcard checks
Enforce that a wildcard can only be added or removed


# Analysis of very small config / coding NITs

## Max Delay unset so a tx can be schedule in an unbounded way

## Delay is stored and used, meaning that if changed it won't apply to previous changes

I think this is good

## Expiration period change may prevent clearning an expired operations

Exec can happen on ready operations
An expired operation is ready 
And can be executed
Clearing vs execution will be up to the choice of the actor performing the operation




## Prob implict check - S

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/Timelock.sol#L307-L308

```solidity
        /// set the admin of the hot signer role to the default admin role
        _setRoleAdmin(HOT_SIGNER_ROLE, DEFAULT_ADMIN_ROLE); /// @audit prob implict
```

Since DEFAULT_ADMIN_ROLE == 0x00


## Overall best to remove on all of these - DONE
https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/Timelock.sol#L774-L806

```solidity
    function grantRole(bytes32 role, address account)
        public
        override(AccessControl, IAccessControl)
    {
        require(role != DEFAULT_ADMIN_ROLE, "Timelock: cannot grant admin role");
        super.grantRole(role, account);
    }

    /// @notice function to revoke a role from an address
    /// @param role the role to revoke
    /// @param account the address to revoke the role from
    function revokeRole(bytes32 role, address account)
        public
        override(AccessControl, IAccessControl)
    {
        require(
            role != DEFAULT_ADMIN_ROLE, "Timelock: cannot revoke admin role"
        );
        super.revokeRole(role, account);
    }

    /// @notice function to renounce a role
    /// @param role the role to renounce
    /// @param account the address to renounce the role from
    function renounceRole(bytes32 role, address account)
        public
        override(AccessControl, IAccessControl)
    {
        require(
            role != DEFAULT_ADMIN_ROLE, "Timelock: cannot renounce admin role"
        );
        super.renounceRole(role, account);
    }
```