[![](https://user-images.githubusercontent.com/39338561/122415864-8d6a7c00-cf88-11eb-846f-a98a936f88da.png)](https://kilt.io)

# KiltPublishedCredentialCollectionV1

### Editors

- **Antonio Antonino** - KILT Protocol [antonio@kilt.io](mailto:antonio@kilt.io)

### Version History

<!---
TODO Replace date before merging PR
-->
- **v1.0 - Xxx.xx 2022**: Initial spec publishing

---

## Abstract

This document defines an extension to the service types supported in the [DID Core W3C spec][did-core-spec] by defining the `KiltPublishedCredentialCollectionV1` service type.
For more information about the KILT DID method, please visit our [official specification][kilt-did-spec].

## Data structure

A service endpoint of type `KiltPublishedCredentialCollectionV1` does not include any additional properties compared to what is defined within the [relative section of the official DID Core spec][did-core-spec-services].
Nevertheless, this specification defines the structure of the data returned by such endpoint.

An example service endpoint could be the following:

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

The example above indicates that the subject has published a collection of KILT credentials that can be obtained by retrieving the resource at the URI indicated by the `serviceEndpoint` property.

This specification defined the expected structure of the resource pointed at by a service endpoint of type `KiltPublishedCredentialCollectionV1` as being **a list of 0 or more elements** having the structure of a KILT credential:

```json
{
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
    "keyId": "did:kilt:4pqDzaWi3w7TzYzGnQDyrasK6UnyNnW6JQvWRrq6r8HzNNGy#0xfb589865a4ecd8bf5e9f9e7c7d26293d6123f9c2d09b92e0a787f9641918d6b3",
    "signature": "0x0c4bb527df1d4c4ce0ac35beeecec42422cd84fdd8272dee2b2f28305c6e73594ff5b72dfad266b6aa756af161690ae96c234ba9a1bb3998c969f3d5ef4b768b"
  },
  "legitimations": [],
  "rootHash": "0x73b2063f713258256c93eecc6b7633583647aa9232b1ed5620eb971cd3309727"
}
```

For more information about KILT and its credential structure, please visit the [official documentation][kilt-credential-docs].

### Service endpoint URIs

This specification does not enforce service endpoints of type `KiltPublishedCredentialCollectionV1` to have any specific value for their `serviceEndpoint` property.
Nevertheless, it is recommended to use integrity-protected URIs ensuring that the off-chain data represented by the collection of credentials is immutably linked to the information specified in the service endpoint and anchored to the KILT blockchain.
Examples of integrity-protected URIs include [IPFS][ipfs] and [Hashlink][hashlink], as the one presented above in the [Data structure section](#data-structure).

[did-core-spec]: https://www.w3.org/TR/did-core
[kilt-did-spec]: did-spec.md
[did-core-spec-services]: https://www.w3.org/TR/did-core/#services=
[kilt-credential-docs]: https://docs.kilt.io/docs/sdk/core-feature/claiming
[ipfs]: https://ipfs.io/
[hashlink]: https://datatracker.ietf.org/doc/html/draft-sporny-hashlink