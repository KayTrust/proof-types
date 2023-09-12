# Ethereum Attestation Registry Proof Type

## Abstract

This document describes an embedded proof type, applicable both for Verifiable Credentials and for Verifiable Presentations, as described in the [Verifiable Credentials Data Model](https://w3c.github.io/vc-data-model/)'s ["Proofs" section](https://w3c.github.io/vc-data-model/#proofs-signatures) by the W3C Credentials Community Group.

## Status of this Document
This specification is an unofficial draft. It is provided as a reference for people and organisations who wish to implement Ethereum-based proofs.

## Introduction

The Verifiable Credentials specification [describes](https://w3c.github.io/vc-data-model/#proofs-signatures) a modular mechanism for declaring how a given credential must be verified.

This specification introduces a new type of embedded proof called "EthereumAttestationRegistry2019". This type of proof is based on a trusted smart contract deployed on an Ethereum blockchain accessible by the verifier. Verification of a credential containing this type of proof is done by calls a specific function to the smart contract.

This document extends the [Verifiable Credentials specification](https://w3c.github.io/vc-data-model/) and assumes that the terminology and concepts of that specification are known.

## Specification

An EthereumAttestationRegistry2019 proof is represented by an object contained in the `"proof"` section of a Verifiable Credential. As such, per the VC Data Model, it is considered an "embedded proof".

It must contain the following attributes:

<dl>
  <dt>type</dt>
  <dd>The proof type, as defined in the Verifiable Credentials specification. The value MUST be <code>"EthereumAttestationRegistry2019"</code>.</dd>

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
  "issuer": "did:ev:2uukHPBYMjdZPkg4p5ZjipKHzkaXLr4T5ut",
  "proof": {
      "type": "EthereumAttestationRegistry2019",
      "contractAddress": "0x123f681646d4a755815f9cb19e1acc8565a0c2ac",
      "networkId": "0x19"
    }
}
```

## Hash calculation method

This proof type involves the calculation of a hash, written on the Ethereum newtwork when the credential is issued.

The following steps MUST be applied to generate the hash.

  1. If present, temporarily strip the whole `"proof"` attribute from the credential, even if it contains multiple proofs.
  2. Serialize the resulting object as a string. TODO: Describe the serialization method (it must be deterministic).
  3. The SHA256 hash of the string is the credential's hash.


## Proof Generation Method

The following steps MUST be applied by a credential's issuer in order to generate an Ethereum Attestation Registry proof of the credential:

1. **Step 1: Calculate Hash** (see above).
2. **Step 2: Send the Ethereum transaction.**
   1. Decide on the Ethereum network and the registry smart contract to be used.
   2. On that Ethereum network, make a call to the registry smart contract containing a call to the `verify(bytes32 hash, uint iat, uint exp)` function, where `hash` is the computed Hash.
      - Good values for `iat` and `exp` are the current time and `0` respectively, but you can use different values or make subsequent calls to the contract as needed. See ERC for details.
      - The call to the registry smart contract must be done from an `ethereumAddress` present in the issuer's DID Document.

3. **Create the `proof` object.**
   1. Use the original credential (i.e. without stripping the `proof object`).
   2. Update the credential with the new `proof` object (see Specification section above).

## Proof Verification Method

The following steps MUST be applied by a credential's verifier in order to verify an Ethereum Attestation Registry proof of a credential:

1. Compute the credential's hash as described above.
2. Determine the Ethereum address corresponding to the issuer's DID, read from the DID Document.
3. Call the `verifications(hash, issuer)` function of the smart contract listed in the proof, where `hash` is the credential's hash and `issuer` is issuer's Ethereum address and store the returned value couple as `(iat, exp)`. Those are the start date and end date, respectively, specified by the issuer when signing the credential.
4. Verify that the time range is acceptable. The following two conditions below SHOULD both be verified for the credential to be considered valid. However, the verifier MAY apply a different policy if the use case justifies it.
    - `iat` is not `0` and is lower than the time of the verification.
    - `exp` is `0` or is higher than the time of the verification.

## Performance Considerations

With this proof type, an Ethereum transaction must be executed for each new attestation. This specification should be improved in the future to allow more scalable approaches.

# Status type

Verifiable Credentials equipped with a `EthereumAttestationRegistry2019` proof use an implicit Status type with the same name. This section describes that Status type.

## Status Verification Method

Since the proof in an `EthereumAttestationRegistry2019` Verifiable Credential can be updated dynamically on an Ethereum ledger, the status of a credential is always "Valid" as long as the proof can be verified. If the issuer (or authorized party) wishes to revoke a VC, they will simply revoke the proof itself in the smart contract.

## Status/proof Revocation Method

For an issued credential to be revoked by the issuer (or any other authorized party) before its expiration date, the following steps must be taken.

1. **Step 1: Calculate Revocation Hash**
2. **Step 2: Send the Ethereum transaction**
   - Use the contract address and network listed in the `proof` object.
   - Same recommendations as for proof generation, this time using the Revocation Hash.
