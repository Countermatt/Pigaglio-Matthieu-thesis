# Thesis Plan — Toward Scalable Blockchains: Sharding State and Sampling Data Availability

## Front matter
- Title page
- Abstract (English)
- Acknowledgments
- Table of contents
- List of abbreviations / glossary (DAS, L2, DA, EVM, MEV, etc.)

## Introduction
- Context: L1 and L2 must scale alike
   - Blockchain adoption puts pressure on both the base layer (L1: execution and state) and the layer-2s built on it (L2: rollups publishing their data to L1)
   - One cannot scale without the other: L2s offload execution but still depend on L1 for settlement and data publication; a faster L1 alone does not absorb the data volume produced by L2s
   - The scalability trilemma (decentralization / security / scalability) only mentioned briefly as background framing, not as the driver of the thesis
- The general idea: sharding, i.e., avoid every node being in charge of every transaction / every piece of data
   - **Computation (state) sharding** on L1: partition execution and state across shards
   - **Data sharding** for L2 data: nodes custody and verify only a subset of the published data (Data Availability Sampling)
- Problem statement: contribute to blockchain scaling across L1 and L2 via sharding techniques, both computation sharding and data sharding, which take different forms in L1 and L2
- Guiding principle: **verifiable yet flexible designs**
   - Both designs rely on deterministic, verifiable operations (any node can recompute / re-derive the assignment), which removes the need for a trusted party and limits adversarial bias
   - But they keep enough flexibility (bounded explicit placement; adaptive fetching) to operate in large-scale, dynamic settings
   - Grounded in practical deployment considerations: real Ethereum workloads, Ethereum consensus timing constraints, compatibility with existing protocols
- Research questions and contributions
   - RQ1: Can we derive solid assumptions on the near-future workload from historical transaction data? (Part II)
   - RQ2: How to place state in an account-based sharded blockchain to reduce cross-shard transactions, without centralizing or exposing placement to selfish optimization? (Part III)
   - RQ3: How to disseminate and sample L2 data within Ethereum consensus timebounds, in a fully decentralized, peer-to-peer way? (Part IV)
   - List of publications mapped to chapters
- Thesis outline

## Part I — Background and State of the Art

1. **Blockchain fundamentals** (only what is needed to understand the rest of the thesis)
   - Distributed ledgers
   - Consensus (PoW, PoS, BFT variants)
   - Ethereum Proof-of-Stake in more detail: validators, slots/epochs, committees and attestations, fork-choice, rewards and safeguards (e.g., slashing)
      - Needed as a basis for SOAPS's consensus integration (Part III) and PANDAS's timing constraints (Part IV)
   - Transaction and state model (UTXO vs. account-based, e.g., Ethereum)
   - Smart contracts and execution environments (EVM and alternatives)
2. **The scalability problem**
   - Throughput, latency, and full-node cost bottlenecks
   - Approaches relying on workload properties: parallel transaction execution, light nodes / stateless clients
      - Needed as a basis for the motivation of Part II
   - On-chain scaling: sharding (network, transaction, state sharding)
      - Existing L1 sharding solutions and their **limitations** (reuse related work of the locality and SOAPS papers)
   - Off-chain / Layer 2 scaling: rollups (optimistic, zk), state channels, sidechains
      - Existing L2 / data-availability solutions and their **limitations** (reuse related work of the PANDAS paper)
3. **Synthesis and motivation**
   - Open challenges motivating this thesis, structured around the limitations above
   - Common requirements for both L1 and L2 sharding: verifiable yet flexible operations, decentralization preserved, deployable in practice
   - Brief positioning of the two contributions: state sharding (Part III) removes execution/state contention, DAS (Part IV) removes the data-availability/bandwidth bottleneck; complementary, not substitutes

## Part II — Workload Analysis
*"Exploring Locality in Ethereum Transactions"*
https://ieeexplore.ieee.org/document/10317054
TODO: update results with recent data

