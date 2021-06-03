# Ethereum Attestation Registry with Intermediate Hash Proof Type

## Abstract

This document describes an embedded proof type, applicable both for Verifiable Credentials and for Verifiable Presentations, as described in the [Verifiable Credentials Data Model](https://w3c.github.io/vc-data-model/)'s ["Proofs" section](https://w3c.github.io/vc-data-model/#proofs-signatures) by the W3C Credentials Community Group.

## Status of this Document
This specification is an unofficial draft. It is provided as a reference for people and organisations who wish to implement Ethereum-based proofs.

## Introduction

The Verifiable Credentials specification [describes](https://w3c.github.io/vc-data-model/#proofs-signatures) a modular mechanism for declaring how a given credential must be verified.

This specification introduces a new type of embedded proof called `"EthereumAttestationRegistryIntermediateStatus2021"`. This type of proof is based on a trusted smart contract deployed on an Ethereum blockchain accessible by the verifier. Verification of a credential containing this type of proof is done by calls a specific function to the smart contract.

This document extends the [Verifiable Credentials specification](https://w3c.github.io/vc-data-model/) and assumes that the terminology and concepts of that specification are known.

## Specification

An "EthereumAttestationRegistryIntermediateStatus2021" proof is represented by an object contained in the `"proof"` section of a Verifiable Credential. As such, per the VC Data Model, it is considered an "embedded proof".

It must contain the following attributes:

<dl>
  <dt>type</dt>
  <dd>The proof type, as defined in the Verifiable Credentials specification. The value MUST be <code>"EthereumAttestationRegistryIntermediateStatus2021"</code>.</dd>

  <dt>contractAddress</dt>
  <dd>A string containing the hexadecimal Ethereum address of the smart contract that allows the verification of the credential. For example: <code>"0x123f681646d4a755815f9cb19e1acc8565a0c2ac"</code>. That contract must implement **EIP-XXX (to be defined): Content Attestation Registry**.</dd>

  <dt>networkId</dt>
  <dd>A string containing the hexadecimal NetworkId of the Ethereum network where the contract is deployed. In order to be able to verify the credential, the verifier must have read access to a node running on that network.</dd>
</dl>

This proof type requires the DID document of the issuer's DID to contain an `ethereumAddress` entry.

## Example

```json
{
  "@type": "VerifiableCredential",
  "@context": "http://schema.org",
  "credentialSubject": {
    "@id": "did:example:abcd",
    "name": "John Doe",
    "birthdate": "2018-01-01"
  },
  "issuer": "did:gid:2uukHPBYMjdZPkg4p5ZjipKHzkaXLr4T5ut",
  "proof": {
      "type": "EthereumAttestationRegistryIntermediateStatus2021",
      "contractAddress": "0x123f681646d4a755815f9cb19e1acc8565a0c2ac",
      "networkId": "0x19"
    }
}
```

## Hash calculation method

This proof type involves 2 different hashes per credential: *Attestation Hash* and *Revocation Hash*, respectively written on the Ethereum newtwork when the credential is issued and (optionally) revoked.

The following steps MUST be applied to generate either hash.

1. **Step 1: Calculate the credential's hash.**
   1. If present, temporarily strip the whole `"proof"` attribute from the credential, even if it contains multiple proofs.
   2. Serialize the resulting object as a string. TODO: Describe the serialization method (it must be deterministic).
   3. The SHA256 hash of the string is the credential's hash.

2. **Step 2: Calculate the final hash.**
   1. Create the following object:
      ```json
      {
         "hash": <credential's hash in hexadecimal>,
         "status": "Valid" | "Revoked"
      }
      ```
   2. Serialize the object as a string. TODO: Describe the serialization method (it must be deterministic).
   3. The SHA256 hash of the string is the final hash (i.e. either Attestation Hash or Revocation Hash depending on the value of `status`).


## Proof Generation Method

The following steps MUST be applied by a credential's issuer in order to generate an Ethereum Attestation Registry proof of the credential:

1. **Step 1: Calculate Attestation Hash** (see above).
2. **Step 2: Send the Ethereum transaction.**
   1. Decide on the Ethereum network and the registry smart contract to be used.
   2. On that Ethereum network, make a call to the registry smart contract containing a call to the `verify(bytes32 hash, uint iat, uint exp)` function, where `hash` is the Attestation Hash.
      - Good values for `iat` and `exp` are the current time and `0` respectively, but you can use different values or make subsequent calls to the contract as needed. See ERC for details.
      - The call to the registry smart contract must be done from an `ethereumAddress` present in the issuer's DID Document.

3. **Create the `proof` object.**
   1. Use the original credential (i.e. without stripping the `proof object`).
   2. Update the credential with the new `proof` object (see Specification section above).

## Proof Revocation Method

This proof type allows for an issued credential to be revoked by the issuer before its expiration date.

1. **Step 1: Calculate Revocation Hash**
2. **Step 2: Send the Ethereum transaction**
   - Use the contract address and network listed in the `proof` object.
   - Same recommendations as for proof generation, this time using the Revocation Hash.


## Proof Verification Method

To verify the proof, the hashes must be looked up on the registry smart contract. A credential is deemed valid if the Attestation Hash is valid *and* the Revocation Hash is not valid.

The overall verification strategy is the following:

- Look up the Verification Hash.
  - If it is found, the credential is not valid.
  - Otherwise, look up the Attestation Hash.
    - If it is found, the credential is valid.
    - Otherwise, the credential is not valid.

The following steps MUST be applied to look up either hash:

- Use the contract address and network listed in the `proof` object.
- Call the `verifications(bytes32 hash, address issuer)` function, where:
  - `hash` is the hash to verify.
  - `issuer` is the `ethereumAddress` of the issuer's DID, read from the DID Document.
- The function returns an `(iat, exp)` pair.
- The hash is found if the following conditions are met:
  - `iat` is not `0` and is lower than the time of the verification.
  - `exp` is `0` or is higher than the time of the verification.

## Performance Considerations

With this proof type, an Ethereum transaction must be executed for each new attestation. This specification should be improved in the future to allow more scalable approaches.
