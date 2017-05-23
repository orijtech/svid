# SPIFFE Verifiable Identity Document (SVID) Document Specification

**Document Version**: 0.4	**Status**: Draft

## Revision History

Version | Description | Date
--------|-------------|-----
0.1 | Initial document, derived from discussion in the SPIFFE SRI document.	| 5/5/2017
0.2 | Added examples. Updated based on SPIFFE cert-format SIG meeting | 5/15/2017
0.3 | Removed version field from SAN | 5/17/2017
0.4 | Changed name to SPIFFE Verifiable Identity Document | 5/20/2017

## Background
SPIFFE describes both a API specification for and a reference implementation of a software framework for issuing identities to workloads. A part of the SPIFFE specification must necessarily include (a) an agreed format for naming identities (the “SPIFFE name”), and (b) is a set of documents that may be issued by the SPIFFE framework that can subsequently be used by two independently hosted workloads to mutually verify each other’s identity without a third party. 

These are defined together as the “SPIFFE Verifiable Identity Document” (SVID) specification. In practice, the SVID specification describes how SPIFFE names are to be constructed and a specific subset of the X.509 (RFC 5280) specification that should be followed in order to properly encode a SPIFFE name into an X.509 certificate. A system that implements the SPIFFE SVIDs according to this specification is considered a “SPIFFE SVID provider”.

A set of SPIFFE SVID providers should be able to pre-negotiate mutual trust ahead of time, such that identities issued by any such provider can be verified by workloads supported by any other trusted SVID provider. Thus the specification thus also describes the set of root certificates that should be made available to a workload to allow it to verify any SVID issued by a mutually trusted provider. The SVID specification does not yet cover how such mutual trust is established between providers.

From the perspective of a workload, SVIDs allows an unambiguous and best-practice process for verifying the identity of another workload (even when at identity is issued by a different but trusted provider) that can be easily implemented by many existing PKI libraries and tools.

It should be noted that the SVID specification does not reason directly about how two services might exchange these documents (for example, as part of an mTLS handshake).

## Specification

### SPIFFE name format
Services in a SPIFFE enabled system will be named by a URI in the following format:

    spiffe://trust-domain/path

#### Trust Domain (TD)
In this URI the TD corresponds to the trust root of a system, that is it can be assumed that the infrastructure that assigned identities under that domain has had trust pre-established prior to issuing those identities. A trust domain could represent an individual, organization or department running their own independent SPIFFE infrastructure.

In practical terms, identity documents issued for identities under the same Trust Domain is anchored by a common root certificate.

Trust domains are nominally self-registered, unlike public DNS there is no delegating authority that acts to assert and register a base domain to an actual legal real-world entity, or assert that legal entity has fair and due rights to any particular TD.

Negotiation of trust between services with a different TD is out of scope of this design document, but is expected to be addressed in future versions of SPIFFE.

#### Path
The path component of a SPIFFE name allows for the unique identification of a given workload. The meaning behind the path is left open ended and the responsibility of the administrator to define. 

Paths are hierarchical - similar to filesystem paths. The specific meaning of paths is reserved as an exercise to the implementer and are outside the SVID specification. However some examples and conventions are expressed below.

#### Examples
##### Identifying services directly
Often it is valuable to identify services directly. For example, an administrator may decree that any process running on a particular set of nodes should be able to present itself as a particular identity. For example:

	spiffe://staging.acme.com/payments/mysql

or

	spiffe://staging.acme.com/payments/web-fe

The two SPIFFE names above refer to two different components - the mysql database service and a web front-end - of a payments service running in a staging environment. The meaning of ‘staging’ as an environment, ‘payments’ as a high level service collection is defined by the implementer.

##### Identifying service owners
Often higher level orchestrators and platforms may have their own identity concepts built in (such as Kubernetes service accounts, or AWS/GCP service accounts) and it is helpful to be able to directly map SPIFFE identities to those identities. For example:

	spiffe://k8s-west.acme.com/ns/staging-ns/sa/default

