# 【案例】

* （第一句介绍系统的架构）**We consider a typical three-tier architecture (Figure xx), which consists of** a backend database, a frontend visualization interface, and a middleware layer in between.
* For Lambada（提出的系统名称）, **our design goal is thus to** use solely existing serverless components. **Figure 2 depicts its high-level architecture**.

# 【案例】

段1：

* （系统架构）Figure x shows an overview of our system architecture, which consists of a high-performance key-value store (KVS) Anna and a function execution layer Cloudburst.
* 
* 
* （介绍选择具体技术的原因）We chose Anna as the storage engine as it offers low latencies and flexible autoscaling, a good fit for serverless. Anna also supports custom conflict resolution policies to resolve concurrent updates. As we discuss in Section 2.2, this provides a necessary foundation to support causal consistency.