# 0138: Contexts for Rich Schema Objects
- Name: rich-schema-contexts
- Author: Ken Ebert ken@sovrin.org, Brent Zundel brent.zundel@evernym.com
- Start Date: 2019-06-07T13:51:17-06:00

## Status
- Status: [PROPOSED](/README.md#hipe-lifecycle)
- Status Date: 2019-06-18
- Status Note: just proposed; community hasn't studied yet 

## Summary
[summary]: #summary

Every rich schema object has an associated `@context`. Contexts are JSON
objects. They are the standard mechanism for defining shared semantic
meaning among rich schema objects.

## Motivation
[motivation]: #motivation

`@context` is JSON-LD’s namespacing mechanism. Contexts allow schemas,
mappings, presentations, etc. to use a common vocabulary when referring to
common attributes, i.e. they provide an explicit shared semantic meaning.

## Tutorial
[tutorial]: #tutorial

### Intro to @context
`@context` is a JSON-LD construct that allows for namespacing and the
establishment of a common vocabulary.

### Stored on ledger
`@context` will be written to the ledger in much the same way that schemas
and credential definitions are written to the ledger now.

### Example context
```
"@context": [
    "did:sov:UVj5w8DRzcmPVDpUMr4AZhJ:7:example:1.0",
    "did:sov:AZKWUJ3zArXPG36kyTJZZm:7:base-context:1.0",
    "did:sov:9TDvb9PPgKQUWNQcWAFMo4:7:new-person:3.5",
    {
          "dct": "http://purl.org/dc/terms/",
          "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
          "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
          "Driver": "did:sov:35qJWkTM7znKnicY7dq5Yk:8:driver:2.4",
          "DriverLicense": "did:sov:Q6kuSqnxE57waPFs2xAs7q:8:driver-license:3.5",
          "CategoryOfVehicles": "DriverLicense:CategoryOfVehicles"
    }
]
```

### Indy and Aries
The complete architecture for `@context` objects involves three separate
repositories:
- `indy-node`: The code run by a validator node participating in an
instance of the indy ledger, e.g., the validators node in the Sovrin
network run `indy-node`. Changes to this code will enable `@context`
objects to be written to and retrieved from an instance of indy.
- `indy-data-manager`: code which a client may use to communicate with
validator nodes in an indy network. Changes to this code will enable
`@context` transaction requests to be sent to validator nodes.
`indy-data-manager` complies with the interface described by the
`aries-data-registry-interface` and is built to plug in to the aries
ecosystem.
- `aries-dri`: This is the location of the `aries-data-registy-interface`.
Changes to this code will enable users of any data registry with an
`aries-dri`-compatible data manager to handle `@context` objects.

Only changes to the indy repositories are described here. For a description
of the changes to aries, please see
[this rfc](https://github.com/hyperledger/aries-rfcs/tree/master/features/0249-rich-schema-contexts).


### Indy Node context API
Indy Node processes ledger transaction requests via request handlers.
Adding `SET_CONTEXT` and `GET_CONTEXT` ledger transactions will involve
creating both a `write` request handler, and a `read` request handler.

The numerical code for a `SET_CONTEXT` transaction is 200.
The numerical code for a `GET_CONTEXT` transaction is 300.

This will be done following the pattern for `schema_handler.py` and
`get_schema_handler.py`

#### SET_CONTEXT
Adds a context to the ledger.

It's not possible to update existing Context. So, if the Context needs to 
be evolved, a new Context with a new version or name needs to be created.

- `data` (dict):

  Dictionary with Context's data:

  - `@context`: This value must be either:
    - a URI (it should dereference to a Context object)
    - a Context object (a dict)
    - an array of Context objects and/or Context URIs

- `meta` (dict)

  Dictionary with Context's metadata

  - `name`: Context's name string
  - `version`: Context's version string
  - `type`: "ctx"


*Request Example*:
```
{
    "operation": {
        "type": "200",
        "data":{
            "@context": [
                "did:sov:UVj5w8DRzcmPVDpUMr4AZhJ:7:example:1.0",
                "did:sov:AZKWUJ3zArXPG36kyTJZZm:7:base-context:1.0",
                "did:sov:9TDvb9PPgKQUWNQcWAFMo4:7:new-person:3.5",
                {
                    "dct": "http://purl.org/dc/terms/",
                    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
                    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
                    "Driver": "did:sov:35qJWkTM7znKnicY7dq5Yk:8:driver:2.4",
                    "DriverLicense": "did:sov:Q6kuSqnxE57waPFs2xAs7q:8:driver-license:3.5",
                    "CategoryOfVehicles": "DriverLicense:CategoryOfVehicles"
                }
            ]
        },
        "meta": {
            "name":"SimpleContext",
            "version":"1.0",
            "type": "ctx"
        },
    },
    "identifier": "L5AD5g65TDQr1PPHHRoiGf",
    "endorser": "D6HG5g65TDQr1PPHHRoiGf",
    "reqId": 1514280215504647,
    "protocolVersion": 2,
    "signature": "5ZTp9g4SP6t73rH2s8zgmtqdXyTuSMWwkLvfV1FD6ddHCpwTY5SAsp8YmLWnTgDnPXfJue3vJBWjy89bSHvyMSdS"
}
```
*Reply Example*:
```
{
    "op": "REPLY", 
    "result": {
        "ver": 1,
        "txn": {
            "type":"200",
            "protocolVersion":2,
            
            "data": {
                "ver":1,
                "data":{
                    "@context": [
                        "did:sov:UVj5w8DRzcmPVDpUMr4AZhJ:7:example:1.0",
                        "did:sov:AZKWUJ3zArXPG36kyTJZZm:7:base-context:1.0",
                        "did:sov:9TDvb9PPgKQUWNQcWAFMo4:7:new-person:3.5",
                        {
                            "dct": "http://purl.org/dc/terms/",
                            "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
                            "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
                            "Driver": "did:sov:35qJWkTM7znKnicY7dq5Yk:8:driver:2.4",
                            "DriverLicense": "did:sov:Q6kuSqnxE57waPFs2xAs7q:8:driver-license:3.5",
                            "CategoryOfVehicles": "DriverLicense:CategoryOfVehicles"
                        }
                    ]
                },
                "meta": {
                    "name":"SimpleContext",
                    "version":"1.0",
                    "type": "ctx"
                },
            },
            
            "metadata": {
                "reqId":1514280215504647,
                "from":"L5AD5g65TDQr1PPHHRoiGf",
                "endorser": "D6HG5g65TDQr1PPHHRoiGf",
                "digest":"6cee82226c6e276c983f46d03e3b3d10436d90b67bf33dc67ce9901b44dbc97c",
                "payloadDigest": "21f0f5c158ed6ad49ff855baf09a2ef9b4ed1a8015ac24bccc2e0106cd905685"
            },
        },
        "txnMetadata": {
            "txnTime":1513945121,
            "seqNo": 10,  
            "txnId":"L5AD5g65TDQr1PPHHRoiGf1|Degree|1.0",
        },
        "reqSignature": {
            "type": "ED25519",
            "values": [{
                "from": "L5AD5g65TDQr1PPHHRoiGf",
                "value": "5ZTp9g4SP6t73rH2s8zgmtqdXyTuSMWwkLvfV1FD6ddHCpwTY5SAsp8YmLWnTgDnPXfJue3vJBWjy89bSHvyMSdS"
            }]
        }
 		
        "rootHash": "5vasvo2NUAD7Gq8RVxJZg1s9F7cBpuem1VgHKaFP8oBm",
        "auditPath": ["Cdsoz17SVqPodKpe6xmY2ZgJ9UcywFDZTRgWSAYM96iA", "66BCs5tG7qnfK6egnDsvcx2VSNH6z1Mfo9WmhLSExS6b"],
		
    }
}
```

#### GET_CONTEXT

Gets a context from the ledger.

- `dest` (base58-encoded string):

    Context DID as base58-encoded string for 16 or 32 byte DID value. It 
    differs from `identifier` metadata field, where `identifier` is the DID
    of the submitter.

    *Example*: `identifier` is a DID of the read request sender, and `dest`
    is the DID of the Context.

- `meta` (dict):

  - `name` (string): Context's name string
  - `version` (string): Context's version string

*Request Example*:
```
{
    "operation": {
        "type": "300"
        "dest": "2VkbBskPNNyWrLrZq7DBhk",
        "meta": {
            "name": "SimpleContext",
            "version": "1.0",
            "type": "ctx"
        },
    },
    
    "identifier": "L5AD5g65TDQr1PPHHRoiGf",
    "reqId": 1514308188474704,
    "protocolVersion": 2
}
```
*Reply Example*:
```
{
    "op": "REPLY", 
    "result": {
        "type": "300",
        "identifier": "L5AD5g65TDQr1PPHHRoiGf",
        "reqId": 1514308188474704,
        
        "seqNo": 10,
        "txnTime": 1514214795,

        "state_proof": {
            "root_hash": "81bGgr7FDSsf4ymdqaWzfnN86TETmkUKH4dj4AqnokrH",
            "proof_nodes": "+QHl+FGAgICg0he/hjc9t/tPFzmCrb2T+nHnN0cRwqPKqZEc3pw2iCaAoAsA80p3oFwfl4dDaKkNI8z8weRsSaS9Y8n3HoardRzxgICAgICAgICAgID4naAgwxDOAEoIq+wUHr5h9jjSAIPDjS7SEG1NvWJbToxVQbh6+Hi4dnsiaWRlbnRpZmllciI6Ikw1QUQ1ZzY1VERRcjFQUEhIUm9pR2YiLCJyb2xlIjpudWxsLCJzZXFObyI6MTAsInR4blRpbWUiOjE1MTQyMTQ3OTUsInZlcmtleSI6In42dWV3Um03MmRXN1pUWFdObUFkUjFtIn348YCAgKDKj6ZIi+Ob9HXBy/CULIerYmmnnK2A6hN1u4ofU2eihKBna5MOCHiaObMfghjsZ8KBSbC6EpTFruD02fuGKlF1q4CAgICgBk8Cpc14mIr78WguSeT7+/rLT8qykKxzI4IO5ZMQwSmAoLsEwI+BkQFBiPsN8F610IjAg3+MVMbBjzugJKDo4NhYoFJ0ln1wq3FTWO0iw1zoUcO3FPjSh5ytvf1jvSxxcmJxoF0Hy14HfsVll8qa9aQ8T740lPFLR431oSefGorqgM5ioK1TJOr6JuvtBNByVMRv+rjhklCp6nkleiyLIq8vZYRcgIA=", 
            "multi_signature": {
                "value": {
                    "timestamp": 1514308168,
                    "ledger_id": 1, 
                    "txn_root_hash": "4Y2DpBPSsgwd5CVE8Z2zZZKS4M6n9AbisT3jYvCYyC2y",
                    "pool_state_root_hash": "9fzzkqU25JbgxycNYwUqKmM3LT8KsvUFkSSowD4pHpoK",
                    "state_root_hash": "81bGgr7FDSsf4ymdqaWzfnN86TETmkUKH4dj4AqnokrH"
                },
                "signature": "REbtR8NvQy3dDRZLoTtzjHNx9ar65ttzk4jMqikwQiL1sPcHK4JAqrqVmhRLtw6Ed3iKuP4v8tgjA2BEvoyLTX6vB6vN4CqtFLqJaPJqMNZvr9tA5Lm6ZHBeEsH1QQLBYnWSAtXt658PotLUEp38sNxRh21t1zavbYcyV8AmxuVTg3",
                "participants": ["Delta", "Gamma", "Alpha"]
            }
        },
        
        "data":{
            "@context": [
                "did:sov:UVj5w8DRzcmPVDpUMr4AZhJ:7:example:1.0",
                "did:sov:AZKWUJ3zArXPG36kyTJZZm:7:base-context:1.0",
                "did:sov:9TDvb9PPgKQUWNQcWAFMo4:7:new-person:3.5",
                {
                    "dct": "http://purl.org/dc/terms/",
                    "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
                    "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
                    "Driver": "did:sov:35qJWkTM7znKnicY7dq5Yk:8:driver:2.4",
                    "DriverLicense": "did:sov:Q6kuSqnxE57waPFs2xAs7q:8:driver-license:3.5",
                    "CategoryOfVehicles": "DriverLicense:CategoryOfVehicles"
                }
            ]
        },
        
        "meta": {
            "name":"SimpleContext",
            "version":"1.0",
            "type": "ctx"
        },
        
        "dest": "2VkbBskPNNyWrLrZq7DBhk"
    }
}
```

### Indy Data Manager API
Indy Data Manager methods for adding and retrieving `@context` from the
ledger comply with the interface described
[in aries-dri](https://github.com/hyperledger/aries-rfcs/tree/master/features/0249-rich-schema-contexts).
This means we define two external-facing methods:
- `indy_read_context`
- `indy_write_context`

#### write_context
```
Writes a context to the ledger.

#Params
submitter: {
    key: public key of the submitter,
    keystore: key manager where private key is stored
}, 
data: {
    id: identifier for the context,
    context: context object,
    name: context name string,
    version: context version string,
    ver: version of the context JSON format
},
registry: identifier for the registry

#Returns
registry_response: result as json,
error: {
    code: aries common error code,
    description:  aries common error description
}
```
#### read_context
```
Reads a context from the ledger.

#Params
submitter (optional): {
    key: public key of the submitter,
    keystore: key manager where private key is stored
}, 
id: identifier for the context,
registry: identifier for the registry

#Returns
registry_response: context object,
error: {
    code: aries common error code,
    description:  aries common error description
}
```
These external methods will use internal methods which follow the common
pattern for methods in Indy-SDK that interact with the ledger. There is a
single method call to build a request to add a transaction to the ledger,
another to build a request to retrieve a transaction from the ledger, and a
third to parse the response from the ledger after submitting a request to
retrieve a transaction. 

The three internal methods we propose adding:
- `indy_build_set_context_request`
- `indy_build_get_context_request`
- `indy_parse_get_context_response`


#### indy_build_set_context_request
```
Builds a SET_CONTEXT request. Request to add a context to the ledger.

#Params
command_handle: command handle to map callback to execution environment.
submitter_did: DID of the submitter stored in secured Wallet.
data: Context.
{
    id: identifier the context,
    context: context object,
    name: Context's name string
    version: Context's version string,
    ver: Version of the Context json
}
cb: Callback that takes command result as parameter.

#Returns
Request result as json.

#Errors
Common*
```
#### indy_build_get_context_request
```
Builds a GET_CONTEXT request. Request to get a context from the ledger.

#Params
command_handle: command handle to map callback to execution environment.
submitter_did: (Optional) DID of the read request sender (if not provided then default Libindy DID will be used).
id: context ID in ledger
cb: Callback that takes command result as parameter.

#Returns
Request result as json.

#Errors
Common*
```
#### indy_parse_get_context_response
```
Parse a GET_CONTEXT response to get context json.

#Params
command_handle: command handle to map callback to execution environment.
get_context_response: response of GET_CONTEXT request.
cb: Callback that takes command result as parameter.

#Returns
Context id and context json.
{
    id: identifier of context
    context: array of context values
    name: context's name string
    version: context's version string
    ver: Version of the context json
}

#Errors
Common*
```

## Reference
[reference]: #reference

More information on the Verifiable Credential data model use of `@context`
may be found [here](https://w3c.github.io/vc-data-model/#contexts)

More information on `@context` from the JSON-LD specification may be found
[here](https://w3c.github.io/json-ld-syntax/#the-context) and
[here](https://w3c.github.io/json-ld-syntax/#advanced-context-usage).

## Drawbacks
[drawbacks]: #drawbacks
Requiring a `@context` for each rich schema object introduces more
complexity.

Implementing an Indy-Node ledger transaction for `@context` and
accompanying Indy-SDK methods for submitting and retrieving `@context`
transactions in a way that follows the existing methodology may increase
the existing technical debt that is found in those libraries.

## Rationale and alternatives
[alternatives]: #alternatives

Though requiring a `@context` for each rich schema object increases the
complexity of the system, it also provides a means for better managing the
complexity already present.

## Unresolved questions and future work
[unresolved]: #unresolved-questions

- Should the GUID portion of the DID which identifies a `@context` be taken
from the DID of the transaction submitter, or should there be established
a common DID to be associated with all immutable content such as `@context`?

- Discovery of `@context` objects on the ledger is not considered part of
this initial phase of work.

- Adding `@context` functionality to the Indy-SDK language wrappers is not
considered part of this initial phase of work.
