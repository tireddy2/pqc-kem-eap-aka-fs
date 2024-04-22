---
title: "Post-Quantum Key Encapsulation Mechanisms (PQ KEMs) in EAP-AKA prime"
abbrev: "PQ KEMs in EAP-AKA prime"
category: std

docname: draft-ra-emu-pqc-eapaka-00
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "EMU"
keyword:
 - PQC
 - EAP-AKA'


venue:
  group: "emu"
  type: "Working Group"
  mail: "emu@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/emu/"


stand_alone: yes
pi: [toc, sortrefs, symrefs, strict, comments, docmapping]

author:
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "kondtir@gmail.com"
 -
    fullname: Aritra Banerjee
    organization: Nokia
    city: Munich
    country: Germany
    email: "aritra.banerjee@nokia.com"

normative:
  RFC9048:

informative:

  PQCAPI:
     title: "PQC - API notes"
     target: https://csrc.nist.gov/CSRC/media/Projects/Post-Quantum-Cryptography/documents/example-files/api-notes.pdf
     date: false
  FO:
     title: "Secure Integration of Asymmetric and Symmetric Encryption Schemes"
     target: https://link.springer.com/article/10.1007/s00145-011-9114-1
     date: false
  HHK:
     title: "A Modular Analysis of the Fujisaki-Okamoto Transformation"
     target: https://link.springer.com/chapter/10.1007/978-3-319-70500-2_12
     date: false
  FIPS203-ipd:
     title: "Module-Lattice-based Key-Encapsulation Mechanism Standard"
     target: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.203.ipd.pdf
     date: false
  
--- abstract

