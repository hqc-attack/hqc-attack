# HQC Attack

Requires docker or podman to run.

## Usage

Results are stored in the `results` directory (created when the respective make targets are run).

### Perform the attack once:

```
make attack
```

This will run the attack, and print for each attempted recovery of a block something like:

```
Took 158 bit flips to obtain decoding failure
Iteration 73 block 38
Skipped count: 416
Decryption oracle calls: 843673
```

The iteration shows the number of recovery attempts that have been performed so far on this block of the ciphertext.
For each bit the majority vote is cast among up to 5 recovery attempts.
If a majority for all bits in a block is reached earlier, e.g. because the first 3 attempts yielded the same result, it will skip the remaining iterations for that block. This is reflected in the `skipped count` in the output, which shows the number of blocks for which the recovery was skipped during an iteration.
Finally, the total number of timing decryption oracle calls is counted - the oracle reveals the idealized timing of a ciphertext, under a specific secret key.

Once either every bit has formed a majority or the total weight of the recovered key among the recovered bits has reached the expected weight $\omega = 66$, the attack terminates.
Finally it prints the recovered key, the original secret key and the XOR of the two (the differences) in hex with zero bytes shown as `--` to make the output more readable.

The last few lines are:

```
Success? 1
Oracle calls 888105
Timing mismatches: 0
Final classification: 0 bits wrong
```

Where success is 1 iff the attack succeeded.
Timing mismatches occur when the timing oracle predicted did not predict that the ciphertext should fail to decode, but the ciphertext did fail to decode (it decoded to a different message than the original one).
The last line shows how many bits were incorrectly recovered in the secret key.


### Collect timing information:

```
make collect-timings # takes ~1 hour on a Ryzen 9 5900X (single-threaded)
```

You can adjust the number of iterations to use for each 100 ciphertexts [here](./hqc/nist-release-2021-06-06/Optimized_Implementation/hqc-128/src/collect_timings.c):

```
#define ITERS 100000
```

and [here](./hqc/nist-release-2021-06-06/Optimized_Implementation/hqc-128/src/collect_timings_messages.c):

```
#define N 1000000
```

try 10 and 1000 for quick testing.


To get the best results:
- set process niceness to -20 (using `nice -n-20 CMD`)
- pin the process to a single core, and all other processes to a different one (using `taskset -p 0x1`, etc)
- disable simultaneous multi-threading
- disable dynamic frequency scaling
- do not use the system for anything else 

### Run the attack 1000 times and gather statistics:

```
make attackstats # takes ~0.75 hours on a Ryzen 9 5900X (multi-threaded)
```

You can edit the number of times the attack is run in the [`Makefile`](./Makefile) under the `attackstats` target:


```sh
docker run -it --rm -v "$(realpath results/attackstats):/collect_attack_stats/results" --entrypoint=../run.sh hqc-attackstats 1000 # edit this 1000
```

### Create figures:

```
make figures # Must make collect-timings and attackstats before making figures!
```

## Structure

- `analyses`
    Julia scripts for analyzing the data from gathered timing information, or output of the attack
- `collect_attack_stats`
    Harness/Wrapper for running the attack many times in parallel and gathering the attack results
- `hqc`
    HQC implementation and the attack and timing information gathering implementations
    - [optimized attack](hqc/nist-release-2021-06-06/Optimized_Implementation/hqc-128/src/attack_hybrid.c)
    - [simpler version of the attack](hqc/nist-release-2021-06-06/Optimized_Implementation/hqc-128/src/attack_original.c)
    - implementations for gathering timing information
        - few ciphertexts, many measurements each: [collect_timings.c](hqc/nist-release-2021-06-06/Optimized_Implementation/hqc-128/src/collect_timings.c)
        - many ciphertexts, 1  measurement each: [collect_timings_messages.c](hqc/nist-release-2021-06-06/Optimized_Implementation/hqc-128/src/collect_timings_messages.c)
    - countermeasures:
        - [original (instrumented to output additional information about the algorithm)](hqc/nist-release-2021-06-06/Optimized_Implementation/hqc-128/src/vector_original.c)
        - [first (incomplete) countermeasure](hqc/nist-release-2021-06-06/Optimized_Implementation/hqc-128/src/vector_countermeasure_1.c)
        - [second (incomplete) countermeasure](hqc/nist-release-2021-06-06/Optimized_Implementation/hqc-128/src/vector_countermeasure_2.c)
        - [third (complete) countermeasure](hqc/nist-release-2021-06-06/Optimized_Implementation/hqc-128/src/vector_countermeasure_3.c)