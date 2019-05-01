---
layout: post
title:  "Daily paper: EnclaveDB, Speicher and other trusted databases"
date:   2019-04-16 23:00:28 -0500
categories: jekyll update
---

# Daily paper: EnclaveDB, Speicher and other trusted databases

## EnclaveDB

EnclaveDB builds a trusted database using SGX based on Hekaton, an in-memory database engine for SQL server. The major limitation for EnclaveDB is that it only evalutes small data (<128MB). The memory encryption and page evication overhead are not discussed. And as always, the side-channel security is not considered. 

## Speicher

Speicher builds a trusted key-value store using SGX based on RocksDB. 
