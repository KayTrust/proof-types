# VC Proof Type based on Zero-knowledge Proofs

## Abstract

This document describes an embedded proof type, applicable for Verifiable Credentials, as described in the [Verifiable Credentials Data Model](https://w3c.github.io/vc-data-model/)'s ["Proofs" section](https://w3c.github.io/vc-data-model/#proofs-signatures) by the W3C Credentials Community Group.

## Status of this Document
This specification by NTT DATA is an unofficial draft. It is provided as a reference for people and organisations who wish to implement Ethereum-based proofs.

## Introduction

The Verifiable Credentials specification [describes](https://w3c.github.io/vc-data-model/#proofs-signatures) a modular mechanism for declaring how a given credential must be verified.

This specification introduces a new type of embedded proof called "BBSPlus2024". In addition to letting verifiers verify the integrity of a VC signed with this proof type, it also lets provers derive statements from this proof, rather than the original information. See BBSPlusRange2024 for an example.

This document builds on top of the [Verifiable Credentials specification](https://w3c.github.io/vc-data-model/) and assumes that the terminology and concepts of that specification are known.

This proof type can be used in zero-knowledge proof scenarios. Such scenarios are useful when the holder of an original VC needs to generate a second VC derived from that original VC, with less specific information. The particularity of such derived VCs is that even though their issuer is the same as the original VC's, the derived VC itself is actually created by the holder. When that happens, the holder is called "prover".

## Specification

A BBSPlus2024 proof is represented by an object contained in the `"proof"` section of a Verifiable Credential. As such, per the VC Data Model, it is considered an "embedded proof".

It must contain the following attributes:

<dl>
  <dt>type</dt>
  <dd>The proof type, as defined in the Verifiable Credentials specification. The value MUST be <code>"BBSPlusRange2024"</code>.</dd>
  <dt>label</dt>
  <dd>The BBS+ label to be used.</dd>
  <dt>sig</dt>
  <dd>The content of the BBS+ signature.</dd>
</dl>

## Message encoding

BBS+ signatures involved a signed list of messages. This document specifies how to go back and forth between a list of attributes in a VC and a list of messages:

- The signer chooses an order in the list of claims present in `credentialSubject`, and generates a `claims` attribute in the `proof` object, representing the list of claims in the chosen order.
- Special key names `.issuanceDate` and `.expirationDate` (with leading dots) must always be present, and correspond to the VC's top level attributes of the same names. If either claim is missing from the VC, the key should still be included in the list, with a value of 0.
- Dates are always expressed as integer values representing a timestamp.

Please note that all claim names mentioned in the VC's credentialSubject must be covered in the message list, otherwise verifiers won't be able to verify their integrity.

## Example

```json
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1"
  ],
  "type": [
    "VerifiableCredential"
  ],
  "credentialSubject": {
    "id": "did:example:123",
    "givenName": "John",
    "familyName": "Doe",
    "birthDate": 762365708
  },
  "issuer": "did:example:456",
  "issuanceDate": "2024-02-13T14:32:31.126Z",
  "proof": {
    "type": "BBSPlus2024",
    "sig": "ibspQVU3EmMxS7jW3_sF0iFa4uu5A5ToZDjLJb-LsAuAkbPeM19CdaLQTvQw50QKcQj05T0VETpZ4Sb18H8tpGlaYMr66QmbRGYHTjQxqgTKdYoJVBswOIUAgzCKnrJEHYZfusgKqITXC97fbk4paA",
    "label": "Some label",
    "claims": [".issuanceDate", ".expirationDate", "id", "givenName", "familyName", "birthDate"]
  }
}
```

## Proof Generation Steps (Issuer)

1. Pick an order for the claims contained in the VC's credentialSubject attribute.
2. Following that order, generate a list of messages in the form: ["key1", "value1", "key2", "value2", ...].
3. Sign the message list with BBS+.
4. Encode the signature in base 64.
5. Generate "proof" object based on the signature and the claim order.

## Proof Verification Steps

1. Following the order of claims in the proof object and the values present in the VC, generate a list of messages in the form ["key1", "value1", "key2", "value2", ...]
2. Decode the signature as base64 and verify the message list against the signature.

# Algorithms

## createBBSPlus2024Proof

The following algorithm creates a BBSPlus2024 proof object for a verifiable credential. The required inputs are a credential, (which optional fields `issuanceDate`, `expirationDate` and a required field `credentialSubject`), a BBS+ private key and a public parameter known as a `label` (a string).

1. Generate the message list `messages` from the credential.
    1. Create an object of depth one with `issuanceDate`, `expirationDate` and all the information from `credentialSubject`.
    2. Define a claim order where the strings `.issuanceDate` and `.expirationDate` are always first.
    3. Flat this new object in an array that must follow the pattern `[key1, value1, key2, value2, ...]`, where `keyn` is either `".issuanceDate"`, `".expirationDate"` or the claim name, always in a even index, and the value in the subsequent odd index. Values for `issuanceDate` or `expirationDate` defaults to `0`.
2. Sign the message list with the BBS+ private key and public parameters. You must derived public parameter `messageCount` from the array generated in step 1.
3. Encode the signature bytes in base 64 url.
4. Generate `proof` object with `type`, `sig`, `label` and `claims` properties.
    1. `type` is always "BBSPlus2024".
    2. `sig` is the base 64 url encoded signature.
    3. `label` is the same public parameter that was used as one of the inputs of this function.
    4. `claims` elements are `".issuanceDate"`, `".expirationDate"` and all the claim names included in the message list `messages`, in the order defined in step 1.2. It size must be half of what the total number of messages that were signed.

Example of `proof` object:

```json
{ 
  "proof": {
    "type": "BBSPlus2024",
    "sig": "ibspQVU3EmMxS7jW3_sF0iFa4uu5A5ToZDjLJb-LsAuAkbPeM19CdaLQTvQw50QKcQj05T0VETpZ4Sb18H8tpGlaYMr66QmbRGYHTjQxqgTKdYoJVBswOIUAgzCKnrJEHYZfusgKqITXC97fbk4paA",
    "label": "Some label",
    "claims": [".issuanceDate", ".expirationDate", "id", "givenName", "familyName", "birthDate"]
  }  
}
```

