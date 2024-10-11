# Fallback? - TODO

https://github.com/solidity-labs-io/kleidi/blob/1a06ac16bc99d0b4081281329d03064c3737f5e4/src/BytesHelper.sol#L7-L17

```solidity
    function getFunctionSignature(bytes memory toSlice)
        public
        pure
        returns (bytes4 functionSignature)
    {
        require(toSlice.length >= 4, "No function signature");

        assembly ("memory-safe") {
            functionSignature := mload(add(toSlice, 0x20))
        }
    }
```



