# Thesis Plan — Sharded Blockchains: Scaling via Sharding and L2 Optimization using DAS

## Front matter (required)
- Title page
- Copyright / declaration of authorship
- Abstract (English)
- Acknowledgments
- Table of contents
- List of abbreviations / glossary (DAS, L2, DA, EVM, MEV, etc.)

## Introduction
- Context: blockchain adoption and the scalability trilemma (decentralization / security / scalability)
- Problem statement: why sharding and L2s need workload-aware, data-availability-aware design
- Research questions and contributions
- Thesis outline

## Part I — Background and State of the Art
1. **Blockchain fundamentals**
   - Distributed ledgers
   - consensus (PoW, PoS, BFT variants)
   - Transaction and state model (UTXO vs. account-based, e.g., Ethereum)
   - Smart contracts and execution environments (EVM and alternatives)

2. **The scalability problem**
   - Throughput, latency, and full-node cost bottlenecks
   - On-chain scaling: sharding (network, transaction, state sharding)
   - Off-chain / Layer 2 scaling: rollups (optimistic, zk), state channels, sidechains

3. **Related work synthesis**
   - Comparative taxonomy of sharding and L2+DAS designs
   - Open challenges motivating this thesis

## Part II — Workload Analysis
*"Exploring Locality in Ethereum Transactions"*
https://ieeexplore.ieee.org/document/10317054
Need to update result with actual data

1. **Motivation**
   - Blockchain designs (sharding, light nodes, parallel execution) implicitly assume locality properties in transaction workloads

2. **Methodology**
   - Dataset: full Ethereum transaction history (largest available transactional workload)
   - Definitions: temporal locality (reuse of accounts/state over time) and spatial locality / disjoint access parallelism (non-conflicting concurrent transactions)

3. **Findings**
   - Transactions generally exhibit good temporal and spatial locality
   - A minority of accounts are responsible for most caching- and parallelism-unfriendly behavior
   - Interplay between the two locality properties

4. **Implications for thesis design choices**
   - Consequences for state-sharding partitioning strategies (account/contract assignment to shards)
   - Consequences for parallel transaction execution in L2 sequencers/rollups
   - Identification of "hot" / adversarial accounts requiring special handling
   - This part grounds Parts III and IV in empirically observed workload properties rather than idealized assumptions

## Part III — Optimized State Placement for Sharded Blockchains
*"SOAPS: Securing Optimized State Placement in Account-based Sharded Blockchains"*
`Paper/Smart-Sharding-claud/paper/main.tex`

1. **Motivation**
   - Sharding partitions blockchain state across independent committees for parallel processing, but cross-shard transactions (accessing state in different shards) require costly coordination (e.g., two-phase commit)
   - Standard hash-based placement performs poorly: with 64 shards, 94.6% of recent Ethereum transactions would be cross-shard

2. **Problem with prior explicit-placement work**
   - Assigning explicit placement rules for *all* accounts is impractical at scale (~400M Ethereum accounts → ~400 MB of metadata every node must maintain/synchronize)
   - Uncontrolled/centralized placement decisions raise security and fairness issues (selfish optimization: an actor biases placement to favor its own accounts/transactions at others' expense)
3. **Design: SOAPS**
   - **Bounded placement rules**: Partial Explicit Placement Rules (PEPR) — explicit shard assignment only for the $K$ accounts with the greatest impact on cross-shard traffic; hash-based placement retained for the rest

   - **Consensus integration**: PEPR computation delegated to an elected beacon-chain committee member; redundant computation/verification only triggered when elected witnesses disagree (economically disincentivized), avoiding systematic redundant computation or full BFT consensus on every PEPR update

   - **Modularity**: pluggable, deterministic optimization oracles (community-detection algorithms), instantiated with METIS and Louvain-based oracles; PEPR periodically reassessed to adapt to evolving workloads while amortizing state-migration cost

4. **Evaluation**
   - Ethereum mainnet transaction traces
   - Up to 40% reduction in cross-shard transactions and up to 4× effective throughput increase, with modest synchronization overhead

5. **Implications for the thesis**
   - Directly builds on Part II: PEPR targets exactly the "hot"/non-local accounts identified as responsible for parallelism-unfriendly behavior in the locality study
   - Provides the on-chain (state-sharding) counterpart to Part IV's off-chain (L2/DAS networking) scaling mechanism, both grounded in the same empirical workload evidence

## Part IV — L2 Optimization via Data Availability Sampling
*"PANDAS: Peer-to-peer, Adaptive Networking for Data Availability Sampling within Ethereum Consensus Timebounds"*
https://arxiv.org/pdf/2507.00824

1. **Motivation**
   - Danksharding aims to scale layer-2 data (blobs) by having nodes hold and verify only a subset of data via DAS, instead of broadcasting everything to everyone
   - Integrating DAS into Ethereum consensus is hard: blob data must be disseminated and sampled within the first 4 seconds of a 12-second slot, with no existing solution meeting this bound

2. **Background: DAS mechanics**
   - The data availability problem in rollup-based systems
   - Erasure coding and DAS constructions (fraud/validity proofs, KZG commitments, sampling protocols)
   - EIP-4844 (Proto-Danksharding) and blobspace; limits of broadcasting blobs to all nodes
   - Blob construction: erasure coding (2D Reed-Solomon) into a 512×512 extended cell matrix with KZG proofs; random cell sampling for probabilistic availability guarantees
   - Proposer-Builder Separation (PBS) as the context in which blob data is seeded

3. **Design: PANDAS**
   - Deterministic assignment of blob data (cells) to nodes
   - Three phases: seeding (by builders), consolidation, and sampling
   - Direct, unreliable one-hop UDP communication with an adaptive fetching protocol (redundancy/time-bound arbitration) tolerant to node unavailability, faults, and inconsistent network views

4. **Evaluation**
   - Prototype on libp2p / Geth, 1,000-node cluster with emulated WAN latencies, cross-validated against a simulator scaled to 20,000 peers
   - Meets the 4-second sampling deadline at moderate and large scale, including under up to 50% unresponsive/misbehaving nodes, with lower overhead than GossipSub- or Kademlia-DHT-based baselines

5. **Implications for the thesis**
   - Shows DAS can be integrated into Ethereum consensus without protocol changes to consensus or node discovery, directly supporting the Danksharding scaling roadmap for L2 data
   - Connects back to Part II: node-level and account-level access patterns/hot spots inform how sampling and fetching load could be further balanced across the network

## Part V — Compare the two solution


## General Conclusion
- Summary of contributions per part
- Answers to the research questions
- Limitations
- Future work / perspectives

## Back matter (required)
- Bibliography (biblatex/Biber, `biblio.bib`)
- List of personal publications / bibliographic notes (using `\myfullcite`, see `commands.tex`/`thesis.tex`)
- Appendices (proofs, additional experimental results, dataset/reproducibility details)
   