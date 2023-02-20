[![Tests & Lint](https://github.com/Elnaril/uniswap-universal-router-decoder/actions/workflows/tests.yml/badge.svg)](https://github.com/Elnaril/uniswap-universal-router-decoder/actions/workflows/tests.yml)
[![Test Coverage](https://img.shields.io/badge/dynamic/json?color=blueviolet&label=coverage&query=%24.totals.percent_covered_display&suffix=%25&url=https%3A%2F%2Fraw.githubusercontent.com%2FElnaril%2Funiswap-universal-router-decoder%2Fmaster%2Fcoverage.json)](https://github.com/Elnaril/uniswap-universal-router-decoder/blob/master/coverage.json)
[![GitHub release (latest by date)](https://img.shields.io/github/v/release/Elnaril/uniswap-universal-router-decoder)](https://github.com/Elnaril/uniswap-universal-router-decoder/releases)
[![PyPi Repository](https://img.shields.io/badge/repository-pipy.org-blue)](https://pypi.org/project/uniswap-universal-router-decoder/)
[![GitHub](https://img.shields.io/github/license/Elnaril/uniswap-universal-router-decoder)](https://github.com/Elnaril/uniswap-universal-router-decoder/blob/master/LICENSE)
[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/uniswap-universal-router-decoder)](https://pypi.org/project/uniswap-universal-router-decoder/)
[![Imports: isort](https://img.shields.io/badge/%20imports-isort-%231674b1?style=flat&labelColor=ef8336)](https://pycqa.github.io/isort/)
[![Type Checker: mypy](https://img.shields.io/badge/%20type%20checker-mypy-%231674b1?style=flat&labelColor=ef8336)](https://mypy-lang.org/)
[![Linter: flake8](https://img.shields.io/badge/%20linter-flake8-%231674b1?style=flat&labelColor=ef8336)](https://flake8.pycqa.org/en/latest/)

# Uniswap Universal Router Decoder & Encoder

## Description

The object of this library is to decode & encode the transaction input sent to the Uniswap universal router (UR)
(address [`0xEf1c6E67703c7BD7107eed8303Fbe6EC2554BF6B`](https://etherscan.io/address/0xEf1c6E67703c7BD7107eed8303Fbe6EC2554BF6B) 
on Ethereum Mainnet).

⚠ This library has not been audited, so use at your own risk !

⚠ There is no guarantee of compatibility between 2 versions: consider forcing the version in your dependency requirements.

⚠ This project is a work in progress so not all commands are decoded yet. Below a list of the already implemented ones:


| Command Id | Function Name | Decode | Encode
| ---------- | ------------- |:------:|:------:
| 0x00 | V3_SWAP_EXACT_IN | ✅ | ❌
| 0x01 | V3_SWAP_EXACT_OUT | ✅ | ❌
| 0x02 - 0x06 |  | ❌ | ❌
| 0x07 | placeholder  | N/A | N/A
| 0x08 | V2_SWAP_EXACT_IN | ✅ | ✅
| 0x09 | V2_SWAP_EXACT_OUT | ✅ | ❌
| 0x0a | PERMIT2_PERMIT | ✅ | ❌
| 0x0b | WRAP_ETH | ✅ | ✅
| 0x0c | UNWRAP_WETH | ✅ | ❌
| 0x0d | PERMIT2_TRANSFER_FROM_BATCH | ❌ | ❌
| 0x0e - 0x0f | placeholders | N/A | N/A
| 0x10 - 0x1d |  | ❌ | ❌
| 0x1e - 0x3f | placeholders | N/A | N/A

## Installation

```bash
# update pip to latest version if needed
pip install -U pip

# install the decoder from pypi.org
pip install uniswap-universal-router-decoder
```

## Usage

The library exposes a class, `RouterDecoder` with several public methods that can be used to decode or encode UR data.

### How to decode a transaction input
To decode a transaction input, use the `decode_function_input` method as follow:

```python
from uniswap_universal_router_decoder.router_decoder import RouterDecoder

trx_input = "0x3593564c000000000000000000 ... 90095b5c4e9f5845bba"  # the trx input to decode
decoder = RouterDecoder()
decoded_trx_input = decoder.decode_function_input(trx_input)
```

### How to decode a transaction
It's also possible to decode the whole transaction, given its hash 
and providing the decoder has been built with either a valid Web3 instance or the link to a rpc endpoint:

```python
# Using a web3 instance
from web3 import Web3
from uniswap_universal_router_decoder.router_decoder import RouterDecoder
w3 = Web3(...)  # your web3 instance
decoder = RouterDecoder(w3=w3)
```

```python
# Using a rpc endpoint
from web3 import Web3
from uniswap_universal_router_decoder.router_decoder import RouterDecoder
rpc_link = "https://..."  # your rpc enpoint
decoder = RouterDecoder(rpc_endpoint=rpc_link)
```

And then the decoder will get the transaction from the blockchain and decode it, along with its input data:
```python
# Using a rpc endpoint
trx_hash = "0x52e63b7 ... 11b979dd9"
decoded_transaction = decoder.decode_transaction(trx_hash)
```

### How to decode an Uniswap V3 swap path
The `RouterDecoder` class exposes also the static method `decode_v3_path` which can be used to decode a given Uniswap V3 path.

```python
from uniswap_universal_router_decoder.router_decoder import RouterDecoder

uniswap_v3_path = b"\xc0*\xaa9\xb2#\xfe\x8d\n\x0e ... \xd7\x89"  # bytes or str hex
fn_name = "V3_SWAP_EXACT_IN"  # Or V3_SWAP_EXACT_OUT
decoded_path = RouterDecoder.decode_v3_path(fn_name, uniswap_v3_path)
```
The result is a tuple, starting with the "in-token" and ending with the "out-token", with the pool fees between each pair.


### How to encode a call to the function WRAP_ETH
This function can be used to convert eth to weth using the UR.
```python
from uniswap_universal_router_decoder.router_decoder import RouterDecoder

decoder = RouterDecoder()
encoded_data = decoder.encode_data_for_wrap_eth(amount_in_wei)  # to convert amount_in_wei eth to weth

# then in your transaction dict:
transaction["data"] = encoded_data

# you can now sign and send the transaction to the UR
```


### How to encode a call to the function V2_SWAP_EXACT_IN
This function can be used to swap tokens. Correct allowances must have been set before using sending such transaction.
```python
from uniswap_universal_router_decoder.router_decoder import RouterDecoder

decoder = RouterDecoder()
encoded_data = decoder.encode_data_for_v2_swap_exact_in(
        amount_in,  # in Wei
        min_amount_out,  # in Wei
        [
            in_token_address,
            out_token_address,
        ],
        timestamp,  # unix timestamp after which the trx will not be valid any more
    )

# then in your transaction dict:
transaction["data"] = encoded_data

# you can now sign and send the transaction to the UR
```
