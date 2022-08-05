[![](../../.maintain/media/kilt-header.png)](https://kilt.io)

### **This repo has been archived in favor of individual repositories for each of the specifications. Please visit https://github.com/KILTprotocol/spec-KiltPublishedCredentialCollectionV1 for the latest version.**

# KiltPublishedCredentialCollectionV1

### Editors

- **Antonio Antonino** - KILT Protocol [antonio@kilt.io](mailto:antonio@kilt.io)

### Version History

- **v1.1 - May.25 2022**: Add support for optional credential metadata
- **v1.0 - May.17 2022**: Initial spec publishing

---

## Abstract

This document defines an extension to the service types supported in the [DID Core W3C spec][did-core-spec] by defining the `KiltPublishedCredentialCollectionV1` service type.
The goal of the endpoints of this class is to expose a collection (i.e., a list) of KILT credentials, which anyone can retrieve and verify using the functionalities that the [KILT SDK][kilt-sdk] provides.
For more information about the KILT DID method, please visit our [official specification][kilt-did-spec].

## Data structure

A service endpoint of type `KiltPublishedCredentialCollectionV1` does not include any additional properties compared to what is defined within the [relative section of the official DID Core spec][did-core-spec-services].
Furthermore, endpoints of such type MUST include at least *one* URI for the `serviceEndpoint` property.
The URIs SHOULD support integrity protection to ensure that the off-chain data represented by the collection of credentials is immutably linked to the information specified in the service endpoint and anchored to the KILT blockchain.
Examples of integrity-protected URIs include [IPFS][ipfs] and [Hashlink][hashlink], as the one presented in the example below.

```json
{
  "id": "did:kilt:4pqDzaWi3w7TzYzGnQDyrasK6UnyNnW6JQvWRrq6r8HzNNGy#my-service-id",
  "type": [
    "KiltPublishedCredentialCollectionV1"
  ],
  "serviceEndpoint": [
    "https://ipfs.io/ipfs/QmNUAwg7JPK9nnuZiUri5nDaqLHqUFtNoZYtfD22Q6w3c8"
  ]
}
```

Each of the URIs in the service endpoint MUST point to a resource that is retrievable with a GET request.
The structure of the resource returned by the service endpoint MUST be **a list of 0 or more elements** with the following structure.

```json
[
  {
    "credential": {
      "claim": {
        "cTypeHash": "0x47d04c42bdf7fdd3fc5a194bcaa367b2f4766a6b16ae3df628927656d818f420",
        "contents": {
          "Twitter": "mr-x"
        },
        "owner": "did:kilt:4pqDzaWi3w7TzYzGnQDyrasK6UnyNnW6JQvWRrq6r8HzNNGy"
      },
      "claimHashes": [
        "0xb5984f7bf846423a4e884958d02fe479740a526d2a48f3eb28c2ebd585d79652",
        "0xdd6d86383c8b70b2fb0a56ff6d91d37220a305f5b72c5b437cc2a3c34c077b0e"
      ],
      "claimNonceMap": {
        "0x23d9ca2b4e78dadf9bba99396f1023a9912dd7ca60f4346a19372c79cf71608e": "05e74568-4685-4550-ac6c-368120696634",
        "0x5690599ee6cb1835146780d883b5ab5ff83a0e55fef79dc5c721c6cb125c6e22": "f9bc9b46-61c3-47f0-95ea-7cc53f374b9e"
      },
      "claimerSignature": {
        "keyUri": "did:kilt:4pqDzaWi3w7TzYzGnQDyrasK6UnyNnW6JQvWRrq6r8HzNNGy#0xfb589865a4ecd8bf5e9f9e7c7d26293d6123f9c2d09b92e0a787f9641918d6b3",
        "signature": "0x0c4bb527df1d4c4ce0ac35beeecec42422cd84fdd8272dee2b2f28305c6e73594ff5b72dfad266b6aa756af161690ae96c234ba9a1bb3998c969f3d5ef4b768b"
      },
      "legitimations": [],
      "rootHash": "0x73b2063f713258256c93eecc6b7633583647aa9232b1ed5620eb971cd3309727"
    },
    "metadata": {
      "label": "My Personal Twitter Credential",
      "blockNumber": 199530,
      "txHash": "0x366a3e938b94cf585896d8da93e0d6729fcb7b3f9d16cf69e07ff595c0becc40"
    }
  }
]
```

Each object in the list MUST have the following structure, and consumers of this list MUST discard any additional properties that are not defined in this specification:

- `credential`: [REQUIRED] It is a regular KILT request for attestation object, as specified in the [KILT official documentation][kilt-credential-docs].
- `metadata`: [OPTIONAL] It contains additional metadata that applications could use to convey additional information to their users.
  - `label`: [OPTIONAL] It contains a label for the credential this metadata object is linked to.
  - `blockNumber`: [OPTIONAL] It contains the block number in which the credential has been attested on the KILT blockchain. This information can be used to perform RPC calls to KILT full nodes via the `chain > getBlockHash(blockNumber)` and `chain > getBlock(hash)` endpoints.
  - `txHash`: [OPTIONAL] It contains the hash of the attestation transaction. If also `blockNumber` is specified, then applications MUST ensure that a transaction identified by `txHash` is present in the block # `blockNumber`.

## Security considerations

In most cases, the credentials published in a collection behind a service endpoint are meant to be consumed in a non-interactive way, i.e., without the participation of the credential subject proving ownership over them.
This means that the trust in the link between the credential subject and the credentials in the published collection SHOULD be established as a separate step.

**Credential consumers SHOULD at the very least verify that the subject of each and every credential of interest in the published collection, i.e., the value of the `claim.owner` property, matches the identity of the intended subject.
This operation is required because it is trivial for malicious third parties to retrieve the credential collection of an unaware user and publish the same collection as one of their own endpoints. This operation might mislead unaware credential consumers.**

## Example implementation

An example code snippet implementing a simple verification flow fetching the DID document, parsing its endpoints, and verifying the credential collection can be found in [KiltPublishedCredentialCollectionV1.ts](../../snippets/src/KiltPublishedCredentialCollectionV1.ts).

[did-core-spec]: https://www.w3.org/TR/did-core
[kilt-sdk]: https://github.com/KILTprotocol/sdk-js
[kilt-did-spec]: did-spec.md
[did-core-spec-services]: https://www.w3.org/TR/did-core/#services=
[kilt-credential-docs]: https://docs.kilt.io/docs/sdk/core-feature/claiming
[ipfs]: https://ipfs.io/
[hashlink]: https://datatracker.ietf.org/doc/html/draft-sporny-hashlink