In this example, the administrator of acme.com is running a Kubernetes cluster k8s-west.acme.com, which has a ‘staging’ namespace, and within this a service account (sa) called ‘default’. These are conventions defined by the spiffe administrator, not assertions guaranteed in SPIFFE SVID format.

### SPIFFE issued X.509 certificate format
While a SPIFFE name identifies a service (just as a person’s name identifies a person), it does not provide cryptographic proof of identity. For that we need an identity document. SPIFFE identity documents are a subset of the X.509 certificate format.
Certificates in SPIFFE may be either root, intermediate or leaf certificates. Only a leaf certificate is considered to correspond to an actual SPIFFE service.

Root and Intermediate certificates do not verify a SPIFFE name, but can be used by an issuing authority to generate SPIFFE certificates.
Leaf certificate format.

SPIFFE will distribute X.509 certificates that allow a workload running in a SPIFFE system to verify its identity. This requires conforming to a specific set of conventions within the X.509 specification, specifically:

Field | Description
------|------------
Subject Alternate Name (SAN) | SPIFFE certificates will specify a SAN, and the SPIFFE name will be encoded as a URI in the SAN, in the following format: `uri:<spiffe_name>` where `spiffe_name` is the name described above. For example: `uri:spiffe://staging.acme.com/payments/mysql`. A SVID may include at most one `spiffe://` URI SAN per certificate (though additional hostnames may be allowed). 
Authority Key Identifier (AKID) | SPIFFE certificates will specify an AKID which will provide the key identifier of the Issuing CA that signed the certificate. The AKID value would match the SKID value of the intermediate CA certificate.
Subject Key Identifier (SKID) |Used in path building before path validation. 
Basic Constraint | The extension indicates whether a certificate is a CA or not. BasicConstraints  is set to `CA:false` and is marked critical for leaf certificates. SVIDs must not use `pathLenConstraint`.
Extended Key Usage | Define what a certificate can be used for. Leaf certificates are constrained `id-kp-serverAuth` and `id-kp-clientAuth`.
Key Usage | The following keys must be marked as critical: `nonRepudiation`, `digitalSignature`, `keyEncipherment`.

#### Leaf Certificate Validation 
For a SPIFFE leaf certificate to be considered valid, all chain certificates must be checked and not have expired, and have a valid signature. 

SPIFFE will use path validation to determine if the certificate is valid. When an agent or workload request the set of SPIFFE certificates for a workload, the intermediate certificates will also be distributed to the Node Agent.

#### Example Leaf Certificate 
```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            60:98:1b:80:56:e9:d7:81:58:a0:45:58:02:10:aa:eb:a7:68:bb:56
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: O=acme.com, CN=acme.com Intermediate CA
        Validity
            Not Before: Mar 28 03:04:40 2017 GMT
            Not After : Apr  7 03:04:40 2017 GMT
        Subject: C=US, ST=CA, L=San Francisco, O=SPIFFE_CO, CN=Blog
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
            RSA Public Key: (2048 bit)
                Modulus (2048 bit):
                    …….. 
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Subject Alternative Name: 
                URI:spiffe://dev.acme.com/foo-service, 
            X509v3 Basic Constraints: critical
                CA:FALSE
           
            X509v3 Extended Key Usage: 
                TLS Web Client Authentication, TLS Web Server Authentication
            X509v3 Subject Key Identifier: 
                64:56:52:B2:7B:DF:86:EC:AD:5D:89:86:5C:C8:FD:F7:4D:FE:B5:73
            X509v3 Authority Key Identifier: 
                keyid:4E:E5:C3:6D:4F:CB:1F:63:6C:B4:B7:72:EB:C6:CC:1D:D9:7F:1D:B0

    Signature Algorithm: sha256WithRSAEncryption
          ……. 
```


