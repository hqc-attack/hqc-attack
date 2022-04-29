# HQC Attack

Requires docker or podman to run.

## Usage

Results are stored in the `results` directory (created when the respective make targets are run).

### Perform the attack once:

```
make attack
```

### Collect timing information:

```
make collect-timings # takes ~1 hour on a Ryzen 9 5900X (single-threaded)
```

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