[![](../../.maintain/media/kilt-header.png)](https://kilt.io)

# Asset Decentralized Identifiers (DID) Method Specification

### Editors

- **Antonio Antonino** - KILT Protocol [antonio@kilt.io](mailto:antonio@kilt.io)

### Version History

- **v1.0 - July.22 2022**: Initial definition and RFC for the new method

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

<!-- TODO: Investigate the usage of the `alsoKnownAs` property when supporting asset transfers -->

### Update an Asset DID

The Asset DID method does not support direct updates, being a purely generative method, but its resolution reflects any changes to the underlying asset or to its credentials.

### Deactivate an Asset DID

The Asset DID method does not support deactivations, being a purely generative method.

## Asset DID Resolution

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
  "chain": {
    "namespace": "eip155",
    "reference": "0"
  },
  "asset": {
    "namespace": "erc721",
    "reference": "0x06012c8cf97bead5deae237070f9587f8e7a266d",
    "identifier": "634446"
  }
}
```

At the present time, the resolution result does not include any document metadata.

## Asset DID URL Dereferencing

Beyond simple identifiers, the DID standard defines URLs to enhance the semantics of the resolution process, allowing resolvers to include additional information to the resolution result or to transform such information according to the client's needs.

Following the [DID URL specification][did-url-spec], an Asset DID URL has the following syntax:

```
asset-did-url               = "did:asset:" + <path-abempty> + ["?" + <query>] + ["#" + <fragment>]
```

where `path-abempty`, `query` and `fragment` are defined in [RFC3986][rfc-3986].

The list of supported paths, queries, and fragments is shown below and follows the rules presented in the [DID Resolution specification][did-resolve-spec].

### Asset DID URL paths

At the present time, not path is allowed in an Asset DID URL.
The specification of paths in an Asset DID URL will result in a parsing error.

### Asset DID URL queries

At the present time, not query parameter is allowed in an Asset DID URL.
The presence of any query parameters in an Asset DID URL will be disregarded.

### Asset DID URL fragments

At the present time, not fragments is allowed in an Asset DID URL.
The specification of fragments in an Asset DID URL will result in a parsing error.

[did-core-spec]: https://www.w3.org/TR/did-core
[did-resolve-spec]: https://w3c-ccg.github.io/did-resolution/
[did-url-spec]: https://www.w3.org/TR/did-core/#did-url-syntax
[caip-repo]: https://github.com/ChainAgnostic/CAIPs
[caip-2-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-2.md
[caip-3-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-3.md
[caip-4-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-4.md
[caip-19-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-19.md
[caip-20-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-20.md
[caip-21-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-21.md
[caip-22-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-22.md
[rfc-3986]: https://www.w3.org/TR/did-core/#bib-rfc3986
[vc-spec]: https://www.w3.org/TR/2022/REC-vc-data-model-20220303/