1. **Motivation**
   - Blockchain designs (sharding, light nodes, parallel execution — introduced in Part I) implicitly assume locality properties in transaction workloads
   - Need to know the nature of transactions to efficiently design sharding solutions
   - Research question: can we derive solid assumptions for the near future based on historical data?
2. **Methodology**
   - Dataset: full Ethereum transaction history (largest available transactional workload)
   - Definitions: temporal locality (reuse of accounts/state over time) and spatial locality / disjoint access parallelism (non-conflicting concurrent transactions)
   - Why this is non-trivial: size of the history, extracting the actual state accessed by each transaction (internal calls, storage accesses) rather than just sender/receiver, computing conflicts at scale
   - Implementation, as described in the paper (data extraction pipeline, tooling, processing)
3. **Findings**
   - Transactions generally exhibit good temporal and spatial locality
   - A minority of accounts are responsible for most caching- and parallelism-unfriendly behavior
   - Interplay between the two locality properties
4. **Implications for the rest of the thesis**
   - Consequences for state-sharding partitioning strategies (account/contract assignment to shards) → directly used in Part III
   - Identification of "hot" / non-local accounts requiring special handling → targeted by PEPR in Part III
   - Same dataset and evaluation setup reused in Part III
   - Broader implications (e.g., parallel execution in L2 sequencers/rollups) stated only as perspectives, not as something addressed in this thesis

## Part III — Optimized State Placement for Sharded Blockchains
*"SOAPS: Securing Optimized State Placement in Account-based Sharded Blockchains"*
`Paper/SOAPS/paper/main.tex`

1. **Motivation**
   - Sharding partitions blockchain state across independent committees for parallel processing, but cross-shard transactions (accessing state in different shards) require costly coordination (e.g., two-phase commit)
   - Standard hash-based placement performs poorly: with 64 shards, 94.6% of recent Ethereum transactions would be cross-shard (measured on the same dataset / evaluation setup as Part II)
2. **System model: a classical sharded blockchain**
   - Beacon chain coordinating shard chains: validator assignment to shards, committees, cross-links
   - Built on the Ethereum PoS concepts introduced in Part I
