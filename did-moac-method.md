# MOAC DID Method Specification

## Introduction
The MOAC distributed identity system aims to provide a secure, robust and flexible implementation of the DID and Verifiable Claims specifications published by the W3C and the Decentralised Identity Foundation. It’s core technologies are the MOAC blockchain.
### MOAC
MOAC is a public permissionless blockchain and smart-contract execution environment. The Virtual Machine system provides certainty and security about code execution by being based upon a public blockchain.
## Overview
The MOAC DID method uses MOAC blockchain as a decentralized storage layer for DID Documents. A deployed smart-contract provides a mapping from a DID to an MOAC blockchain hash address of the corrosponding DID Document. This enables DID Documents on MOAC blockchain to be effectively addressed via their DIDs. 

## MOAC DID Method Specification
### MOAC DID Method Name:
The namestring that shall identify this DID method is: `moac`.
A DID that uses this method **MUST** begin with the following prefix: `did:moac`. According to DID specification, this string MUST be in lowercase.
### Method DID Format
MOAC DIDs are identifiable by their did\:moac: method string and conform to the [Generic DID Scheme](https://w3c-ccg.github.io/did-spec/#the-generic-did-scheme).

Example `moac` DID:
`did:moac:2f2b37c890824242cb9b0fe5614fa2221b79901e`

### Method Specific Identifier:
The method specific identifier is represented as the hex-encoded Moac address on the target network.

    did-moac = "did:moac" {Address}
    did-moac-specific-idstring = [BaseChain/AppChain":"]  {Address}
    did-moac-network = "moac Chain"/"Test-Net Chain"/"Private Chain"
Note:
1. The address consists of 40 numbers or letters and is not case sensitive.
2. If the did-moac-specific-idstring defaults,
## CRUD Operation Definitions
### DID Creation
The creation of a DID follows a few steps:
1. Generate 32 bytes of entropy
2. From the entropy, generate an MOAC blockchain key pair
3. From the MOAC blockchain key pair, take the keccak256 hash of the public key

The hash from step 3 is your DID.

**The Definition of Moac JSON_LD Context is:**
```jsonld
{
  "@context": "https://w3id.org/did/v1",
  "id": "did:mc:2f2b37c890824242cb9b0fe5614fa2221b79901e",
  "publicKey": [{
    "id": "did:moac:2f2b37c890824242cb9b0fe5614fa2221b79901e",
    "type": "Secp256k",
    "owner":"2f2b37c890824242cb9b0fe5614fa2221b79901e",
    "mcAddress":"2f2b37c890824242cb9b0fe5614fa2221b79901e"
    }],
  "authentication": [
  "type":"Secp256k",
  "publicKey":"did:moac:2f2b37c890824242cb9b0fe5614fa2221b79901e#owner"
  ]
}
```
### DID Registration/Anchoring
DID Anchoring refers to creating the mapping from DID to MOAC blockchain address on the smart-contract using the setRecord function, effectively 'anchoring' the DID to the smart-contract and making its DID Document accessible  with only the DID. This anchoring process is analogous the Create step of a CRUD database. This process is also tied to the creation of the DID, as the MOAC blockchain key pair used to generate the DID should also be the one to perform anchoring transaction on the smart-contract.

### DID Document Resolution (Read)
DID Document resolution is achieved by querying the registry smart-contract getRecord function with a DID. If that DID is registered, an MOAC blockchain address will be returned. Otherwise, an empty address is returned. This MOAC blockchain address can then be resolved through smart-contract to the DID Document. Moac provides a lot of different methods that will return content based on the request.

Format:
```jsonld
{
  "@context": "https://w3id.org/did/v1",
  "jsonrpc": "2.0",
  "method": {method requested name},
  "result": {param content returned by json format}
}
```
Example:
```jsonld
{
  "@context": "https://w3id.org/did/v1",
  "jsonrpc": "2.0",
  "method": "mc_accounts",
  "result": [2f2b37c890824242cb9b0fe5614fa2221b79901e,(other addresses belong yours)]
}
```
Example:
```jsonld
{
  "@context": "https://w3id.org/did/v1",
  "jsonrpc": "2.0",
  "method": "mc_getContractInfo",
  "result": [
    "contract-address": "0x2f2b37c890824242cb9b0fe5614fa2221b79901e",
    "contract-type":
      "ERC20",
    "createId": (
      "did:mc:2f2b37c890824242cb9b0fe5614fa2221b79901e"
      .....
    )
  ]
}
```

### DID Document Updating and Deleting
MOAC blockchain addresses are hashes of their content, so an updated DID Document will also have a new MOAC blockchain address. Thus updating simply uses the setRecord smart-contract function again with the same DID and the new MOAC blockchain hash of the updated DID Document. Deletion, similarly, is updating the registry to return an all-0 byte string, as if it had never been initialized.
## Key Management
### Key Recovery
Writing to the MOAC registry mapping contract requires control over the private key used to anchor the DID, so any key recovery method which applies to MOAC blockchain keys can be applied here. This can involve seed phrases stored in an analogue manner among other methods.
### Key Revocation
As there is no centralised control of the registry contract, no party can revoke the keys of DID control of another party under the MOAC system.

## Privacy and Security Considerations
### Key Control
As mentioned in the Key Recovery section, the entity which controls the **private key** which anchored the DID also effectively controls the DID Document which the DID resolves to. Thus great care should be taken to ensure that the **private key** is kept private. Methods for ensuring key privacy are outside the scope of this document.

### DID Document Public Profile
The DID Document anchored with the registry contract can contain any content, though it is recommended that it conforms to the [W3C DID Document Specificaiton](https://w3c-ccg.github.io/did-spec/#did-documents). As registered DIDs can be resolved by anyone, care should be taken to only update the registry to resolve to DID Documents which DO NOT expose any sensitive personal information, or information which you may not wish to be public.

### Sensitive Personal Information
Keep personally-identifiable information (PII) off-ledger. We consider that personal information is mostly sensitive content and DID documents may be parsed by others. Therefore, care should be taken to ensure that the DID document does not contain any sensitive personal information. Personal information in the real world is not recommended for inclusion in the document. So others cannot contact someone in the real world by parsing this personal DID document. 

### The permission of "Read"
People with special needs can create a private chain by setting the ID value of chain. Whether the person who is not involved in the private chain or the public chain has the authority to read the data (including the contract). Although we understand that owners of private chains may be reluctant to be read by others, they can do so by hiding the ID of their own chain. But the permission on the future public chain may be controversial.

************** **TODO** *****************
## References
[1]. MOAC foundation, https://www.moac.io

[2]. MOAC project github, https://github.com/MOACChain

[3]. W3C Decentralized Identifiers (DIDs) v0.13, https://w3c-ccg.github.io/did-spec
