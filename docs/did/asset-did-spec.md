[![](../../.maintain/media/kilt-header.png)](https://kilt.io)

# Asset Decentralized Identifiers (DID) Method Specification

### Editors

- **Antonio Antonino** - KILT Protocol [antonio@kilt.io](mailto:antonio@kilt.io)

### Version History

- **v1.0 - July.13 2022**: Initial definition and RFC for the new method

---

## Abstract

This document defines the Asset DID Method that conforms to the [DID Core W3C Spec][did-core-spec].
By the term *asset*, the authors refer to both an entire asset (or token) class (e.g., the entire Bitcoin fungible asset class) as well as a specific instance within that class (e.g., a specific NFT - the instance - within an NFT collection - the asset class).

Hence, an Asset Decentralized Identifier (DID) is a string uniquely identifying assets on any existing and future blockchain.

## Method Syntax

### DID Scheme and Identifiers

The Asset DID method is identified by the `asset` scheme, as in the following example: `did:asset:bip122:000000000019d6689c085ae165831e93.slip44:0`.

Asset DIDs are composed of the following elements:

```
asset-did                   = "did:asset:" + <modified-caip-19-asset-id>
modified-caip-19-asset-id   = <caip-2-chain-id> + "." + <asset-namespace> + ":" + <asset-reference> + [":" + <asset-id>]
caip-2-chain-id             = <chain-namespace> + ":" + <chain-reference>
chain-namespace             = [-a-z0-9]{3,8}
chain-reference             = [-a-zA-Z0-9]{1,32}
asset-namespace             = [-a-z0-9]{3,8}
asset-reference             = [-a-zA-Z0-9]{1,64}
asset-id                    = [-a-zA-Z0-9]{1,78}
```

The `caip-2-chain-id` is a CAIP (Chain Agnostic Improvement Proposal) chain identifier as described in the [CAIP-2 Blockchain ID Specification][caip-2-spec].
The `modified-caip-19-asset-id` is derived from a CAIP asset identifier as described in the [CAIP-19 Asset Type and Asset ID Specification][caip-19-spec], with the `/` symbol separating the chain identifier from the asset identifier being replaced with the `.` symbol, and the optional `/` symbol preceding the `asset-id` component being replaced by the `:` symbol.
This is required to render the resulting string a valid URI according to the [DID Core Specification][did-core-spec].

Chain and asset namespaces are defined with Improvement Proposals and are stored under the [CAIP repository][caip-repo].
For instance, the namespaces for [EIP155][caip-3-spec] (Ethereum-based) and [BIP122][caip-4-spec] (Bitcoin-based) chains as well as for [SLIP44][caip-20-spec] (native fungible tokens), [ERC20][caip-21-spec] (smart contract based tokens), and [ERC721][caip-22-spec] (NFTs) assets have already been defined.

## Method Operations

### Create an Asset DID

An Asset DID as such is not directly created, but it is generated at any time from a specific asset on as specific network.
Because of the chain agnostic nature of Asset DIDs, this process can take place on any blockchain that is identifiable by a [CAIP-2 Identifier][caip-2-spec].

Furthermore, as far as Asset DIDs are concerned, the assets identified do not need to already exist.
For instance, a "right to create" a new NFT collection on a given blockchain for a user could be encoded in a [Verifiable Credential][vc-spec] where the identifier of the asset is an Asset DID.

### Resolve an Asset DID

The information required to resolve an Asset DID is the following:

- The **chain** on which the asset lives, e.g., the Ethereum mainnet
- The **namespace** defining the asset, e.g., the ERC721 namespace
- The **reference** to the asset class within the defined namespace, e.g., the smart contract address for an NFT collection
- [OPTIONAL] The unique **identifier** of the asset instance within the asset class, e.g., the NFT ID within a given collection

The resolution process then follows a hierarchical structure.

The chain identifier is decomposed into its namespace and reference.
Then, the resolver for the specified namespace is used to resolve the specified reference.

The same process is followed for the asset identifier.
The identifier is decomposed into its namespace, reference, and optional ID.
Then, the resolver for the specified asset namespace is used to resolve the asset reference and ID.

As an example, the returned DID Document for [one of the CryptoKitties NFTs](https://opensea.io/assets/ethereum/0x06012c8cf97bead5deae237070f9587f8e7a266d/634446), identified with the Asset DID `did:asset:eip155:0.erc721:0x06012c8cf97bead5deae237070f9587f8e7a266d:634446`, is the following:

<!-- TODO: Store the context somewhere and update this link when defined. -->

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://kilt.io/asset-did-context.json"
  ],
  "id": "did:asset:eip155:0.erc721:0x06012c8cf97bead5deae237070f9587f8e7a266d:634446",
  "controller": "did:pkh:eip155:0:0x929C3dAF2E2Be4C74A56EC01dF374bc46A34C6A1",
  "service": [{
    "id": "did:asset:eip155:0.erc721:0x06012c8cf97bead5deae237070f9587f8e7a266d:634446#0x1234567812345678123456781234567812345678123456781234567812345678",
    "type": "KiltPublicCredentialV1",
    "serviceEndpoint": "polkadot:411f057b9107718c9624d6aa4a3f23c1#123"
  }],
}
```

where the `controller` field is optional and present only if the referenced asset class supports the concepts of ownership (which is not present, for instance, in the definition of a fungible asset class).
If present, the `controller` property represents the asset owner and their [DID PKH](https://github.com/w3c-ccg/did-pkh/blob/main/did-pkh-method-draft.md).

The `service` property contains a list of objects with the following structure:

- `id`: it is the Asset DID URI fragment `{asset_did}#{credential_root_hash}` where the `credential_root_hash` is the root hash of the KILT credential being exposed by the service endpoint. For more information about the public credential, refer to the section below about [Asset DID fragments](#resolve-an-asset-did-fragment)
- `type`: for asset public credentials, it is always `KiltPublicCredentialV1`
- `serviceEndpoint`: indicates the blockchain and block number in which the credential was issued. It has the format `{caip_2}#{block_number}` where `caip_2` indicates the blockchain that contains the credential information and `block_number` the block at which the issuance transaction was submitted. In this specification and in this version, we restrict the set of possible values to only `polkadot:411f057b9107718c9624d6aa4a3f23c`, which is the CAIP-2 identifier for the KILT Spiritnet blockchain

