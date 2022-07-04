[![](https://user-images.githubusercontent.com/39338561/122415864-8d6a7c00-cf88-11eb-846f-a98a936f88da.png)](https://kilt.io)

# Asset Decentralized Identifiers (DID) Method Specification

### Editors

- **Antonio Antonino** - KILT Protocol [antonio@kilt.io](mailto:antonio@kilt.io)

### Version History

- **v1.0 - July.04 2022**: Initial definition and RFC for the new method

---

## Abstract

This document defines the Asset DID Method that conforms to the [DID Core W3C Spec][did-core-spec].
By the term *asset*, the authors refer to both an entire asset (or token) class (e.g., the entire Bitcoin fungible asset class) as well as a specific instance within that class (e.g., a specific NFT - the instance - within an NFT collection - the asset class).

Hence, an Asset Decentralized Identifier (DID) is a string uniquely identifying assets on any existing and future blockchain.

## Method Syntax

### DID Scheme and Identifiers

The Asset DID method is identified by the `asset` scheme, as in the following example: `did:asset:bip122:000000000019d6689c085ae165831e93:slip44:0`.

Asset DIDs are composed of the following elements:

```
asset-did         = "did:asset:" + <caip-19-asset-id>
caip-19-asset-id  = <caip-2-chain-id> + ":" + <asset-namespace> + ":" + <asset-reference> + [":" + <asset-id>]
caip-2-chain-id   = <chain-namespace> + ":" + <chain-reference>
chain-namespace   = [-a-z0-9]{3,8}
chain-reference   = [-a-zA-Z0-9]{1,32}
asset-namespace   = [-a-z0-9]{3,8}
asset-reference   = [-a-zA-Z0-9]{1,64}
asset-id          = [-a-zA-Z0-9]{1,78}
```

The `caip-2-chain-id` is a CAIP (Chain Agnostic Improvement Proposal) chain identifier as described in the [CAIP-2 Blockchain ID Specification][caip-2-spec].
The `caip-19-asset-id` is derived from a CAIP asset identifier as described in the [CAIP-19 Asset Type and Asset ID Specification][caip-19-spec], where the occurrences of the `/` symbol are replaced with the `:` symbol, to render the resulting string a valid URI according to the [DID Core Specification][did-core-spec].

Chain and asset namespaces are defined with Improvement Proposals and are stored under the [CAIP repository][caip-repo].
For instance, the namespaces for [EIP155][caip-3-spec] (Ethereum-based) and [BIP122][caip-4-spec] (Bitcoin-based) chains as well as for [SLIP44][caip-20-spec] (native fungible tokens), [ERC20][caip-21-spec] (smart contract based tokens), and [ERC721][caip-22-spec] (NFTs) assets have already been defined.

## Method Operations

### Create an Asset DID

A new Asset DID creation is automatically performed whenever a new asset class is deployed or a new asset instance for a given class minted.
Because of the chain agnostic nature of Asset DIDs, this process can take place on any blockchain that is identifiable by a [CAIP-2 Identifier][caip-2-spec].

### Resolve an Asset DID

The information required to resolve an Asset DID is the following:

- The **chain** on which the asset live, e.g., the Ethereum mainnet
- The **namespace** defining the asset, e.g., the ERC721 namespace
- The **reference** to the asset class within the defined namespace, e.g., the smart contract address for an NFT collection
- [OPTIONAL] The unique **identifier** of the asset instance within the asset class, e.g., the NFT ID within a given collection

The resolution process then follows a hierarchical structure.

The chain identifier is decomposed into its namespace and reference.
Then, the resolver for the specified namespace is used to resolve the specified reference.

The same process is followed for the asset identifier.
The identifier is decomposed into its namespace, reference, and optional ID.
Then, the resolver for the specified asset namespace is used to resolve the asset reference and ID.

As an example, the returned DID Document for [one of the CryptoKitties NFTs](https://opensea.io/assets/ethereum/0x06012c8cf97bead5deae237070f9587f8e7a266d/634446), identified with the Asset DID `did:asset:eip155:0:erc721:0x06012c8cf97bead5deae237070f9587f8e7a266d:634446`, is the following:

```json
{
  "@context": "https://www.w3.org/ns/did/v1",
  "id": "did:asset:eip155:0:erc721:0x06012c8cf97bead5deae237070f9587f8e7a266d:634446",
  "controller": "did:pkh:eip155:0:0x929C3dAF2E2Be4C74A56EC01dF374bc46A34C6A1",
}
```

where the `controller` field is optional and present only if the referenced asset class supports the concepts of ownership (which is not present, for instance, in the definition of a fungible asset class).
If present, the `controller` property represents the asset owner and their [DID PKH](https://github.com/w3c-ccg/did-pkh/blob/main/did-pkh-method-draft.md).

DID resolvers compliant to the DID Specification will return the relevant error codes in case the asset specified cannot be resolved (e.g., if it does not exist).

<!-- TODO: Investigate the usage of the `alsoKnownAs` property when supporting asset transfers -->

### Update an Asset DID

An Asset DID update is automatically performed whenever the underlying asset is updated, e.g., when its ownership changes.
Updates operations are defined by the asset class and regulated by the network on which the asset lives, hence they are not within the scope of this method specification.

### Deactivate an Asset DID

An Asset DID deactivation is automatically performed whenever the underlying asset ceases to exist, e.g., when it is burned.

[did-core-spec]: https://www.w3.org/TR/did-core
[caip-repo]: https://github.com/ChainAgnostic/CAIPs
[caip-2-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-2.md
[caip-3-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-3.md
[caip-4-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-4.md
[caip-19-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-19.md
[caip-20-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-20.md
[caip-21-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-21.md
[caip-22-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-22.md