---
layout: post
title:  "SGX attestation in plain words"
date:   2018-02-10 05:00:28 -0500
categories: jekyll update
---
Alice has an Enclave E. Alice uploads E to Amazon who provides secure execution service (by Intel SGX). Amazon wants to convince Alice that it is Alice’s desired E that is running in an authentic SGX CPU. If Amazon fails to do so, Alice will not pay for the service. This is accomplished by software attestation. 

A straightforward way to do this is that Intel generate a key pair for each SGX CPU, and burn the private key to a tamper-resistant storage. In addition, we assume the Intel CPU can perform complex cryptographic primitives, such as signing. Intel, as a CA, also signs a certificate which proves the binding between the identity of a CPU and its public key. After initialization, the Enclave E ask Intel CPU to sign the hash digest of the initial state of the Enclave, then sends both hash digest and its signature back to Alice. Alice, receives the signature, verifies it against the public key issued by Intel’s signed certificate. If the verification passes, Alice can be convinced that it is her Enclave E that is running on an authentic Intel CPU. An attacker can’t forge a signature and makes Alice convinced, because 1, an attacker can’t read the key due to the tamper-resistant hardware, 2, only an Enclave program can ask CPU to sign the hash digest and 3, an attack can’t forge a hash digest with different E.

In addition, Alice also wants to talks with E to setup a secure channel. It is finished by running DH key exchange protocol and by extending the above procedure. Instead of doing nothing, Alice also sends g^x. The Enclave, will prepares g^y, and asks SGX to sign both the hash digest of the enclave state and the key exchanging messages (g^x+g^y). Alice, after verifies the signature, can be convinced that it is her enclave E running on an authentic SGX CPU, and also, this Enclave E will use g^x to derive the shared secret key. 

Unfortunately, the signing is too complex to implement in hardware (but hardware can still do MAC!).  Intel resorts to a special software, named Quoting Enclave (QE) to do the signing.  If we assume that Alice trusts the QE, the left problem is for enclave E to prove its own identity to QE. This process is named as Local Attestation. Here, “local” means that both E and QE resides in one machine with the same SGX CPU. The benefit of this co-location is E and QE can have a shared secret key derived from the common secret burned in the CPU hardware. Using this shared secret key, E can ask SGX CPU to generate a MAC tag for its own state hash digest, and this message can go across the Enclave boundaries. 

The complete picture of attestation process is called Remote Attestation. Here, “remotes” means that QE depends on a remote trusted party (called Intel’s provisioning service, can be viewed as a CA) to issue the signing key. The reason for that, is that Intel is not simply putting the signing key inside of CPU during manufacturing process, this signing key is provided dynamically during the attestation process. The end result is that if Alice trusts Intel’s provisioning service, Alice will trust the Quoting Enclave, and hence the signed attested message.

Above is the attestation process in a big picture. 

However, they are many details to say. For example, the signing key (attestation key) is not following a traditional public-key scheme. Intel uses EPID, a group signature scheme. The benefit of using group signature scheme is that the anonymity of the signer is preserved.  Each signer (e.g. Quoting Enclave) can talk with CA to have a Member Private Key. The signature signed by this member private key can be verified against a single Group Public Key published by CA.

  

