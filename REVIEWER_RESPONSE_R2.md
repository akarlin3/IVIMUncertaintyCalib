# Response to reviewer comments — second round

This round addresses the four remaining items from the editor's assessment of the
revised manuscript (two Medium, two Low). As in the first round
(`REVIEWER_RESPONSE.md`), each item below states what was done, gives drop-in
manuscript text, and — where a concrete artifact was produced — names the
committed file and the command that reproduces it. All numbers come from scripts
in `npe/` run against the committed model (`npe/npe_posterior_setB.pt`).

| # | Severity | Comment | Status |
|---|----------|---------|--------|
| 1 | MEDIUM | Figure 5 is a single in-vivo case (N=1) with a 54.7%-railed NLLS reference | **Decision required (data-blocked):** only one in-vivo abdominal dataset is available locally; cannot add cases. Drop-in text supplied for both resolutions (demote to supplementary vs. retain as a single labelled illustration). |
| 2 | MEDIUM | Generality across priors untested (architecture S1 and density S2 are controlled; prior choice is not) | Acknowledged in-text (done in round 1); tightened limitation text below, plus a full spec for the optional prior-sensitivity ablation that would convert it to a result. |
| 3 | LOW | Timing benchmark (418 ms / 0.956 s / 635 s per 1,000 voxels) lacks methods: hardware, implementation parity | **Done** — new methods-documented benchmark (`npe/run_h_benchmark.py`, `npe/benchmark_timings.csv`), drop-in methods note, and a fix for a warm-up bug in the ad-hoc script that had inflated the NPE number ~3–6×. |
| 4 | LOW | Gaussian CRLB approximation weakest for skewed D*, where the central claims sit | Disclosed in Limitations (done); tightened drop-in text below, plus a spec for the optional sampling-based bound on a low-SNR subset. |

---

## 1 — MEDIUM: Figure 5 single in-vivo case (N=1)

**Comment.** *"Figure 5 remains a single in-vivo case (N=1) with a 54.7%-railed
NLLS reference, in the main text. To fully close: add 1–2 more abdominal cases, or
demote to supplementary. This is the highest-value remaining lift."*

**Assessment.** This is the highest-value item, but the "add 1–2 more abdominal
cases" path is **blocked by data availability, not by code**. The repository
ships exactly one in-vivo abdominal acquisition
(`download/Data/abdomen.nii.gz`, with `mask_abdomen_homogeneous.nii.gz` and
`abdomen.bval`) plus the brain dataset; there is no second or third abdominal
acquisition locally, and none is available from the open OSIPI data we draw on.
The batch pipeline is already written to pick up any additional cases the moment
they are provided — `fig5_batch.py` globs `download/Data/abdomen_case*.nii.gz`
and processes every match through the identical NPE-vs-NLLS comparison — so adding
cases is a data-supply step, not an analysis step. **If 1–2 further abdominal
IVIM acquisitions (4D NIfTI + b-value file + an ROI mask) can be provided, the
multi-case figure regenerates automatically with no code changes.**

Absent that data, the actionable resolution is editorial, and there are two
defensible options. The recommendation is **(A) demote to supplementary**, because
it removes a single-subject anecdote from the main text without weakening any
quantitative claim — every load-bearing result (the CRLB efficiency audit, the
held-out-b miscalibration, the OOD gate) rests on the simulation study and the
gray-matter brain analysis, not on the abdominal case. Option (B) retains it in
the main text but only if it is explicitly labelled as a single mechanistic
illustration rather than evidence.

On the 54.7% NLLS railing specifically: this is itself a finding, not a defect of
the figure — it is the per-voxel manifestation of the well-posedness problem the
paper is about (the NLLS D\* estimate hits a bound in a majority of voxels because
D\* is weakly identified from the clinical-sparse acquisition). The comparison in
`fig5_batch.py` already excludes boundary-railed NLLS voxels before computing the
SD/IQR ratios, so the reported spread comparison is not contaminated by the rail;
the railing fraction should be **reported as a result** wherever the figure lands.

