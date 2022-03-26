# DID-Method-Specification
# Abstract
SNPLab Decentralized Identity Service(SNPLab DID Service) provides the infrastructure for trusted digital identity service for SNPLab MyData Service(MyD).<br>
SNPLab DID Service is based on blockchain and provides a distributed mechanism for generating and verifying DID(Decentralized Identifier).

<br />


# Introduction
SNPLab is a leading MyData service platform industry in South Korea that empowers individuals by improving their right to self-determination
regarding their personal data.<br>
MyD is a personal data trading platform service provided by SNPLab and the SNPLab DID Service provides the digital identity service for MyD service.<br>
SNPLab DID Service provides a mechanism for distributed and globally unique identifiers(DID, Decentralized Identifier).

<br>

# DID Method Syntax

## DID Naming Scheme.
A DID consists of three parts:
1. The `did` URI scheme identifier.
2. The identifier for the DID method.
3. The DID method-specific identifier.
 - The identifier of SNPLab DID method is `snplab`. So, the SNPLab DID starts with `did:snplab:`.
 - SNPLab DID Service uses Ed25519 crypto algorithm to generate the private-public key pair associated with the owner's DID.
 - The DID method-specific identifier will be the base58 encoding of the first 16 bytes of DID owner's Ed25519 public key.

An example of the SNPLab DID
```
"did:snplab:V4SGRU86Z58d6TV7PBUe6f"
```

## DID Document Format

DID Document format example
```json
{
   "@context": "https://www.w3.org/ns/did/v1",
   "id": "did:snplab:V4SGRU86Z58d6TV7PBUe6f",
   "created": "2022-03-15 02:36:24",
   "verificationMethod": [
     {
        "id": "did:snplab:V4SGRU86Z58d6TV7PBUe6f#verkey",
        "type": "Ed25519VerificationKey2018",
        "publicKeyBase58": "GJ1SzoWzavQYfNL9XkaJdrQejfztN4XqdsiV4ct3LXKL",
        "controller": "did:snplab:V4SGRU86Z58d6TV7PBUe6f"
      }
   ],
   "updated": "2022-03-15 02:36:24",
   "authentication": [
       "did:snplab:V4SGRU86Z58d6TV7PBUe6f#verkey"
   ]
}
```


## CRUD Operations
CRUD operations store, read, update and delete the created DID in the blockchain.

### Getting Nonce
A nonce is used as a challenge in update and delete operations in order to verify that the DID owner is in possession of the private key associated with the DID.<br>
Consequently, a nonce SHOULD be acquired before update and delete operations are performed.

Nonce Request curl example
```bash
curl -X POST "https://did.operation.myd.world/account/nonce" \
     -H "accept: application/json" \
     -H "Content-Type: application/json" \
     -d "{\"did\": \"did:snplab:V4SGRU86Z58d6TV7PBUe6f\"}"
```
Nonce response example
```json
{"nonce":"17bf271c-4373-43b2-9eec-332505a262cd"}
```
<br>

### Create Operation
This operation allows the DID owner to register the SNPLab DID and the DID Document.<br>

