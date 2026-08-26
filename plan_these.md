# Thesis Plan — From Transaction Locality to Scalable Design: Workload-Driven Sharding and Data Availability Sampling for Blockchains"

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
   - State of the art on blockchains
2. **The scalability problem**
   - Throughput, latency, and full-node cost bottlenecks
   - On-chain scaling: sharding (network, transaction, state sharding)
   - Off-chain / Layer 2 scaling: rollups (optimistic, zk), state channels, sidechains
   - State of the art existing solutions
3. **Related work synthesis**
   - Open challenges motivating this thesis

## Part II — Workload Analysis
*"Exploring Locality in Ethereum Transactions"*
https://ieeexplore.ieee.org/document/10317054
Need to update result with actual data

1. **Motivation**
   - Blockchain designs (sharding, light nodes, parallel execution) implicitly assume locality properties in transaction workloads
   - Need to know the nature of transactions to efficiently design sharding and L2 solutions
   - Objective is to predict future workload based on past activity
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

1. **Motivation**
   - Parts III (SOAPS) and IV (PANDAS) address the scalability trilemma from two different angles: on-chain state partitioning vs. off-chain L2 data-availability networking, but both target the same underlying tension: how to let a blockchain scale (throughput, data volume) without concentrating trust or excluding low-resource participants
   - Neither solution is presented in the literature as an alternative to the other in practice (Ethereum's roadmap pursues sharded execution/state ideas *and* Danksharding/DAS concurrently).
   - The thesis is positioned to make explicit what is usually left implicit, where the two approaches are substitutes, where they are complements, and what they cost relative to each other
   - Goal: give a reader (protocol designer, researcher) a framework for choosing/combining state-sharding and L2-DAS techniques depending on their bottleneck (execution/state contention vs. data-availability/bandwidth)
2. **Comparison framework**
   - **Layer addressed**: SOAPS operates on-chain, partitioning *execution and state* across shard committees; PANDAS operates off-chain/networking, distributing custody and verification of *L2 data* (blobs) that never needs re-execution, only availability
   - **Unit being placed/assigned**: SOAPS assigns *accounts* to shards (a semantically meaningful, workload-dependent decision, since it changes who conflicts with whom); PANDAS assigns *erasure-coded cells* to nodes (a content-agnostic, load-balancing decision, since any cell is interchangeable once erasure-coded)
   - **Adaptivity mechanism**: SOAPS periodically recomputes Partial Explicit Placement Rules (PEPR) for the top-$K$ accounts and amortizes migration cost across reassessment periods; PANDAS recomputes its deterministic assignment every epoch (~6.4 min) via pseudo-random sortition, and within a single 4 s window adapts fetching aggressiveness/redundancy round by round, a slower, workload-driven adaptivity loop vs. a faster, fixed-cadence, network-condition-driven one
   - **Consensus integration strategy**: both explicitly avoid heavyweight, on-every-update BFT consensus. SOAPS delegates PEPR computation to an elected committee member with redundant recomputation only on witness disagreement; PANDAS adds no new consensus step at all, folding DAS into the existing attestation precondition (tight fork-choice rule). SOAPS is the more consensus-invasive of the two, PANDAS the more conservative
   - **Adversarial model**: SOAPS defends against *selfish/centralized placement bias* (an actor skewing shard assignment to favor its own accounts); PANDAS defends against *Sybil/eclipse attacks* on the peer-to-peer fetch layer and flags an open *free-rider* incentive gap (no reward for answering sampling/consolidation requests). The threat surfaces are largely disjoint (state-assignment integrity vs. networking-layer availability)
   - **Evaluation methodology**: SOAPS validates via mainnet transaction-trace replay against a fixed hash-based baseline; PANDAS validates via a real prototype deployment (1,000 nodes) cross-validated with a simulator (up to 20,000 nodes) against GossipSub/Kademlia-DHT baselines — trace replay measures algorithmic effectiveness on real workloads, while prototype+simulation measures systems-level feasibility under real timing/network constraints; together they exemplify the two main evaluation paradigms used in the thesis
   - **Primary metric optimized**: cross-shard transaction ratio and effective throughput (SOAPS) vs. time-to-sampling relative to a hard 4 s deadline and per-node bandwidth (PANDAS) — one targets *coordination cost reduction*, the other targets *meeting a real-time constraint under decentralization limits*
3. **Findings**
   - Both designs reject "assign everything explicitly" in favor of a *bounded, principled subset* strategy: SOAPS explicitly places only the $K$ highest-impact accounts (falling back to hash-based placement otherwise); PANDAS gives every node a full, deterministic but content-agnostic assignment, with adaptivity pushed into the fetching protocol rather than the assignment itself — the shared lesson is that full, arbitrary flexibility (explicit rules for all ~400M accounts; unconstrained ad-hoc networking) is unworkable at Ethereum scale, and both instead constrain the *design space* of the assignment function to make it cheap to compute, verify, and keep consistent across nodes
   - Both rely on **determinism as a security property**: PEPR outputs are verifiable by any node re-running the same deterministic oracle (METIS/Louvain) on the same input; PANDAS's `σ` is verifiable by any node re-deriving it from the public epoch seed. In both cases determinism removes the need for a trusted third party to announce assignments and limits an adversary's ability to bias or predict them
   - Where they differ most: SOAPS's benefit compounds with *skewed, non-local workloads* (the more accounts violate locality, the more PEPR helps, directly the population identified in Part II), whereas PANDAS's benefit is largely workload-independent (blob cells carry no semantic locality; its bottleneck is network heterogeneity and node responsiveness, not transaction patterns) — meaning Part II's locality findings inform Part III far more directly than Part IV
   - Complementary rather than substitutable in practice: adopting state sharding (Part III) does not reduce the need for scaling L2 data availability (Part IV), and a sharded L1 still needs L2s to publish blobs, and a rollup-heavy L1 still benefits from parallel execution; the two mechanisms could in principle be deployed together, with cross-shard coordination cost (SOAPS) and blob dissemination cost (PANDAS) as separate, independently tunable overheads
   - Both surface an unresolved tension between decentralization and optimization: SOAPS must bound $K$ and use economic disincentives (rather than full BFT) to keep placement computation cheap and non-centralizing; PANDAS deliberately keeps the network open (no validator-only restriction) at the cost of unresolved free-rider incentives. In both cases, the "cheap, best-effort correctness with fallback/verification" pattern substitutes for expensive guaranteed correctness
4. **Discussion: towards a combined view**
   - Sketch of a hypothetical combined system: a sharded execution layer (SOAPS-style state placement) where each shard's data availability for its L2/blob-equivalent state is itself sampled (PANDAS-style), i.e., DAS ideas generalize beyond monolithic-L1 blobs to sharded state commitments
   - Revisit Part II through both lenses: hot/adversarial accounts (locality study) matter for *where* to place state (Part III) but not for *how* to disseminate data-availability cells (Part IV); this asymmetry itself is a finding worth stating explicitly — not every scaling technique benefits equally from workload-awareness
   - Cost comparison caveat: the two are evaluated on different metrics (throughput/cross-shard ratio vs. latency-to-deadline/bandwidth) and cannot be reduced to a single number. The thesis should be explicit that "which is better" is not a well-posed question. The honest comparison is over *what bottleneck each removes*
5. **Implications for the thesis**
   - Demonstrates the thesis's overall thesis statement: scaling blockchains requires workload- and network-aware, *bounded-adaptive* mechanisms rather than either fully static (naive hash-based sharding, broadcast-to-all DAS) or fully dynamic/unconstrained (per-account placement rules, ungoverned P2P fetching) designs
   - Provides the basis for the General Conclusion's "answers to research questions": both artifacts validate that empirically grounded (Part II), bounded, deterministic, decentralization-preserving mechanisms can achieve large practical gains (up to 4× throughput; sub-4s DAS at Ethereum scale) without sacrificing the core trust assumptions of a permissionless blockchain**



## General Conclusion
- Summary of contributions per part
- Answers to the research questions
- Limitations
- Future work / perspectives

## Back matter (required)
- Bibliography (biblatex/Biber, `biblio.bib`)
- List of personal publications / bibliographic notes (using `\myfullcite`, see `commands.tex`/`thesis.tex`)
- Appendices (proofs, additional experimental results, dataset/reproducibility details)
   