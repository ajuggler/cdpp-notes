---
label: Metadata
icon: file
author:
  name: Antonio Hernandez
  email: contacto@antoniohernandez.mx
order: -3
---


<a id="org4ecf6c0"></a>

# Working with Metadata

We can attach *metadata* to a transaction.  From the [documentation](https://github.com/input-output-hk/cardano-node/blob/master/doc/reference/tx-metadata.md):

> Transaction metadata (tx metadata) can contain details about a specific transaction, including sender and receiver IDs, transaction comments, and tags. Adding metadata to transactions provides transaction information by adding arbitrarily structured data onto the chain and is a useful feature in Cardano Shelley. Tx metadata is stored on-chain and is carried along with each transaction. A factor in its design is that the on-chain metadata is not stored in the ledger state and does not influence transaction validation, thereby not compromising ledger performance.

The structure of the metadata is defined by a key-value mapping.  Keys are non-negative integers limited in size up to 64 bits and values are simple terms, consisting of integers, text strings, byte strings, lists, and maps.  Values are required to be structured.

On-chain metadata that is carried along with the transaction is encoded as CBOR.  To create a transaction, you can add the metadata with

-   Pre-encoded CBOR
-   Metadata in JSON format with *no schema*
-   Metadata in JSON format with *detailed schema*

In these notes we:  a) write the json file with metadata; b) build, sign and send a transaction with metadata.


<a id="org320ea80"></a>

## The metadata

In this exercise, the purpose of this metadata is to assign a value to a given metadata key (integer) `k = 152755`.  The value is in turn a mapping that assigns the hash of a name to key *name* and the value "1" to the key flag *completed*.


<a id="org3d1fccf"></a>

### Metadata json file

Wrote the file `metadata.json` :

    $ cat metadata.json
    {"152755" : {
        "name" : "7260b0c5083b0109fd9b9e026c0b9f519dc64bdfa901bc8dbc316a2d60870e4f",
        "completed" : 1
    }}

We will use the option  `--metadata-json-file`  to load this metadata.


<a id="org2f5590f"></a>

## The transaction


<a id="org1a81ba9"></a>

### Scripts for build and sign

Wrote the *build* and *sign* scripts.  Don't forget to `chmod u+x <scriptname>` after creating the script files.

1.  Build

    Wrote the script `tx_meta_draft.sh` :
    
        cardano-cli transaction build \
        	    --babbage-era \
        	    --testnet-magic 1097911063 \
        	    --witness-override 1 \
        	    --tx-in $(cat utxo1.tmp) \
        	    --change-address $(cat myWallet1.addr) \
        	    --metadata-json-file metadata.json \
        	    --protocol-params-file $HOME/code/haskell/emurgo/testnet/testnet-protocol-params.json \
        	    --out-file tx_meta.draft

2.  Sign

    Wrote the script `tx_meta_signed.sh` :
    
        cardano-cli transaction sign \
        	    --tx-body-file tx_meta.draft \
        	    --signing-key-file myWallet1.skey \
        	    --testnet-magic 1097911063 \
        	    --out-file tx_meta.signed


<a id="org7db484b"></a>

### Script execution

    $ # Build
    $ ./tx_meta_draft.sh 
    Estimated transaction fee: Lovelace 169637
    $ 
    $ # Sign
    $ ./tx_meta_signed.sh

Let us explore the (not yet submitted) signed transaction:

    $ # View the transaction
    $ cardano-cli transaction view --tx-file tx_meta.signed
    auxiliary scripts: null
    certificates: null
    collateral inputs: []
    era: Babbage
    fee: 169637 Lovelace
    inputs:
    - 30eea3e9228b94cad0cbfb131f65ebf3e9d5b24820394dca8408920037797e64#0
    metadata:
      '152755':
      - - completed
        - 1
      - - name
        - 7260b0c5083b0109fd9b9e026c0b9f519dc64bdfa901bc8dbc316a2d60870e4f
    mint: null
    outputs:
    - address: addr_test1vzxx4hxudpattj4zv8200nn32mvrhyyxpemr3ghwlf82pnq7ewwsz
      address era: Shelley
      amount:
        lovelace: 699658790
      datum: null
      network: Testnet
      payment credential key hash: 8c6adcdc687ab5caa261d4f7ce7156d83b90860e7638a2eefa4ea0cc
      reference script: null
      stake reference: null
    required signers (payment key hashes needed for scripts): null
    update proposal: null
    validity range:
      lower bound: null
      upper bound: null
    withdrawals: null
    witnesses:
    - key: VKey (VerKeyEd25519DSIGN "7fd4a6131432173c843689d13c64b3e616671efa411fb5a812450a28c9ff3088")
      signature: SignedDSIGN (SigEd25519DSIGN "6f83d3cbaeaa9e54a2bc9d6c7ca69c5e912b5464d6d10a7794008544ae27faa2f2c3d6467afcbb76f1a1138fc3065bac529e0ae3fa27f1815b13502d9260c20c")

So we confirm that the *metadata* is part of the transaction.


<a id="org71d0d29"></a>

### Submitting the transaction

    $ # Before submission:
    $ cardano-cli query utxo --address $(cat myWallet1.addr) --testnet-magic 1097911063
    			   TxHash                                 TxIx        Amount
    --------------------------------------------------------------------------------------
    30eea3e9228b94cad0cbfb131f65ebf3e9d5b24820394dca8408920037797e64     0        699828427 lovelace + TxOutDatumNone
    $ 
    $ # Tx submission:
    $ cardano-cli transaction submit --testnet-magic 1097911063 --tx-file tx_meta.signed
    Transaction successfully submitted.
    $ 
    $ # After submission:
    $ cardano-cli query utxo --address $(cat myWallet1.addr) --testnet-magic 1097911063
    			   TxHash                                 TxIx        Amount
    --------------------------------------------------------------------------------------
    888eb4a8a5a63eeb809cb565a6cc42bb42b7b89ba54362abbcf7f743a18fe94c     0        699658790 lovelace + TxOutDatumNone

We see that a utxo was consumed and a new utxo was created.


<a id="orgb26fc91"></a>

## References

-   [Cardano Node - Transaction Metadata](https://github.com/input-output-hk/cardano-node/blob/master/doc/reference/tx-metadata.md)
-   [Simple Scripts](https://github.com/input-output-hk/cardano-node/blob/master/doc/reference/simple-scripts.md)
-   [Koios](https://testnet.koios.rest/#overview)
-   [CardanoScan](https://testnet.cardanoscan.io/)
-   [SHA3-256](https://emn178.github.io/online-tools/sha3_256.html)

