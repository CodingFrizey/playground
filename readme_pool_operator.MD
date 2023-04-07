# POOL OPERATOR INFO #

General:
Almost all pools are using the cryptonote pool as a basis and have updated pool.js. It is basically using Stratum protocol for the communication between miner and pool. As you know is DynexSolve a Proof-of-Useful-Work (PoUW) algorithm. As such, there are a few important considerations:

1) When a miner submits a share, additional information (other than found nonce) is submitted from the miner to the pool:

std::string COMMAND_SUBMIT = "{\"jsonrpc\":\"2.0\",\"id\":" + std::to_string(++id) + ",\"method\":\"submit\",\"params\":{\"id\":\""+H_STRATUM_ID+"\",\"job_id\":\""+H_STRATUM_JOB_ID+"\",\"nonce\":\""+found_nonce+"\",\"result\":\""+found_hash+"\",\"algo\":\"dynexsolve224final\",\"mallob\":\""+H_MALLOB_NETWORK_ID+"\",\"POUW_DATA\":\""+POUW_DATA+"\"}}\n";
        
-> params->id: pool job id of the solution
-> params->job_id: pool job id of the solution
-> params->nonce: found nonce
-> params->result: found hash
-> params->algo: dynexsolve224final which identifies the latest accepted version of DynexSolve 
-> params->mallob: the mallob id of the miner (to reference it with our mallob system)
-> params->pouw_data: proof of work data from the miner

This is required, because PoUW requires a tight entanglement between the "Useful work" (GPU POUW) and the traditional hashing. You can see in the code above that, in addition to the CN nonce, the mallob ID (unique miner identifier in the job scheduling system "Mallob"), as well as information about the Job, and the POUW_BLOB including a POW_HASH and POUW_DIFF are being submitted. the pool has to use this data subsequently for verification

2) The performance of .js is not strong enough to accurately verify both POUW and the entangled CN hash. We therefore have created a RPC service "poolservice", which the pools call for verification of the submitted share. We have a public endpoint for that, larger pools usually have their own poolservice running on a local Ubuntu 22.04 machine. A single instance of the poolservice can handle up to 50k RPS. We have pools which also load balance multiple pool service instances.

Rather than having the pool implement CN hashing or POUW checking, a single RPC call to the poolservice is required. You can find an example of the correct implementation in file "pool2.js" (which also includes utils.js)

It returns "200" if the share passes validation (= CN hash is ok, POUW work is OK, entanglement is OK). 
If the return code is 200, the share is valid and can be accepted. If the share represents a found block, the block must be submitted to the node.

Notes:
a) EVERY share has to be validated with poolservice, sharetrust of the pool implementation is exploitable by malicious actors. 
b) you will need a valid "pool_uuid" to be able to submit blocks. for testing, you can use any random uuid, but you won't be able to submit a valid block. We issue pool_uuid after testing of your pool implementation and when your pool is listed on miningpoolstats (MPS).


