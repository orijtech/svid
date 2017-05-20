# SPIFFE Verifiable Identity Document (SVID)

> This document describes how SPIFFE identities and supporting documents are structured. It is *not* a specification for SPIFFE itself, nor a design for a SPIFFE-compliant system.

The [specification](SPECIFICATION.md) outlines how SPIFFE identities are defined, and how these identities can be encoded in an X.509 certificate. This encoding and naming scheme together define the SPIFFE Verifiable Service Identity Document (SVID). Two services each presenting SVID, along with access to a root bundle, should be able to mutually authenticate themselves.

At heart SPIFFE represents a mechanism for ensuring a set of services running across disparate infrastructure are granted unique identities and a mechanism to support mutual verification of those identities.

> While this specification is in draft status the open issues will be discussed and closed out by the [SPIFFE Certificate Format SIG](https://docs.google.com/document/d/1pSUGC4Ye0Mfq3sM7PTkVnLqzR8I671Na82LTDF_zLrU/edit#). You find a list of issues under discussion in the agenda and minutes of the SIG, if you want to participate [learn how to join](https://github.com/spiffe/community/blob/master/sig-list.md).