The create operation is as follows:
- DID owner creates Ed25519 private-public key pair and prepares the DID Document.
- The DID Document is submitted to the SNPLab DID service via the REST API(https://did.operation.myd.world/account/register)
    - The REST API HTTP request SHOULD contain the DID Document in the request message body.<br>
    - On submission, SNPLab DID Service will trigger blockchain ledger transaction for the DID and DID Document registration.
    - If there are no DID conflicts in the blockchain and the DID along with the DID Document are written into blockchain, then the operation succeeds,
- SNPLab DID Service returns the DID Document with created and updated time information.

DID Create(Registration) curl exmaple.
```bash
curl -X POST "https://did.operation.myd.world/account/register" \
     -H "accept: application/json" \
     -H "Content-Type: application/json" \
     -d "{ \"@context\": \"https://www.w3.org/ns/did/v1\", \
           \"id\": \"did:snplab:V4SGRU86Z58d6TV7PBUe6f\", \
           \"verificationMethod\": [ { \"id\": \"did:snplab:V4SGRU86Z58d6TV7PBUe6f#verkey\", \
                                       \"type\": \"Ed25519VerificationKey2018\", \
                                       \"controller\": \"did:snplab:V4SGRU86Z58d6TV7PBUe6f\", \
                                       \"publicKeyBase58\": \"GJ1SzoWzavQYfNL9XkaJdrQejfztN4XqdsiV4ct3LXKL\" } ], \
           \"authentication\": [ \"did:snplab:V4SGRU86Z58d6TV7PBUe6f#verkey\"]}"
```

Create operation response example.
```json
{
   "@context": "https://www.w3.org/ns/did/v1",
   "id": "did:snplab:V4SGRU86Z58d6TV7PBUe6f",
   "created": "2022-03-15 02:36:24",
   "verificationMethod": [
     {
        "id": "did:snplab:V4SGRU86Z58d6TV7PBUe6f#verkey",
        "type": "Ed25519VerificationKey2018",
        "publicKeyBase58": "GJ1SzoWzavQYfNL9XkaJdrQejfztN4XqdsiV4ct3LXKL",
        "controller": "did:snplab:V4SGRU86Z58d6TV7PBUe6f"
      }
   ],
   "updated": "2022-03-15 02:36:24",
   "authentication": [
       "did:snplab:V4SGRU86Z58d6TV7PBUe6f#verkey"
   ]
}
```
<br>

### Read Operation
This operation allows a lookup of a registered DID Document from a DID.

The lookup is performed via the REST API(https://did.operation.myd.world/account/document).<br>
   - The HTTP request SHOULD contain the DID for lookup in the request message body.<br>

SNPLab DID Service returns the DID Document if there is the matched DID.

DID Read curl example.
```bash
curl -X POST "https://did.operation.myd.world/account/document" \
     -H "accept: application/json" \
     -H "Content-Type: application/json" \
     -d "{\"did\": \"did:snplab:V4SGRU86Z58d6TV7PBUe6f\"}"
```

Read operation response example.
```json
{
   "@context": "https://www.w3.org/ns/did/v1",
   "id": "did:snplab:V4SGRU86Z58d6TV7PBUe6f",
   "created": "2022-03-15 02:36:24",
   "verificationMethod": [
     {
        "id": "did:snplab:V4SGRU86Z58d6TV7PBUe6f#verkey",
        "type": "Ed25519VerificationKey2018",
        "publicKeyBase58": "GJ1SzoWzavQYfNL9XkaJdrQejfztN4XqdsiV4ct3LXKL",
        "controller": "did:snplab:V4SGRU86Z58d6TV7PBUe6f"
      }
   ],
   "updated": "2022-03-15 02:36:24",
   "authentication": [
       "did:snplab:V4SGRU86Z58d6TV7PBUe6f#verkey"
   ]
}
```
<br>

### Update Operation
This operation allows a DID owner to update their public key associated with the DID after proving possession of the existing private key.

The update operation is as follows:
- DID owner requests a nonce from SNPLab DID Service
- Generates the signature of the nonce with the private key as a challenge to prove the DID owner's possession of the existing private key.
- Generates a new DID Document containing the new public key.
- Updates the DID to associate with a new public key via REST API(https://did.operation.myd.world/account/update).
   - The HTTP request SHOULD contain the signature of nonce in the request message header.
   - The HTTP request SHOULD contain the new DID Document which has new public key in the request message body.
- SNPLab DID Service returns the DID Document with updated public key and updated time information.

DID Update curl example.
```bash
curl -X POST "https://did.operation.myd.world/account/update" \
     -H "accept: application/json" \
     -H "Content-Type: application/json" \
     -H "Signature: pV6X93hyXpu1A9Fjhdmj3unacLn0FGfCUzwnoDPIhHF6D8lsP+nSbmuMtkXx1FkUXeDIm3ZPhzUtkJlCHpihADE3YmYyNzFjLTQzNzMtNDNiMi05ZWVjLTMzMjUwNWEyNjJjZA==" \
     -d "{ "id":"did:snplab:V4SGRU86Z58d6TV7PBUe6f", \
           "verificationMethod":[{"publicKeyBase58":"5Pw9ffk5qC8sXaJq4G6YAMJ5HXgDe7qtXD2TGB2e7DSB", \
                                 "controller":"did:snplab:V4SGRU86Z58d6TV7PBUe6f", \
                                 "id":"did:snplab:V4SGRU86Z58d6TV7PBUe6f#verkey"\ ,
                                 "type":"Ed25519VerificationKey2018"}],
           "@context":"https://www.w3.org/ns/did/v1", \
           "authentication":["did:snplab:V4SGRU86Z58d6TV7PBUe6f#verkey"]}"
 ```


Update operation response example.
```json
{
   "@context": "https://www.w3.org/ns/did/v1",
   "id": "did:snplab:V4SGRU86Z58d6TV7PBUe6f",
   "created": "2022-03-15 02:36:24",
   "verificationMethod": [
     {
        "id": "did:snplab:V4SGRU86Z58d6TV7PBUe6f#verkey",
        "type": "Ed25519VerificationKey2018",
        "publicKeyBase58": "5Pw9ffk5qC8sXaJq4G6YAMJ5HXgDe7qtXD2TGB2e7DSB",
        "controller": "did:snplab:V4SGRU86Z58d6TV7PBUe6f"
      }
   ],
   "updated": "2022-03-15 02:38:51",
   "authentication": [
       "did:snplab:V4SGRU86Z58d6TV7PBUe6f#verkey"
   ]
}
```
<br>

### Delete Operation
This operation allows a DID owner to disable their DID and DID Document by proving possession of the private key associated with the DID.

The delete operation is as follows:
- DID owner requests a nonce from SNPLab DID Service.
- Generates the signature of the nonce with the private key as a challenge to prove the DID owner's possession of the private key.
- Disables the DID and DID Document via REST API(https://did.operation.myd.world/account/breakaway).
   - The HTTP request SHOULD contain the signature of nonce in the request message header.
   - The HTTP request SHOULD contain the DID to be disabled in the request message body.

DID Delete curl example.
```bash
curl -X POST "https://did.operation.myd.world/account/breakaway" \
     -H "accept: application/json" \
     -H "Content-Type: application/json" \
     -H "Signature: pV6X93hyXpu1A9Fjhdmj3unacLn0FGfCUzwnoDPIhHF6D8lsP+nSbmuMtkXx1FkUXeDIm3ZPhzUtkJlCHpihADE3YmYyNzFjLTQzNzMtNDNiMi05ZWVjLTMzMjUwNWEyNjJjZA==" \
     -d "{ \"did\": \"did:snplab:V4SGRU86Z58d6TV7PBUe6f\"}"
```

Delete operation response example.
```json
{"breakaway":"complete","did":"did:snplab:V4SGRU86Z58d6TV7PBUe6f"}
```
<br>

## Security Considerations
1. All communication between DID owner and SNPLab DID Service use TLS 1.2 and higher version with strong cipher suit.
2. SNPLab DID service uses a permissioned, distributed ledger (Hyperledger Indy) to mitigate attacks like eavesdropping, replay, message insertion, deletion, modification, impersonation, and man-in-the-middle attacks.
3. CRUD operations require for MyD application to have the access information to SNPLab DID service network. The access information wil be acquired after device attestation which MyD application is installed.
4. DID Document, private key and public key are created by DID owner and SNPLab DID service can have only public information(DID Document and public key).
5. The DID identifier of the SNPLab DID Service is generated by DID owner with the public key, so it supports the DID owner's own private key.
<br>


## Privacy Considerations
1. The DID Document of DID does not contain any personal identifiable information (PII).
2. CRUD operation requests does not contain any personal identifiable information (PII).
3. SNPLab DID service blockchain does not store any personal identifiable information (PII) or credentials.
