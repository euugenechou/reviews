# MINOS

## Paper Summary

Log-structured storage systems depend on garbage collection (GC) to reclaim
space occupied by invalid data. Notably, the GC process can cause write
amplification as a result of migrating valid data from victim segments. The
magnitude of these extra writes is measured by Write Amplification Factor (WAF),
which can be affected by the amount of over-provisioning (OP) space and, most
notably, by data separation policy. Unfortunately, state-of-the-art (SOTA)
policies are guided by "Oracles" that employ ad-hoc heuristics without formal
optimization objectives, and thus do not realize the potential of data
separation for reducing WAF. This paper addresses this issue with its two key
contributions: the OP-Minimized Oracle and MINOS. The OP-Minimized Oracle guides
according to a formal model that determines optimal separation thresholds while
minimizing OP space for a theoretical WAF of 1. MINOS builds on the theory
established by the OP-Minimized Oracle for practical use in online systems and
achieves significant WAF reductions over SOTA policies on a large set of
real-world workloads.

### Strengths

The authors articulate the motivation behind the work clearly and demonstrate
significant improvements over SOTA. The paper and work are very thorough, and
rigorously prove/demonstrate the effectiveness of the theory behind the
OP-Minimized Oracle and practicality of the MINOS design.

### Weaknesses

While the evaluation is thorough and comprehensive, there are a few aspects
and takeaways that are unclear.

## Comments for authors

Thank you for your submission to FAST'26! I was very impressed with your paper.
I enjoyed all your insights in this problem space and find the work quite
impactful. I found the formal data separation model and its accompanying
figures not only effective for communciating the insights on the tradeoff
between WAF and OP space, but also a great help for facilitating understanding
of the differences between SOTA and your work.

Overall, I find your paper to be very strong and cannot overstate my apprecation
for the level of rigor displayed with all the formalism, data measurement, and
analysis. The evaluation across the various performance metrics is comprehensive
and thorough. I did have some small concerns about the evaluation, which I
expand more on in the detailed comments. But at a high level, there are certain
aspects and takeaways from the presented results which left me with some
confusion---I believe these can all be addressed with the inclusion of
additional information.

### Detailed comments

- Page 10 (Figures 12 and 13): I think that Figure 13 is incredibly effective
  for visualizing the data separation efficiency of the baselines, and was
  wondering if it'd be possible to also include the Tencent and Ali Volume
  workloads. On that note, I think it'd be also nice to see the WAF CDF of the
  Tencent, Ali, and Cloud backend traces in Figure 12 as well. I understand the
  page space constraint, and appreciate the Figures 12 and 13 together span all
  the workloads in Figure 11.

- Page 10 (Figure 13): I was wondering about the rationale in the selection of
  the baselines that MINOS is compared to. I initially thought that the selected
  baseline for each backend workload would be the next-best policy according to
  Figure 12, but that appears to not be the case. I guess maybe the intention was
  to try and show each of the non-MINOS baselines?

- Page 11 (Ablation Study and Algorithmic Parameter Sensitivity): I'm a bit
  confused about the information presented in Figure 17 in relation to the
  described takeaways from the ablation study (and vice versa). For example, the
  paper includes the following statement on longer history lengths:

  > "We use an ID-Vector history length of L equal to the number of segments in the system, as longer histories were found to hurt performance by reacting too slowly to workload changes."

  But, Figure 17 reports that a 2x longer history length results in about the
  same GC write reduction as the shorter 0.25x and 0.5x history lengths. It also
  isn't clear exactly how the number of virtual streams for user and GC data
  respectively contribute to the GC write reduction, since the configurations
  starting from, and to the right of, 8U4G seem to have similar results. In
  particular, I'm curious about why 12U4G (not shown in Figure 17) was selected
  in particular and not 8U4G. The same goes for the reclustering interval of 8
  segment writes (not show in Figure 17).

- Page 11 (Adaptability to Dynamic Workloads): I think it'd be nice if there
  was some way to quantify the WAF volatility visualized in Figure 18. Especially
  since it's a bit hard to make out the instantaneous WAF for the different
  baselines (e.g., SepBIT and MiDA between the ticks at 2TiB and 3TiB).

- Page 12 (Memory Overhead): It isn't clear to me where the average WAF values in
  Table 5 come from. The computational overhead analysis explicitly mentioned
  a 4KiB random write workload, but there doesn't seem to be a corresponding one
  for the memory overhead analysis. I think it'd also be nice to reiterate that
  AggN (e.g., Agg8 and Agg32) indicates the number of adjacent blocks that share a
  metadata entry.

### Nits

- Page 5 (Reducing OP Space with Data Separation, last paragraph): "the total
  integrated volume this buffered invalid data" -- extra "this".

- Page 11 (Figure 20): I think that indicating the WAF for the different
  workloads can be done with a horizontal line, or should at least be done using a
  consistent shape across baselines.

- Page 12 (Computational Overhead): "4,KiB" -- extra comma
