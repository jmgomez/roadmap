---
title: "Frequently Asked Questions"
tags:
  - "LogosBFT"
  - "consensus"
openToc: true
---

## Data Distribution

### How much failure rate of erasure code transmission are we expecting. Basically, what are the EC coding parameters that we expect to be sending such that we have some failure rate of transmission? Has that been looked into? - Dmitriy
> `Moh:` This is a great question and it points to the tension between the failure rate vs overhead. We have briefly looked into this (today me and Marcin @madxor discussed such cases), but we haven’t thoroughly analyzed this. In our case, the rate of failure also depends on committee size. We look into $10^{-3}$ to $10^{-6}$ probability of failure. And in this case, the coding overhead can be somewhere between 200%-500% approximately. This means for a committee size of 500 (while expecting receipt of messages from 251 correct nodes), for a failure rate of $10^{-6}$ a single node has to send > 6Mb of data for a 1Mb of actual data. Though 5x overhead is large, it still prevent us from sending/receiving 500 Mb of data in return for a failure probability of 1 proposal out of 1 million. From the protocol perspective, we can address EC failures in multiple ways: a: Since the root committee only forwards the coded chunks only when they have successfully rebuilt the block. This means the root committee can be contacted to download additional coded chunks to decode the block. b: We allow this failure and let the leader be replaced but since there is proof that the failure is due to the reason that a decoder failed to reconstruct the block, therefore, the leader cannot be punished (if we chose to employ punishment in PoS). 

### How much data should a given block be. Are there limits on this and if so, what are they and what do they depend on? - Dmitriy
> `Moh:` This question can be answered during simulations and experiments over links of different bandwidths and latencies. We will test the protocol performances with different block sizes. As we know increasing the block size results in increased throughput as well as latency. What is the most appropriate block size can be determined once we observe the tradeoff between throughput vs latency.

## Signature Propagation

### Who sends the signatures up from a given committee? Do that have any leadered power within the committee? - Tanguy
> `Moh:` Each node in a committee multicasts its vote to all members of the parent committee. Since the size of the vote is small the bit complexity will be low. Introducing a leader within each committee will create a single point of failure within each committee. This is why we avoid maintaining a leader within each committee

## Network Scale

### What is our expected minimum number of nodes within the network? - Dmitriy
> For a small number of nodes we can have just a single committee. But I am not sure how many nodes will join our network 

## Byzantine Behavior

### Can we also consider a flavor that adds attestation/attribution to misbehaving nodes? That will come at a price but there might be a set of use cases which would like to have lower performance with strong attribution. Not saying that it must be part of the initial design, but can be think-through/added later. - Marcin
> `Moh:` Attestation to misbehaving nodes is part of this protocol. For example, if a node sends an incorrect vote or if a leader proposes an invalid transaction, then this proof will be shared with the network to punish the misbehaving nodes (Though currently this is not part of pseudocode). But it is not possible to reliably prove the attestation of not participation.

> `Marcin:` Great, and definitely, we cannot attest that a node was not participating - I was not suggesting that;). But we can also think about extending the attestation for lazy-participants case (if it’s not already part of the protocol).

> `Moh:` OK, thanks for the clarification 😁 . Of course we can have this feature to forward the proof of participation of successor committees. In the first version of LogosBFT we had this feature as a sliding window. One could choose the size of the window (in terms of tree levels) for which a node should forward the proof of participation. In the most recent version the size of sliding window is 0. And it is 1 for the root committee. It means root committee members have to forward the proof of participation of their child committee members. Since I was able to prove protocol correctness without forwarding the proofs so we avoid it. But it can be part of the protocol without any significant changes in the protocol

> If the proof scheme is efficient ( as the results you presented) in practice and the cost of creating and verifying proofs is not significant then actually adding proofs can be good. But not required.

### Also, how do you reward online validators / punish offline ones if you can't prove at the block level that someone attested or not? - Tanguy
> `Moh:` This is very tricky and so far no one has done it right (to my knowledge). Current reward mechanism for attestation, favours fast nodes.This means if malicious nodes in the network are fast, they can increase their stake in the network faster than the honest nodes and eventually take control of the network. Or in the case of Ethereum a Byzantine leader can include signature of malicious nodes more frequently in the proof of attestation, hence malicious nodes will be rewarded more frequently. Also let me add that I don't have definite answer to your question currently, but I think by revising the protocol assumptions, incentive mechanism and using a game theoretical approach this problem can be resolved.

> An honest node should wait for a specific number of children votes (to make sure everyone is voting on the same proposal) before voting but does not need to provide any cryptographic proof. Though we build a threshold signature from root committee members and it’s children but not from the whole tree. As long as enough number of nodes follow the the protocol we should be fine. I am working on protocol proofs. Also I think bugs should be discovered during development and testing phase. Changing protocol to detect potential bug might not be a good practice.