**Proposed manuscript edit — Option A (recommended), supplementary caption.**

> *Supplementary Figure S4. Single-subject in-vivo illustration (abdomen). NPE and
> NLLS D\* maps for one open IVIM acquisition. In 54.7% of high-SNR ROI voxels the
> NLLS D\* estimate is boundary-railed — the per-voxel signature of the weak D\*
> identifiability characterised in simulation (Figures 2–3); these voxels are
> excluded before the spread comparison. The case is shown as a qualitative
> illustration of that mechanism in vivo and is not used to support any
> quantitative claim; N = 1 precludes population inference.*

And in the main text, replace the Figure 5 reference with: *"A single in-vivo
abdominal case is shown in Supplementary Figure S4 as a qualitative illustration
of the same weak-identifiability signature; it is not used for inference."*

**Proposed manuscript edit — Option B (retain in main text).** Keep the figure but
prepend to its caption: *"This single subject (N = 1) is a mechanistic
illustration, not evidence; it shows in one in-vivo dataset the weak-D\*
identifiability quantified in simulation. The NLLS reference is boundary-railed in
54.7% of high-SNR ROI voxels — itself the per-voxel signature of that weak
identifiability — and these voxels are excluded before the NPE-vs-NLLS spread
comparison."*

Existing artifacts: `fig5_batch.py` (multi-case-ready batch pipeline),
`download/Data/abdomen.*`, `roi_hi_dstar.csv`.

---

## 2 — MEDIUM: Generality across priors

**Comment.** *"Generality across priors is untested; architecture (S1) and
acquisition density (S2) are controlled, but prior choice is not. Acknowledge as
the primary open item. A prior-sensitivity ablation would convert it from
limitation to result, but is not required for this submission."*

**Assessment.** The manuscript already names this as the primary open item (round 1
softened the limitation once acquisition density joined flow architecture as a
controlled axis). Two controlled ablations now bracket the estimator —
**architecture** (NSF vs MAF, Supplementary Figure S1) and **acquisition density**
(8-b vs 16-b, Supplementary Figure S2) — and in both the below-floor D\*
overconfidence persists, which is what licenses attributing it to the amortized
estimator under weak identifiability rather than to any one design choice. Prior
choice is the one remaining uncontrolled axis. The minimum fix (explicit
acknowledgement) is in place; the text below tightens it so the limitation reads
as "one remaining axis of a deliberately bracketed design," not an unexamined gap.

**Proposed manuscript edit (Limitations).**

> *Two controlled ablations bracket the amortized estimator: flow architecture
> (NSF versus MAF, Supplementary Figure S1) and acquisition density (8- versus
> 16-point b-schemes, Supplementary Figure S2). The below-floor D\* overconfidence
> persists across both, which is what attributes it to the estimator under weak
> identifiability rather than to a single design choice. The one axis we do not
> vary is the prior: all models use the same log-uniform-style IVIM prior. A
> prior-sensitivity ablation — re-training under a substantively different prior
> (e.g. a tissue-informed or heavier-tailed D\* prior) and re-auditing against the
> same CRLB floor — would establish whether the overconfidence is prior-induced
> reversion or intrinsic to amortization; we identify this as the primary open
> item for future work.*

**Optional ablation spec (to convert limitation → result; not required for this
submission).** This mirrors the S1/S2 machinery exactly, so it is low-risk to add:
re-train one NPE with the existing `train_npe.py` recipe but a different prior
(swap the `npe_prior` construction for, e.g., a tissue-informed Normal-on-log-D\*
or a heavier-tailed prior), audit it with `run_e_efficiency.py` against the same
CRLB grid, and render an S4-style panel with `run_s1_figure.py`'s transform. The
result that would close the item: if the D\* below-floor fraction stays high
(≈ the 41–69% seen in S1/S2) under a materially different prior, the overconfidence
is intrinsic; if it collapses, it is prior-reversion and the recommendation
changes to "choose the prior carefully." Cost ≈ one training run (~1 h CPU) plus
the audit, with no new data.

