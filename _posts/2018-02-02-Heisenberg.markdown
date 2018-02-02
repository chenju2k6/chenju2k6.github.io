---
layout: post
title:  "Heisenberg Defense Summary"
date:   2018-02-02 05:00:28 -0500
categories: jekyll update
---
Short Version
---

The basic idea for this defense is to preload the TLB so that the attacker can't know the page access pattern. They insert a security module which preloads the TLB at the enclave reentry/resume path so that the enclave is executed page-fault free.


Long Version
---

They do three things in the security module.


Verify all the enclave pages are presented in the EPC, marked present in the page table and has the correct access rights.

Ensure during the enclave execution, page tables are not accessed. 

Ensure the attack can't evict a TLB entry. 

Size Limitation:


Since there are only 1536 TLB entries, then only 6MB pages can be preloaded. For enclave size larger than 6MB, but smaller than 96MB (EPC size), they use static analysis. They claim that only secret-dependent pages need to be preloaded. 


For enclave size larger than 96MB, they have no solutions but propose to use ORAM to hide access patterns.



