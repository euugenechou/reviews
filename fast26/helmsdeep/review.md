# HelmsDeep

## Paper Summary

Self-securing storage (S3) systems defend against ransomware by periodically
creating versioned backups of data using a versioning system (VS). Malicious
data updates are prevented using _timelocks_, which ensure immutability of data
for specified durations. Existing S3 systems incorporate the VS into their TCB,
which are susceptible to modern exploits, and prohibit users from modifying
versioning policies without modifying the TCB. This paper presents HelmsDeep
(HD) to address these issues. HD's key contributions are its decoupling of the
VS from the TCB, and its solutions to the challenges that arise in realizing the
decoupled design. Namely, an untrusted VS that uses timelocks for both data and
metadata, a formally verified HD microcontroller, and a way to disambiguate
legitimate vs. spoofed writes during recovery even with an untrusted VS.
Evaluation of the HD implementation demonstrate that HD has negligible
performance and storage overhead despite its minimal TCB and increased
user-defined policy flexibility.

### Strengths

HD is well-motivated and its contributions are well-articulated. Its design is
well-through-out, and include a number of optimizations for performance (e.g.,
sequential layout, incremental checkpoints, secure host-side HD metadata
caching, delayed defragmentation). The authors do an excellent job establishing
HD's integrity and security guarantees; a proof sketch for version integrity
guarantee in S4.2, and the formal verification of the HD controller in Dafny.
The authors also include examples of HD uses beyond ransomware.

### Weaknesses

There isn't much detail on HD's fault tolerance, which makes it harder to reason
about HD's security guarantees in the presence of failures. The evaluation is a
bit light, and could use further experimentation to explain certain aspects of
the results.

## Comments for authors

Thank you for your submission to FAST'26! I really enjoyed reading your paper
and enjoy the problem that it tackles. I find the design of HD to be novel,
elegant, and effective for its security goals---it also includes several systems
optimizations for its performance goals. I, in particular, found the use of the
**inc** command to not only serve as a mechanism for users to dynamically change
timelock duration, but to also serve as the mechanism for incremental
checkpointing to be quite elegant.

My main concerns with the work are the lack of detail on HD's fault tolerance
and the comprehensiveness of the evaluation. For the former: there are
discussions of crash/power failure in Sections 4.3 and 5.2.5, but I was hoping
for more clarity on what HD guarantees in the event of crash between persisting
metadata and data. It seems to me that HD needs a persisted timelock for before
data is persisted to ensure an invariant that all persisted data is timelocked,
but it's not super clear to me how it all shakes out with incremental
checkpointing with timelock increments. Moreover, fleshing out HD's fault
tolerance should also help demonstrate its security in the presence of failures.

For the latter: the evaluation lists six evaluated configurations (baselines),
but none of the baselines appear to be prior state-of-the-art S3 systems. While
it's understood that HD innovates on prior work by defending against recent
exploits like the trimming attack, I'd appreciate getting a better sense of HD's
performance and the impact of the tradeoffs it makes compared to prior work for
security and policy flexibility. I'd also appreciate more microbenchmarks or
performance drill-downs to explain results like in Section 7.3.1, which are
claimed to be a direct result of HD's design decisions.

I think that addressing these concerns, especially by making the evaluation more
thorough and comprehensive, will significantly bolster the paper---it is
interesting, impactful, and important work which displays a lot of promise.

### Detailed comments

- Page 2 (Introduction):

  > We design and implement a secure backup system based on timelocks, without trusting the VS. It is the first to timelock version metadata and never require overwrites.

  It feels to me that timelocking version metadata is the fundamental part to
  achieving a secure backup system with an untrusted VS, and not so much the
  "never require overwrites".

- Page 4 (HelmsDeep Interface): Is it possible for an adversary to maliciously
  use the **inc** command to prevent storage space from being reused? This seems
  like an attack vector that is out-of-scope for this work and may be orthogonal
  to the goals of the work.

- Page 7 (Implementation): I think it'd be good to demonstrate how HD's
  decoupling of the VS from the TCB allows for flexibility in defining versioning
  policy. This might not go in the implementation specifically, and might be a bit
  out-of-scope for this work, but it would be quite powerful if there were a
  framework (perhaps a DSL?) to help users define HD versioning policies on a
  per-use-case basis.

- Page 9 (Evaluation): I think that the security analysis experiment in Section
  7.2 is incredibly important to have. In particular, HD successfully recovering
  the state of the filesystem on every one of the randomware samples is a strong
  testament for its security, and should be called out as one of the goals of the
  evaluation: verifying that HD achieves its security goals. And, as you've
  already done, I would keep the recovery throughput separate from this analysis.

- Page 11 (I/O Overhead): It's implied that the layout of HD metadata/data is
  what amortizes the cost of the 2x I/O overhead -- I think this claim can be
  significantly strengthened with an experimental drill-down. I'd also
  appreciate getting a sense of the overhead of the read and write paths,
  individually.

### Nits

- Page 1 (Introduction): "begnign" -- typo

- Page 10 (Performance Analysis): This section references Figures 6a, 6b, and
  6c, which are currently labeled 7a, 7b, and 7c.

- Page 11 (Security Analysis): missing period at the end of the section