Forward Secrecy for the Extensible Authentication Protocol Method for Authentication and Key Agreement (EAP-AKA' FS) is specified in {{!I-D.ietf-emu-aka-pfs}}, providing updates to {{RFC9048}} with an optional extension that offers ephemeral key exchange using the traditional Ephemeral Elliptic Curve Diffie-Hellman (ECDHE) key agreement algorithm for achieving perfect forward secrecy (PFS). However, it is susceptible to future threats from Cryptographically Relevant Quantum Computers, which could potentially compromise a traditional ephemeral public key. If the adversary has also obtained knowledge of the long-term key and ephemeral public key, it could compromise session keys generated as part of the authentication run in EAP-AKA'.

This draft aims to enhance the security of EAP-AKA' FS making it quantum-safe using Post-Quantum Key Encapsulation Mechanisms (PQ-KEMs).

--- middle

# Introduction

Forward Secrecy for the Extensible Authentication Protocol Method for Authentication and Key Agreement (EAP-AKA' FS) defined in {{I-D.ietf-emu-aka-pfs}} updates the improved Extensible Authentication Protocol Method for 3GPP Mobile Network Authentication and Key Agreement (EAP-AKA') specified in {{RFC9048}}, with an optional extension providing ephemeral key exchange. This prevents an attacker who has gained access to the long term key from obtaining session keys established in the past, assuming these have been properly deleted. EAP-AKA' FS mitigates passive attacks (e.g., large scale pervasive monitoring) against future sessions.

Nevertheless, EAP-AKA' FS uses traditional algorithms public-key algorithms (e.g., ECDH) which will be broken by a Cryptographically Relevant Quantum Computer (CRQC) using Shor's algorithm. The presence of a CRQC would render state-of-the-art, traditional public-key algorithms deployed today obsolete and insecure, since the assumptions about the intractability of the mathematical problems for these algorithms that offer confident levels of security today no longer apply in the presence of a CRQC. A CRQC could recover the SHARED_SECRET from the ECDHE public keys (Section 6.3 of {{I-D.ietf-emu-aka-pfs}}). If the adversary has also obtained knowledge of the long-term key, it could then compute CK', IK', and the SHARED_SECRET, and any derived output keys. This means that the CRQC would disable the forward security capability provided by {{I-D.ietf-emu-aka-pfs}}.

Researchers have developed Post-Quantum Key Encapsulation Mechanisms (PQ-KEMs) to provide secure key establishment resistant against an adversary with access to a quantum computer.

As the National Institute of Standards and Technology (NIST) is still in the process of selecting the new post-quantum cryptographic algorithms that are secure against both quantum and classical computers, the purpose of this document is to propose a PQ-KEMs to protect the confidentiality of content encrypted using JOSE and COSE against the quantum threat.

Although this mechanism could thus be used with any PQ-KEM, this document focuses on Module-Lattice-based Key Encapsulation Mechanisms (ML-KEMs). ML-KEM is a one-pass (store-and-forward) cryptographic mechanism for an originator to securely send keying material to a recipient using the recipient's ML-KEM public key. Three parameters sets for ML-KEMs are specified by {{FIPS203-ipd}}. In order of increasing security strength (and decreasing performance), these parameter sets are ML-KEM-512, ML-KEM-768, and ML-KEM-1024.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Terminology

This document makes use of the terms defined in {{?I-D.ietf-pquip-pqt-hybrid-terminology}}. The following terms are repeately used in this specification:

- KEM: Key Encapsulation Mechanism
- PQ-KEM: Post-Quantum Key Encapsulation Mechanism
- CEK: Content Encryption Key
- ML-KEM: Module-Lattice-based Key Encapsulation Mechanism

For the purposes of this document, it is helpful to be able to divide cryptographic algorithms into two classes:

"Traditional Algorithm":  An asymmetric cryptographic algorithm based on integer factorisation, finite field discrete logarithms or elliptic curve discrete logarithms. In the context of JOSE, examples of traditional key exchange algorithms include Elliptic Curve Diffie-Hellman Ephemeral Static {{?RFC6090}} {{?RFC8037}}. In the context of COSE, examples of traditional key exchange algorithms include Ephemeral-Static (ES) DH and Static-Static (SS) DH {{?RFC9052}}. 

"Post-Quantum Algorithm":  An asymmetric cryptographic algorithm that is believed to be secure against attacks using quantum computers as well as classical computers. Post-quantum algorithms can also be called quantum-resistant or quantum-safe algorithms. Examples of Post-Quantum Algorithm include ML-KEM.

# Background on EAP-AKA' with perfect forward secrecy

In EAP-AKA', The authentication vector (AV) contains a random part RAND, an authenticator part AUTN used for authenticating the network to the USIM, an expected result part XRES, a 128-bit session key for integrity check IK, and a 128-bit session key for encryption CK.

As described in the draft {{?I-D.draft-ietf-emu-aka-pfs-11}}, the server has the EAP identity of the peer. The server asks the AD to run AKA algorithm to generate RAND, AUTN, XRES, CK and IK. Further it also derives CK’ and IK’ keys which are tied to a particular network name. The server now generates the ephemeral key pair and sends the public key of that key pair and the first EAP method message to the peer. In this EAP message, AT_PUB_ECDHE (carries public key) and the AT_KDF_FS(carries other FS related parameters). Both of these might be ignored of USIM doesn’t support the Forward Secrecy extension. The peer checks if it wants to have a Forward extension in EAP AKA'. If yes, then it will eventually respond with AT_PUB_ECDHE and MAC. If not, it will ignore AT_PUB_ECDHE. If the peer wants to participate in FS extension, it will then generate its ECDH key pair, calculate a shared key based on its private key and server public key. The server will receive the RES from peer and AT_PUB_ECDHE. The shared key will be generated both in the peer and the server with key pairs exchanged, and later master key is also generated.

~~~
MK_ECDHE = PRF'(IK'| CK'|SHARED_SECRET,"EAP-AKA' FS"|Identity)
~~~

## Key Encapsulation Mechanisms

For the purposes of this document, we consider a Key Encapsulation Mechanism (KEM) to be any asymmetric cryptographic scheme comprised of algorithms satisfying the following interfaces {{PQCAPI}}.  

* def kemKeyGen() -> (pk, sk)
* def kemEncaps(pk) -> (ct, ss)
* def kemDecaps(ct, sk) -> ss

where pk is public key, sk is secret key, ct is the ciphertext representing an encapsulated key, and ss is shared secret.

KEMs are typically used in cases where two parties, hereby refereed to as the "encapsulater" and the "decapsulater", wish to establish a shared secret via public key cryptography, where the decapsulater has an asymmetric key pair and has previously shared the public key with the encapsulater.

# Design Rationales {#rational}

It is essential to note that in the PQ-KEM, one needs to apply Fujisaki-Okamoto {{FO}} transform or its variant {{HHK}} on the PQC KEM part to ensure that the overall scheme is IND-CCA2 secure, as mentioned in {{?I-D.ietf-tls-hybrid-design}}. The FO transform is performed using the KDF such that the PQC KEM shared secret achieved is IND-CCA2 secure. 

Note that during the transition from traditional to post-quantum algorithms, there may be a desire or a requirement for protocols that incorporate both types of algorithms until the post-quantum algorithms are fully trusted. HPKE is an KEM that can be extended to support hybrid post-quantum KEMs and the specifications for the use of HPKE with EAP-AKA prime is described in 
{{?I-D.ietf-draft-ar-emu-pqc-eapaka}}. 

# PQC KEM Enhancements by Design

We suggest the following changes and enhancements:

- A new attribute, AT_PUB_KEM, is defined to carry the PQC KEM public key from the EAP server. 

- A new attribute, AT_KEM_CT, is defined to carry the ciphertext (ct) generated by the PQC KEM Encapsulation function from the EAP peer.

- The AT_KDF_FS attribute is updated to indicate the PQC KEM for generating the Master Key MK_PQ_SHARED_SECRET.

- Multiple AT_KDF_FS attributes is included in the EAP-Request to handle the EAP peer not supporting PQC KEM.

- The PQC KEM can be included first in the AT_KDF_FS attribute in the EAP-Request to indicate a higher priority for its use compared to the traditional key derivation functions.

# KEM PQC Algorithms

The National Institute of Standards and Technology (NIST) started a process to solicit, evaluate, and standardize one or more quantum-resistant public-key cryptographic algorithms, as seen [here](https://csrc.nist.gov/projects/post-quantum-cryptography). Said process has reached its [first announcement](https://csrc.nist.gov/publications/detail/nistir/8413/final) in July 5, 2022, which stated which candidates to be standardized for KEM:

* Key Encapsulation Mechanisms (KEMs): [CRYSTALS-Kyber](https://pq-crystals.org/kyber/): ML-KEM, previously known 
 as Kyber, is a module learning with errors (MLWE)-based KEM. Three security levels have been defined in the NIST PQC Project, namely Level 1, 3, and 5. These levels correspond to the hardness of breaking AES-128, AES-192 and AES-256, respectively.

NIST announced as well that they will be [opening a fourth round](https://csrc.nist.gov/csrc/media/Projects/post-quantum-cryptography/documents/round-4/guidelines-for-submitting-tweaks-fourth-round.pdf) to standardize an alternative KEM, and a [call](https://csrc.nist.gov/csrc/media/Projects/pqc-dig-sig/documents/call-for-proposals-dig-sig-sept-2022.pdf) for new candidates for a post-quantum signature algorithm.

## ML-KEM

ML-KEM offers several parameter sets with varying levels of security and performance trade-offs. This document specifies the use of the ML-KEM algorithm at three security levels: ML-KEM-512, ML-KEM-768, and ML-KEM-1024. The main security property for KEMs standardized in the NIST Post-Quantum Cryptography Standardization Project is indistinguishability under adaptive chosen ciphertext attacks (IND-CCA2) (see Section 10.2 of {{?I-D.ietf-pquip-pqc-engineers}}). The public/private key sizes, ciphertext key size, and PQ security levels of ML-KEM are detailed in Section 12 of {{?I-D.ietf-pquip-pqc-engineers}}.

# Protocol Construction

This section defines the construction for PQC KEM in EAP-AKA' FS. 
 
## Protocol Call Flow


     USIM           Peer                        Server              AD
      |              |                            |                |
      |              |           EAP-Req/Identity |                |
      |              |<---------------------------+                |
      |              |                            |                |
      |              | EAP-Resp/Identity          |                |
      |              | (Privacy-Friendly)         |                |
      |              +--------------------------->|                |
      |      +-------+----------------------------+----------------+--+
      |      | Server now has an identity for the peer. The server    |
      |      | then asks the help of AD to run AKA algorithms,        |
      |      | generating RAND, AUTN, XRES, CK, IK. Typically, the    |
      |      | AD performs the first part of key derivations so that  |
      |      | the authentication server gets the CK' and IK' keys    |
      |      | already tied to a particular network name.             |
      |      +-------+----------------------------+----------------+--+
      |              |                            |                |
      |              |                            | ID, key deriv. |
      |              |                            | function,      |
      |              |                            | network name   |
      |              |                            +--------------->|
      |              |                            |                |
      |              |                            |    RAND, AUTN, |
      |              |                            | XRES, CK', IK' |
      |              |                            |<---------------+
      |      +-------+----------------------------+----------------+--+
      |      | Server now has the needed authentication vector. It    |
      |      | generates an a PQC KEM key pair, sends the public key  |
      |      | of that key pair and the first EAP method message      |
      |      | to the peer. In the message the AT_PUB_KEM attribute   |
      |      | carries the PQC KEM public key and the AT_KDF_FS       | 
      |      | attribute carries PQC KEM algorithm. Both of           |
      |      | these are skippable attributes that can be ignored     |
      |      | if the peer does not support this extension.           |
      |      +-------+----------------------------+----------------+--+
      |              |                            |                |
      |              |     EAP-Req/AKA'-Challenge |                |
      |              |  AT_RAND, AT_AUTN, AT_KDF, |                |
      |              |   AT_KDF_FS, AT_KDF_INPUT, |                |
      |              |      AT_PUB_KEM, AT_MAC    |                |
      |              |<---------------------------+                |
    +--+--------------+----------------------------+---------+     |
    | The peer checks if it wants to do the FS extension. If |     |
    | yes, it will eventually respond with AT_KEM_CT and     |     |
    | AT_MAC. If not, it will ignore AT_PUB_KEM and          |     |
    | AT_KDF_FS and base all calculations on basic EAP-AKA'  |     |
    | attributes, continuing just as in EAP-AKA' per RFC     |     |
    | 9048 rules. In any case, the peer needs to query the   |     |
    | auth parameters from the USIM card.                    |     |
    +--+--------------+----------------------------+---------+     |
      |              |                            |                |
      |   RAND, AUTN |                            |                |
      |<-------------+                            |                |
      |              |                            |                |
      | CK, IK, RES  |                            |                |
      +------------->|                            |                |
    +--+--------------+----------------------------+---------+     |
    | The peer now has everything to respond. If it wants to |     |
    | participate in the FS extension, it will calculate a   |     |
    | PQC KEM shared secret key based on the server's PQC    |     |
    | KEM public key. Finally, it proceeds to derive all     |     |
    | EAP-AKA' key values and  constructs a full response.   |     |
    +--+--------------+----------------------------+---------+     |
      |              |                            |                |
      |              | EAP-Resp/AKA'-Challenge    |                |
      |              | AT_RES, AT_KEM_CT,         |                |
      |              | AT_MAC                     |                |
      |              +--------------------------->|                |
      |      +-------+----------------------------+----------------+--+
      |      | The server now has all the necessary values. It        |
      |      | generates the PQC KEM shared secret and checks the RES |
      |      | and MAC values received in AT_RES and AT_MAC,          |
      |      | respectively. Success requires both to be found        |
      |      | correct. Note that when this document is used,         |
      |      | the keys generated from EAP-AKA' are based on CK, IK,  |
      |      | and PQC KEM value. Even if there was an                |
      |      | attacker who held the long-term key, only an active    |
      |      | attacker could have determined the generated session   |
      |      | keys; additionally an attacker with a cryptographically|
      |      | relevant quantum computer cannot get access to the     |
      |      | server KEM private key and decrypt the data.           |
      |      +-------+----------------------------+----------------+--+
      |              |                            |                |
      |              |                EAP-Success |                |
      |              |<---------------------------+                |
      |              |                            |                |
      

## Key Steps in protocol construction

We outline the following key steps in the protocol:

- Server generates the PQC KEM public key(pk), private key (sk) pair. The server will generate the AKA challenge and sends the EAP AKA' Authentication Vector (AV). The server PQC KEM key pair is derived as:

~~~
   sk, pk = kemKeyGen()
~~~

- The server will store the expected response XRES, the PQC KEM private key sk. The server will forward the EAP AKA' AV to peer along with pk.

- The USIM will validate the AKA challenge received, also verifies the MAC-I. After the verification is successful and if the peer also supports the Forward secrecy, peer will invoke kemEncaps using pk:

~~~
   ct, ss = kemEncaps(pk) 
~~~

"ct" is the ciphertext from PQC KEM whereas "ss" is shared secret key. 

- The peer will send the Authentication response RES and enc to the server.

- The server will verify the RES with XRES. The server will use the ct and PQC KEM private key sk to generate shared secret:

~~~
   ss = kemDecaps(ct, sk)
~~~

The generated ss from Decap is the shared secret key derived from PQC KEM. The peer and the server first generate the MK_PQ_SHARED_SECRET and subsequently generate MSK, EMSK as shown below:

~~~
   MK = PRF'(IK'|CK',"EAP-AKA'"|Identity)
   ct, ss = kemEncaps(pKR)
   MK_PQ_SHARED_SECRET = PRF'(IK'|CK'|ss,"EAP-AKA' FS"| Identity | ct)  
   K_encr = MK[0..127]
   K_aut = MK[128..383]
   K_re = MK_PQ_SHARED_SECRET [0..255] 
   MSK = MK_PQ_SHARED_SECRET [256..767] 
   EMSK = MK_PQ_SHARED_SECRET [768..1279]
~~~

where, pkR is PQC KEM public key of the receiver, ct is the ciphertext from the PQC KEM and the Encap function is perfomed by the peer only.
The pseudo-random function binds the shared secret to the ciphertext (ct), achieving MAL-BIND-K-CT. ML-KEM already is MAL-BIND-K-PK as the hash of the encapsulation key (pk) is an input to the computation of the shared secret (ss).

# Extensions to EAP-AKA' FS

## AT_PUB_KEM {#pqkem}

   The format of the AT_PUB_KEM attribute is shown below.

~~~
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     | AT_PUB_KEM    | Length        | Value                         |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
      
   The fields are as follows:

   AT_PUB_KEM:
   
   This is set to TBA1 BY IANA.

   Length:
   
   The length of the attribute, set as other attributes in EAP-AKA {{!RFC4187}}. The length is expressed in multiples of 4 bytes.  The length includes the attribute type field, the Length field itself, and the Value field (along with any padding).

   Value: 

      *  EAP-Request: It contains the public key, which is the PQC KEM public key from the EAP server.

   Because the length of the attribute must be a multiple of 4 bytes,the sender pads the Value field with zero bytes when necessary. To retain the security of the keys, the sender SHALL generate a fresh value for each run of the protocol.

## AT_PUB_CT {#pqct}

   The format of the AT_PUB_CT attribute is shown below.

~~~
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     | AT_PUB_CT     | Length        | Value                         |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
      
   The fields are as follows:

   AT_PUB_CT:
   
   This is set to TBA2 BY IANA.

   Length:
   
   The length of the attribute, set as other attributes in EAP-AKA {{!RFC4187}}. The length is expressed in multiples of 4 bytes.  The length includes the attribute type field, the Length field itself, and the Value field (along with any padding).

   Value:
      
      *  EAP-Response: It contains the ciphertext (ct) from the PQC KEM Encapsulation function from the EAP peer.


   Because the length of the attribute must be a multiple of 4 bytes,the sender pads the Value field with zero bytes when necessary. To retain the security of the keys, the sender SHALL generate a fresh value for each run of the protocol.

# Security Considerations

ML-KEM is believed to be IND-CCA2 secure based on multiple analyses. The ML-KEM variant and its underlying components should be selected consistently with the desired security level. For further clarity on the sizes and security levels of ML-KEM variants, please refer to the tables in Sections 12 and 13 of {{?I-D.ietf-pquip-pqc-engineers}}.

The security of the ML-KEM algorithm depends on a high-quality pseudo-random number generator. For further discussion on random number generation, see {{?RFC4086}}.

In general, good cryptographic practice dictates that a given ML-KEM key pair should be used in only one EAP session. This practice mitigates the risk that compromise of one EAP session will compromise the security of another EAP session and is essential for maintaining forward security.

# IANA Considerations

   Two new values (TBA1, TBA1) in the skippable range need to be assigned by IANA 
   for AT_PUB_KEM ({{pqkem}}) and AT_PUB_CT ({{pqct}}) in the "Attribute Types" registry 
   under the "EAP-AKA and EAP-SIM Parameters" group.
 
   IANA is requested to update the registry "EAP-AKA' AT_KDF_FS
   Key Derivation Function Values" with the PQC KEM algorithm entries:

~~~
   +=========+===============================+=========================+
   | Value   | Description                   | Reference               |
   +=========+===============================+=========================+
   | TBA2    | MLKEM512                      | [TBD BY IANA: THIS RFC] |
   +=========+===============================+=========================+
   | TBA3    | MLKEM768                      | [TBD BY IANA: THIS RFC] |
   +=========+===============================+=========================+
   | TBA4    | MLKEM1024                     | [TBD BY IANA: THIS RFC] |
   +=========+===============================+=========================+
~~~

--- back

# Acknowledgements

This draft leverages text from {{?I-D.draft-ietf-emu-aka-pfs-11}}.