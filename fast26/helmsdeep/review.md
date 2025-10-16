# HelmsDeep

## Paper Summary

Self-securing storage (S3) systems defend against ransomware by periodically
creating versioned backups of data using a versioning system (VS). Malicious
data updates are prevented using _timelocks_, which ensure immutability of data
for specified durations. Existing S3 systems incorporate the VS into their TCB,
which are susceptible to modern exploits, and prohibit users from modifying
system policies without modifying the TCB. This paper presents HelmsDeep (HD) to
address these issues. HD's key contributions are its decoupling of the VS from
the TCB, and its solutions to the challenges that arise in realizing the
decoupled design. Namely, an untrusted VS that uses timelocks for both data and
metadata, a formally verified HD microcontroller, and a way to disambiguate
legitimate vs. spoofed writes during recovery even with an untrusted VS.
Evaluation of the HD implementation demonstrate that HD has negligible
performance and storage overhead despite its minimal TCB and increased
user-defined policy flexibility.

## Strengths

HD is well-motivated and its contributions are well-articulated. Its design is
also well-through-out, and include a number of optimizations for performance
(e.g., sequential layout, incremental checkpoints, secure host-side HD metadata
caching, delayed defragmentation). The authors also do an excellent job
establishing HD's integrity and security guarantees; a proof sketch for version
integrity guarantee in S4.2, and the formal verification of the HD controller in
Dafny. The authors also include examples of HD uses beyond ransomware.

## Weaknesses

There isn't much detail on HD's crash consistency guarantees. The evaluation
seems to be missing a proper baseline, and could use further experimentation to
explain certain aspects of the results.

## Comments for authors

Thank you for your submission to FAST'26! I really enjoyed reading your paper
and enjoy the problem that it tackles. The design of HD is elegant and effective
for its security goals, and includes several systems optimizations for its
performance goals. In particular, I found the use of the _inc_ command to not
only serve as a mechanism for users to dynamically change timelock duration, but
to also serve as the mechanism for incremental checkpointing to be quite
elegant.

My main concerns with the work are the lack of detail on HD's crash consistency
guarantees and some aspects of the evaluation. For the former: there were
discussions of crash/power failure in S4.3 and S5.2.5, but I was hoping for more
clarity on what HD guarantees in the event of crash between persisting data and
metadata. It seems to me that HD needs a persisted timelock for before data is
persisted to ensure an invariant that all persisted data is timelocked, but it's
not super clear to me how it all shakes out with incremental checkpointing with
timelock increments.

For the latter: the evaluation lists six configurations that are evaluated, but
the results only depict five across the various traces and workloads. Moreover,
none of the baselines appear to be prior state-of-the-art S3 systems. While it's
understood that HD innovates on prior work by defending against recent exploits
like the trimming attack, I'd appreciate getting a better sense of HD's
performance and the impact of the tradeoffs it makes compared to prior work for
security and user policy flexibility. I'd also appreciate some microbenchmarks
or performance drill-downs to explain results like in S7.3.1, which are claimed
to be a direct result of HD's design decisions.

I think that addressing these concerns, especially by making the evaluation more
thorough, will significantly bolster the paper.

## Detailed comments

- Page 2 (1 Introduction):

  > We design and implement a secure backup system based on timelocks, without trusting the VS. It is the first to timelock version metadata and never require overwrites.

  It feels to me that timelocking version metadata is the fundamental part to
  achieving a secure backup system with an untrusted VS, and not so much the
  "never require overwrites".

- Page 8 (5.3 Versioning System)

  > The nodes in the metadata linked list are chronologically ordered, except for the last unwritten pre-allocated metadata block.

  Is it possible to come across a "broken" linked list on crash? I assume that
  the append-only metadata log helps with this.

## Nits

- Page 1 (1 Introduction): "begnign" -- typo
- Page 11 (7.2 Security Analysis): missing period at the end of the section