3. **Problem with prior explicit-placement work**
   - Assigning explicit placement rules for *all* accounts is impractical at scale (~400M Ethereum accounts → ~400 MB of metadata every node must maintain/synchronize)
   - Uncontrolled/centralized placement decisions raise security and fairness issues (selfish optimization: an actor biases placement to favor its own accounts/transactions at others' expense)
4. **Design: SOAPS**
   - **Bounded placement rules**: Partial Explicit Placement Rules (PEPR) — explicit shard assignment only for the $K$ accounts with the greatest impact on cross-shard traffic; hash-based placement retained for the rest
   - **Consensus integration**: PEPR computation delegated to an elected beacon-chain committee member; redundant computation/verification only triggered when elected witnesses disagree, avoiding systematic redundant computation or full BFT consensus on every PEPR update
      - Build directly on the Ethereum safeguards from Part I (slashing, rewards) to argue that misbehavior is economically disincentivized — more explicitly than in the paper
   - **Modularity**: pluggable, deterministic optimization oracles (community-detection algorithms), instantiated with METIS and Louvain-based oracles; PEPR periodically reassessed to adapt to evolving workloads while amortizing state-migration cost
5. **Evaluation**
   - Ethereum mainnet transaction traces — experimental setup partly shared with Part II (same dataset and processing pipeline)
   - Up to 40% reduction in cross-shard transactions and up to 4× effective throughput increase, with modest synchronization overhead
6. **Implications for the thesis**
   - Directly builds on Part II: PEPR targets exactly the "hot"/non-local accounts identified as responsible for parallelism-unfriendly behavior in the locality study
   - Computation-sharding contribution on L1; Part IV provides the data-sharding counterpart for L2 data

## Part IV — Scaling L2 Data via Data Availability Sampling
*"PANDAS: Peer-to-peer, Adaptive Networking for Data Availability Sampling within Ethereum Consensus Timebounds"*
https://arxiv.org/pdf/2507.00824

1. **Motivation**
   - How L1, L2 and data availability fit together, developed on a concrete example (e.g., Optimism): the rollup executes transactions off-chain, posts its data to L1, and anyone must be able to retrieve that data to reconstruct the state / challenge invalid state transitions
   - The data availability problem in rollup-based systems: L2 data volume grows, but broadcasting all data to all nodes does not scale
   - Ethereum's roadmap (Danksharding, mentioned but not bound to the term) aims to scale L2 data (blobs) by having nodes hold and verify only a subset of the data via DAS
   - Integrating DAS into Ethereum consensus is hard: blob data must be disseminated and sampled within the first 4 seconds of a 12-second slot, with no existing solution meeting this bound
2. **Background: DAS mechanics**
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
   - Shows DAS can be integrated into Ethereum consensus without protocol changes to consensus or node discovery
   - Data-sharding contribution for L2 data, complementary to the computation sharding of Part III
   - Unlike Part III, largely workload-independent (erasure-coded cells carry no semantic locality): its bottleneck is network heterogeneity and node responsiveness, not transaction patterns

## General Conclusion
1. **Summary of contributions per part**
2. **Answers to the research questions**
3. **Discussion: comparing the two approaches** (former Part V, condensed; parts of the comparison moved to the Introduction and the end of Part I)
   - **Layer and unit assigned**: SOAPS partitions *execution and state* by assigning *accounts* to shards (semantic, workload-dependent decision); PANDAS distributes custody of *L2 data* by assigning *erasure-coded cells* to nodes (content-agnostic, load-balancing decision)
   - **Adaptivity**: SOAPS — slow, workload-driven loop (periodic PEPR recomputation, migration cost amortized); PANDAS — epoch-level deterministic reassignment plus fast, network-condition-driven adaptation of fetching within the 4 s window
   - **Consensus integration**: both avoid heavyweight, on-every-update BFT consensus; SOAPS is the more consensus-invasive (elected computation + witnesses), PANDAS the more conservative (folds DAS into the existing attestation precondition)
   - **Adversarial model**: SOAPS — selfish/centralized placement bias; PANDAS — Sybil/eclipse on the fetch layer, open free-rider incentive gap; largely disjoint threat surfaces
   - **Evaluation methodology and metrics**: trace replay measuring algorithmic effectiveness (cross-shard ratio, throughput) vs. prototype + simulation measuring systems feasibility (time-to-sampling vs. deadline, bandwidth); "which is better" is not a well-posed question — the honest comparison is over *which bottleneck each removes*
   - **Shared lessons**:
      - Both reject "assign everything explicitly" in favor of a bounded, principled assignment function that is cheap to compute, verify, and keep consistent across nodes
      - Determinism as a security property (back to the guiding principle of the Introduction): PEPR verifiable by re-running the deterministic oracle; PANDAS's assignment re-derivable from the public epoch seed
      - "Cheap, best-effort correctness with fallback/verification" substitutes for expensive guaranteed correctness, at the cost of open decentralization/incentive questions
   - **Asymmetry w.r.t. workload awareness**: Part II's locality findings matter for *where* to place state (Part III) but not for *how* to disseminate DA cells (Part IV) — not every scaling technique benefits equally from workload awareness
4. **Limitations**
5. **Future work / perspectives**
   - Combined system: sharded execution layer (SOAPS-style placement) where each shard's data availability is itself sampled (PANDAS-style)
   - Parallel execution in L2 sequencers informed by the locality findings of Part II
   - Incentives for serving sampling/consolidation requests in PANDAS
   - Related work I contributed to: evaluation of Disc-NG, which follows the same principle — a tradeoff between pure determinism (DHT) and purely unstructured operations (random walks)

## Back matter (required)
- Bibliography (biblatex/Biber, `biblio.bib`)
- List of personal publications / bibliographic notes (using `\myfullcite`, see `commands.tex`/`thesis.tex`)
- Appendices (proofs, additional experimental results, dataset/reproducibility details)