Existing artifacts: `figures/manuscript/figS1_maf_ablation.*` (architecture),
`figures/manuscript/figS2_dense_acquisition.*` (density); `train_npe.py`,
`npe/run_e_efficiency.py`, `npe/run_s1_figure.py` (reusable for the prior ablation).

---

## 3 — LOW: Timing-benchmark methods (hardware + implementation parity)

**Comment.** *"Timing benchmark (418 ms / 0.956 s / 635 s per 1,000 voxels) lacks
methods: hardware, implementation parity. Add a one-line benchmark-methods note
(hardware, that all three estimators used comparable implementations)."*

**What was done.** A methods-documented benchmark replaces the ad-hoc script. New
script `npe/run_h_benchmark.py` (Checkpoint H) times all three estimators on the
**same machine and the same CPU**, on an identical 1,000-voxel synthetic workload
(one fixed biexponential ground truth, Gaussian noise, the 10-point evaluation
b-scheme), and writes the hardware/library provenance into the output CSV
(`npe/benchmark_timings.csv`) so the numbers can never be quoted without their
methods again. Implementation parity is enforced by construction:

- **CPU-only.** No estimator uses a GPU, so the comparison is hardware-fair.
- **NPE** — amortized neural spline flow, a batched 100-sample posterior draw per
  voxel, timed at **steady state**: a warm-up draw precedes timing so the figure
  excludes one-time model load and lazy graph/JIT construction.
- **NLLS** — SciPy `least_squares` (TRF), one independent fit per voxel, serial.
- **MCMC** — the OSIPI `OGC_AmsterdamUMC_Bayesian_biexp` estimator (emcee),
  32 walkers × 1500 steps, 500 burn-in, thin 5 — the manuscript's settings —
  timed on a 50-voxel subset and linearly projected (its per-voxel cost is fixed).

**Two findings worth flagging.**

1. **The committed ad-hoc `benchmark_inference.py` had a warm-up bug**: it timed
   the *cold* first NPE call (model load + lazy build), inflating NPE latency
   3–6×. With a warm-up draw the steady-state NPE time on an idle Apple M4 is
   **≈ 410 ms / 1,000 voxels — matching the manuscript's quoted 418 ms.** The fix
   is in `run_h_benchmark.py`; the old script should be retired or warmed.
2. **Absolute wall-times are hardware- and thermal-state-dependent and are not
   stably reproducible on a commodity laptop.** On this Apple M4 the *same* warmed
   NPE workload measured 0.41 s when the machine was idle/cool and 2.5–2.9 s under
   sustained load (a ~6× swing); NLLS and MCMC vary similarly. What is invariant —
   and what carries the manuscript's claim — is the **ratio**: NPE is within ~2×
   of a single per-voxel NLLS fit-set, and **≈ 3 orders of magnitude (≈ 800–1500×)
   faster than per-voxel MCMC**.

Measured set (Apple M4, 10-core, macOS 26.3.1, CPU-only, torch 2.8.0; one full
run under load, `npe/benchmark_timings.csv`): NPE 2.54 s, NLLS 2.98 s, MCMC
2,077 s per 1,000 voxels. The manuscript's idle-machine set (418 ms / 0.956 s /
635 s) is reproducible for NPE and consistent in ordering for all three.

**Proposed manuscript edit (Methods or the timing figure/table caption — the
requested one-liner, plus the parity clause).**

> *Inference-time benchmarks were measured on a single CPU (Apple M4, 10-core,
> macOS; no GPU) with all three estimators run on identical 1,000-voxel synthetic
> workloads and comparable per-voxel implementations: the amortized NPE as a
> warmed, batched posterior draw (neural spline flow); NLLS as one SciPy
> trust-region fit per voxel; and the Bayesian baseline as MCMC (emcee, 32 walkers
> × 1500 steps). Absolute wall-times depend on hardware and thermal state (we
> observed several-fold run-to-run variation on the same machine); the
> load-bearing comparison is the relative cost — amortized NPE is comparable to a
> single NLLS fit and roughly three orders of magnitude faster than per-voxel
> MCMC.*

