*P2QRH Related Changes to Bitcoin Core*


P2QRH functionality in Bitcoin Core spans several modules, including script execution, consensus rules, transaction validation, wallet support, and networking.

## 1. Script Execution and Opcodes (src/script/)

The scripting engine is central to P2QRH, as it will handle the validation of P2QRH outputs and execution.


- **src/script/interpreter.cpp**:
  - `VerifyWitnessProgram`: Validates witness programs including P2QRH
  - `ExecuteWitnessScript`: Executes P2QRH script path spends
  - `ComputeTapleafHash`: Computes leaf hashes for Taproot scripts.  Should leverage as is.
  - `ComputeTapbranchHash`: Computes branch hashes for Taproot Merkle trees. Should leverage as is.

**Key Functions:**
- `CheckSchnorrSignature`: Validates Schnorr signatures for Taproot
- `SignatureHashSchnorr`: Computes signature hashes for Taproot spends
- `VerifyTaprootCommitment`: Validates Taproot output commitments

- **src/script/script.h**: Defines script opcodes and constants, including those modified or introduced for Tapscript (e.g., OP_CHECKSIGADD).
- **src/script/signature_checker.h** and **src/script/signature_checker.cpp**: Implement signature verification logic, extended for Schnorr signatures (BIP-340) and Taproot-specific checks.

**Key Classes:**
- `BaseSignatureChecker`: Abstract base class for signature verification, extended for Taproot.
- `TransactionSignatureChecker`: Handles transaction-specific signature checks for P2QRH.


## 2. Cryptographic Library (src/secp256k1/)

