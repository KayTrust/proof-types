# VC Proof Type based on Zero-knowledge Proofs

## Abstract

This document describes an embedded proof type, applicable for Verifiable Credentials, as described in the [Verifiable Credentials Data Model](https://w3c.github.io/vc-data-model/)'s ["Proofs" section](https://w3c.github.io/vc-data-model/#proofs-signatures) by the W3C Credentials Community Group.

## Status of this Document
This specification by NTT DATA is an unofficial draft. It is provided as a reference for people and organisations who wish to implement Ethereum-based proofs.

## Introduction

The Verifiable Credentials specification [describes](https://w3c.github.io/vc-data-model/#proofs-signatures) a modular mechanism for declaring how a given credential must be verified.

This specification introduces a new type of embedded proof called "BBSPlusRange2024". This VC proof is based on zero-knowledge range proofs derived from a message signed with the BBS+ cryptographic suite.

This document builds on top of the [Verifiable Credentials specification](https://w3c.github.io/vc-data-model/) and assumes that the terminology and concepts of that specification are known.

This proof type can be used in zero-knowledge proof scenarios. Such scenarios are useful when the holder of an original VC needs to generate a second VC derived from that original VC, with less specific information. The particularity of such derived VCs is that even though their issuer is the same as the original VC's, the derived VC itself is actually created by the holder. When that happens, the holder is called "prover".

## Specification

A BBSPlusRange2024 proof is represented by an object contained in the `"proof"` section of a Verifiable Credential. As such, per the VC Data Model, it is considered an "embedded proof".

It must contain the following attributes:

<dl>
  <dt>type</dt>
  <dd>The proof type, as defined in the Verifiable Credentials specification. The value MUST be <code>"BBSPlusRange2024"</code>.</dd>
  <dt>label</dt>
  <dd>The BBS+ label. It must be constructed as JSON array in the following form: \["Type","attribute1",...,"attributeN"\], serialized as string.</dd>
  <dt>signature</dt>
  <dd>The content of the generated zero-knowledge proof.</dd>

</dl>


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
    "birthDate": 762365708
  },
  "issuer": "did:example:456",
  "issuanceDate": "2024-02-13T14:32:31.126Z",
  "proof": {
    "type": "BBSPlus2024",
    "sig": "ibspQVU3EmMxS7jW3_sF0iFa4uu5A5ToZDjLJb-LsAuAkbPeM19CdaLQTvQw50QKcQj05T0VETpZ4Sb18H8tpGlaYMr66QmbRGYHTjQxqgTKdYoJVBswOIUAgzCKnrJEHYZfusgKqITXC97fbk4paA",
    "label": "[\"BirthInformation\",\"birthDate\"]"
  }
}
```

## Proof Generation Steps



## Proof Verification Steps