**Reproduce.**
```
.venv/bin/python npe/run_h_benchmark.py --out npe/benchmark_timings.csv
# fast smoke (skips the ~1 min MCMC leg):
.venv/bin/python npe/run_h_benchmark.py --skip-mcmc
```

New committed artifacts: `npe/run_h_benchmark.py`, `npe/benchmark_timings.csv`.

---

## 4 — LOW: Gaussian CRLB approximation for skewed D*

**Comment.** *"Gaussian CRLB approximation weakest for skewed D* where the central
claims sit. Disclosed in Limitations; an optional sampling-based bound on a
low-SNR subset would close it fully."*

**Assessment.** The CRLB used throughout the efficiency audit is the analytical
Gaussian (Fisher-information) bound, and the manuscript's central D\* claims do sit
where its posterior is most skewed (low SNR, weak identifiability) — exactly where
a local-Gaussian bound is least tight. This is disclosed in Limitations. The text
below tightens that disclosure to state *which direction* the approximation errs
and why the conclusion survives it.

The key point that makes this Low rather than Medium: the Gaussian CRLB is, if
anything, a **conservative reference for our claim**. Our finding is that the NPE
claims precision *below* the floor (over-confidence). Where the true posterior is
skewed, the achievable variance is generally *higher* than the Gaussian CRLB
predicts, so the true floor sits *above* the Gaussian one — which would make the
NPE's claimed SD *even further below* the real floor. The Gaussian approximation
therefore biases against, not toward, the over-confidence conclusion; a tighter
(sampling-based) bound would strengthen it, not threaten it.

**Proposed manuscript edit (Limitations).**

> *The efficiency floor is the analytical Gaussian (Fisher-information) CRLB. It is
> a local approximation and is least tight precisely where the D\* posterior is
> most skewed — at low SNR under weak identifiability, where our central claims
> sit. The direction of this error is favourable to the conclusion: where the
> posterior is skewed the achievable variance generally exceeds the Gaussian CRLB,
> so the true floor lies above the value we plot, and the estimator's claimed SD is
> then even further below the true floor. A sampling-based bound would tighten the
> reference but would not overturn the below-floor over-confidence it is used to
> demonstrate.*

**Optional sampling-based bound spec (to close fully; not required).** On a
low-SNR subset (e.g. SNR 10–20, the regime where D\* skew is worst), replace the
analytical CRLB with a Monte-Carlo estimate of the minimum achievable variance:
simulate many noise realizations at fixed ground truth, compute the efficient
estimator variance (or a posterior-sampling lower bound), and compare the NPE's
claimed SD against that empirical floor instead of the Gaussian one. The grid and
SNR sweep already exist in `npe/run_e_efficiency.py`; the change is swapping the
analytical CRLB column for a sampled-variance column on the low-SNR rows. Expected
result, per the argument above: the below-floor fraction is unchanged or larger.

Existing artifacts: `npe/run_e_efficiency.py` (CRLB grid + SNR sweep),
`npe/efficiency_map.csv`.

---

## Summary of this round's repository changes

| Item | Code / data committed | Manuscript action |
|---|---|---|
| 1 (Fig 5) | none possible without new in-vivo data; `fig5_batch.py` already multi-case-ready | **Editorial decision required** — drop-in text for demote-to-supplementary (recommended) and retain-as-illustration both supplied |
| 2 (priors) | reuses S1/S2 machinery; optional ablation fully specified | tightened Limitations text (drop-in) |
| 3 (timing) | `npe/run_h_benchmark.py`, `npe/benchmark_timings.csv` | drop-in methods note (drop-in); warm-up bug fixed |
| 4 (CRLB) | optional bound specified against existing `run_e_efficiency.py` | tightened Limitations text (drop-in) |

The single item that cannot be closed from this side is **Figure 5**: it needs
either 1–2 additional in-vivo abdominal acquisitions (after which the figure
regenerates automatically) or your sign-off to demote it to the supplement.