The following is specific to P2TR using Schnorr.
The equivalent needs to be implemented for PQC algorithms implemented in [libbitcoinpqc](https://github.com/cryptoquick/libbitcoinpqc/tree/main). 


- **src/secp256k1/src/schnorr.c**: Implements Schnorr signature algorithms (signing and verification) per BIP-340.
- **src/secp256k1/include/secp256k1_schnorr.h**: Defines the API for Schnorr signatures.
- **src/secp256k1/src/modules/taproot/**:
  - `secp256k1_taproot_tweak_pubkey`: Tweaks public keys for Taproot
  - `secp256k1_taproot_tweak_seckey`: Tweaks private keys for Taproot
  - `secp256k1_taproot_leaf_hash`: Computes Taproot leaf hashes
  - `secp256k1_taproot_compute_merkle_root`: Computes Merkle roots

- **src/secp256k1/src/schnorr_impl.h**:
  - Implements core Schnorr signature algorithms
  - Provides batch verification functionality


## 3. Consensus Rules (src/consensus/ and src/validation/)


- **src/consensus/consensus.h**: Defines consensus parameters, including Taproot activation height.
- **src/consensus/tx_verify.cpp**: Contains transaction verification logic.

**Key Functions:**
- `CheckTransaction`: Ensures P2QRH transactions comply with consensus rules.

- **src/validation.cpp**: Handles block validation.

**Key Functions:**
- `CheckBlock`: Validates blocks containing P2QRH transactions.
- `ContextualCheckBlock`: Ensures Taproot activation rules are respected.

- **src/policy/policy.cpp**: Defines standardness rules for P2QRH transactions.

**Key Functions:**
- `IsStandard`: Checks if P2QRH transactions are relayable by nodes.

- **src/validation.cpp**:
  - `CheckTaproot`: Validates Taproot consensus rules
  - `VerifyWitnessProgram`: Validates witness v1 programs
  - `CheckScriptFlags`: Enforces Taproot script validation flags

**Key Functions:**
- `AcceptToMemoryPool`: Validates Taproot transaction acceptance
- `CheckInputs`: Verifies Taproot input scripts

## 4. Wallet Support (src/wallet/)


- **src/wallet/wallet.cpp**: Core wallet logic for Taproot
  - `SignTransaction`: Signs transactions including Taproot inputs
  - `FillPSBT`: Fills and signs PSBTs with Taproot data
  - `TransactionChangeType`: Handles Taproot change output creation

- **src/wallet/scriptpubkeyman.cpp**: 
  - Manages Taproot output types via `DescriptorScriptPubKeyMan`
  - Handles Taproot address generation and key management

- **src/wallet/spend.cpp**:
  - `CreateTransaction`: Creates Taproot transactions
  - Implements coin selection for Taproot outputs

## 5. Address Encoding


- **src/key_io.cpp**:
  - `EncodeBech32`: Implements Bech32m encoding for Taproot addresses
  - `DecodeBech32`: Parses Bech32m-encoded Taproot addresses
  - `GetDestinationForKey`: Creates Taproot destinations from keys

- **src/script/standard.cpp**:
  - `IsValidDestination`: Validates Taproot address formats
  - `GetScriptForDestination`: Creates scripts for Taproot addresses

- **src/core_io.cpp**:
  - Handles serialization of Taproot addresses and scripts
  - Implements Taproot address formatting

## 6. Networking and P2P

- **src/validation.cpp**:
  - `AcceptToMemoryPool`: Validates and accepts Taproot transactions
  - `PrecomputedTransactionData`: Handles Taproot signature verification data
  - `CheckTxInputs`: Validates Taproot input scripts

- **src/net_processing.cpp**:
  - `PeerManagerImpl::ProcessMessage`: Processes Taproot transactions
  - `BroadcastTransaction`: Relays Taproot transactions
  - `RelayTransaction`: Handles transaction relay policies

- **src/policy/policy.cpp**:
  - `IsStandardTx`: Enforces Taproot standardness rules
  - `IsWitnessStandard`: Validates Taproot witness structures

## 7. Descriptor Wallet Support

- **src/descriptor/descriptor.cpp**:
  - should implement `qrh()` and `rawqrh()` descriptors
  - Manages script and spending policies

- **src/wallet/scriptpubkeyman.cpp**:
  - `DescriptorScriptPubKeyMan`: Manages Taproot descriptors
  - Handles key generation and derivation
  - Implements address creation from descriptors

- **src/wallet/wallet.cpp**:
  - Integrates descriptor wallet support with Taproot
  - Manages descriptor-based wallet operations

## 8. p2qrh related rpc functions

| Functional Group | Method | Notes |
|-----------------|---------|-------|
| Descriptor Operations | deriveaddresses | Supports deriving addresses from qrh() descriptors for P2QRH outputs |
| | getdescriptorinfo | Analyzes and validates tr() descriptors for P2QRH outputs |
| | importdescriptors | Enables importing P2QRH descriptors into wallet |
| | listdescriptors | Shows all descriptors including P2QRH tr() descriptors |
| | createwalletdescriptor | Creates descriptor wallets supporting P2QRH functionality |
| Address Operations | getnewaddress | Generates P2QRH addresses when address_type="bech32m" |
| | getrawchangeaddress | Creates P2QRH change addresses with bech32m encoding |
| | validateaddress | Validates P2QRH addresses (bc1p prefix) |
| | getaddressinfo | Provides detailed information about P2QRH addresses |
| PSBT Operations | walletprocesspsbt | Processes PSBTs containing P2QRH inputs/outputs |
| | createpsbt | createpsbt '[]' '{"bc1r...":0.01}' |
| | descriptorprocesspsbt | Handles PSBTs with P2QRH descriptor data |
| | walletcreatefundedpsbt | Creates new PSBTs with P2QRH outputs |
| | utxoupdatepsbt | Updates PSBT data for P2QRH inputs/outputs |
| Raw Transaction Operations | signrawtransactionwithwallet | Signs P2QRH inputs using wallet keys |
| | createrawtransaction| createrawtransaction '[]' '{"bc1r...":0.01}'
| | signrawtransactionwithkey | Signs P2QRH inputs with specified keys |
| | decoderawtransaction | Decodes transactions with P2QRH inputs/outputs |
| | decodescript | decodescript "<pq2rh scriptPubKey>" |
| Utxo | scantxoutset | scantxoutset start '["p2qrh(quantum_key)"] |
| | gettxout | gettxout "txid" 0 |


## 9. P2QRH Related Tests

### 9.1. Unit Tests

- **test/functional/feature_taproot.py**:
  - Tests Taproot activation sequence
  - Validates key-path and script-path spending
  - Tests commitment scheme verification
  - Checks address encoding/decoding

- **test/functional/wallet_taproot.py**:
  - Tests descriptor wallet integration
  - Verifies PSBT handling for Taproot
  - Tests change output creation
  - Validates address generation

- **src/test/script_tests.cpp**:
  - Unit tests for Tapscript execution
  - Tests Schnorr signature validation
  - Verifies leaf version handling
  - Tests script path validation

- **src/test/key_tests.cpp**:
  - Tests x-only pubkey operations
  - Validates key tweaking operations
  - Tests key aggregation


### 9.2. Integration tests

1. Execute _validateaddress_ test (which includes P2PQRH related address tests)

    ```
    $ build/test/functional/rpc_validateaddress.py
    ```
