## Hot Signer that later gets compromised - S

Original malicious / compromised hot signer can cause ruckus later

The original hot signers can deploy an instance of the contract meaning they retain this power indefinitely

After some time, one of the hot signers could be compromised and they would still retain the ability to deploy compromised setups in other chains

This means that any deployer should consider deploying their instance early and then remove the hot signers and then add them again

It may be slightly safer to enforce that one of the callers is an owner of the safe | TODO

-------