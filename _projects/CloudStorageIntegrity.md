---
layout: page
title: Cloud Storage Integrity
description:
img: 
importance: 2
category: Projects
related_publications: false
---

From my Network Security grad class and a talk by [Dr. Roche](https://roche.work/) made me think, **_"Can I be sure the cloud providers aren't violating my data's integrity?"_**

## Data Generation
For data generation, I use openssl,
```sh
$ openssl rand <size_bytess>  > data_<size_bytess>.bin
```

## Methodology
I will be computing the SHA256 hash for these generated data and storing it locally. The data will be uploaded to local and commercial data storage options. After which the local data will be deleted.

## Verification
<u><b>Naive Approach</b></u>: Download the entire data, hash it and verify.

<u><b>Naive but Smarter Approach</b></u>: Chop the intial data into $N$ chunks, compute the hash for each chunk $h_N$. Download some $k$ chunks, recompute their hashes and if you succeed, you can say with reasonable accuracy that the data has not been tampered with. 

_**Note**_:_I am currently reviewing literate to read more methods for verification._