DID resolvers compliant to the DID Specification will return the relevant error codes in case the asset specified cannot be resolved (e.g., if it does not exist).

<!-- TODO: Investigate the usage of the `alsoKnownAs` property when supporting asset transfers -->

#### Resolve an Asset DID fragment

<!-- TODO: Update once the node PR has been reviewed and merged or is at least ready to be merged -->

Fragments in Asset DID are used to indicate public credentials that have been issued to that asset.

In the example above: the DID fragment `did:asset:eip155:0.erc721:0x06012c8cf97bead5deae237070f9587f8e7a266d:634446#0x1234567812345678123456781234567812345678123456781234567812345678` indicates that the asset `did:asset:eip155:0.erc721:0x06012c8cf97bead5deae237070f9587f8e7a266d:634446` has been issued a KILT credential with the root hash `0x1234567812345678123456781234567812345678123456781234567812345678`.
The complete credential can be fetched from any KILT full node that is running in archive mode, i.e., that stores all the blocks ever produced.

Using the block number specified in the `serviceEndpoint` property, the block information can be queried from the node with the following RPC calls: `chain > getBlockHash(blockNumber)` to get the hash of the block with a given number, and then `chain > getBlock(hash)` to get all the transactions submitted and the events generated within that block.

Once the list of transactions is available, only the successful ones belonging to the `public-credentials` section and for the `add` method must be considered.
In some cases, more than one transaction of this type might be included in the same block: to distinguish, another filter based on the Asset DID and the credential root hash must be used.
In the edge case in which the result still contains multiple transactions, e.g. if within the same block a credential is issued, then revoked, then issued again, only the last `add` operation should be considered.

The information returned by the node has the following relevant fields, which is a non-exhaustive list:

```json
{
  "claim": {
    "ctype_hash": "0xabcd...",
    "subject": "did:asset:eip155:0.erc721:0x06012c8cf97bead5deae237070f9587f8e7a266d:634446",
    "contents": "0xdcba..."
  },
  "nonce": "0x01234....",
  "claim_hash": "0x1234567812345678123456781234567812345678123456781234567812345678",
  "claimer_signature": {
    "claimer_id": "4fa...",
    "signature_payload": "0x4321..."
  }
}
```

- `claim`
  - `ctype_hash`: the KILT Ctype which the public credential conforms to, as specified by the KILT issuance protocol
  - `subject`: the Asset DID to which the credential has been issued. **This value must match the one used to retrieve this information**
  - `contents`: the CBOR serialisation of the credential claims, as described in the [KILT documentation](https://docs.kilt.io/docs/concepts/credentials/claiming)
- `nonce`: the nonce used to hash each claim and generate the credential root hash
- `claim_hash`: the credential root hash
- `claimer_signature` [OPTIONAL]
  - `claimer_id`: the identifier of the claimer's KILT DID. When parsed, it must be prefixed with `did:kilt`
  - `signature_payload`: the DID signature the claimer generated over the credential root hash, and that is used to proof the involvement of the claimer in the credential issuance process to the specified asset

<!-- TODO: Add link to the docs when support is added and docs are deployed -->

The obtained object can then be parsed and validated using the functionalities provided by the KILT SDK.

### Update an Asset DID

The Asset DID method does not support direct updates, being a purely generative method, but its resolution reflects any changes to the underlying asset or to its credentials.

### Deactivate an Asset DID

The Asset DID method does not support deactivations, being a purely generative method.

[did-core-spec]: https://www.w3.org/TR/did-core
[vc-spec]: https://www.w3.org/TR/2022/REC-vc-data-model-20220303/
[caip-repo]: https://github.com/ChainAgnostic/CAIPs
[caip-2-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-2.md
[caip-3-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-3.md
[caip-4-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-4.md
[caip-19-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-19.md
[caip-20-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-20.md
[caip-21-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-21.md
[caip-22-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-22.md