# Do learned world representations causally support novel inference?

**Status:** proposed reproduction and extension; hypotheses untested by this project.  
**Proposer:** [weiks](https://github.com/weiks), developed in discussion with OpenAI Codex; Codex assisted with design, source discovery, and drafting.  
**Proposal date:** 2026-09-30.  
**Stable proposal path:** `proposals/causal-world-representations.md`.

## Question and significance

Does a sequence model use a learned representation of the state of a world to make correct predictions on unfamiliar cases, or can observed performance be explained without that representation playing the proposed causal role?

The first experiment concerns Othello board states and legal next moves. Here, “novel inference” means predicting legal moves for positions and structural cases excluded from a documented training split. It does not mean inventing a concept, proving philosophical understanding, or merely processing a new wording.

The mechanistic hypothesis is that a state representation mediates predictions: selectively changing it should change the predicted legal moves in the corresponding direction, and restoring it should restore the predictions. Decodability alone does not establish this.

A staged follow-up asks which training conditions produce that mechanism. A representation can emerge without explicit state supervision and still be caused by the training data and objective. This project treats those as compatible descriptions.

## Motivating source and attribution

Andy Masley, *Why I believe current AI models can really think and understand — Part 1 of 2*.

Exact original URL: https://blog.andymasley.com/p/why-i-believe-current-ai-models-can

Relevant passage, in “Conclusion”:

> “whether it can represent the world with models”

Context, paraphrased: Masley proposes examining specific cognitive abilities rather than assuming that thinking requires a human-like inner observer. His predictive-processing discussion connects learned prediction with internal representations. Part 1 primarily motivates this perspective; the case for current models is deferred to Part 2.

This experimental question and its controls were developed by the proposer and Codex in the discussion leading to this repository. They are not a study suggested by Masley, and no endorsement is implied. His essay motivates the research; it supplies no result for this proposed experiment.

The essay has no canonical DragonForge claim reference available to this project. No such reference is invented. Existing Peer Review claims about other projects are not evidence for this hypothesis and are not assigned to this proposal.

## Existing experimental starting point

Recommended reproduction fork: https://github.com/likenneth/othello_world

Inspected upstream revision: `f23bb5696cf30b93bd8af8a391ee33fc3aac417e`.

Pinned instructions: https://github.com/likenneth/othello_world/blob/f23bb5696cf30b93bd8af8a391ee33fc3aac417e/README.md

The upstream project provides training, probing, and intervention code. Its README directs readers to:

- `environment.yml` for setup;
- `train_gpt_othello.ipynb` for training;
- `train_probe_othello.py` for probes;
- `intervening_probe_interact_column.ipynb` for interventions.

Relevant original paper: Kenneth Li and colleagues, *Emergent World Representations: Exploring a Sequence Model Trained on a Synthetic Task*, https://arxiv.org/abs/2210.13382. Its abstract reports board-state representations and output-controlling interventions. Those are the authors' findings, not results reproduced here.

Before claiming novelty, review related work, including https://arxiv.org/abs/2310.07582 (layer-dependent causal representations) and https://arxiv.org/abs/2503.04421 (broader model comparisons). These sources were identified during proposal preparation; a full methods review and novelty assessment remain prerequisites. The immediate value is a transparent reproduction with specified controls, not a claimed first discovery.

Fork the upstream code into a separate experimental repository, preserve upstream attribution, and record the starting revision. This proposal repository contains no copied implementation. Confirm licensing and redistribution permissions for code, weights, and datasets before distributing derived artifacts.

## Concrete approach

### Stage A: reproduce a bounded intervention

1. Verify one available checkpoint and a small held-out collection of legal games. Record checkpoint and data digests, tokenization, turn convention, environment, and source revision. Audit the rules engine, including passes and terminal positions.
2. Reproduce baseline next-move legality and the upstream probe/intervention procedure. Keep probe fitting, layer selection, and intervention tuning separate from final evaluation.
3. Construct source/donor position pairs whose relevant board-state differences predict a known change in legal moves. Use reachable legal states as the primary experiment; report artificial board edits separately. Do not silently treat an impossible edited board as an ordinary game state.
4. Intervene on candidate state representations at prespecified sites. Measure both movement toward the donor state's legal-move distribution and damage to unrelated predictions. Validate whether the intended state feature changed, but do not use probe success as the sole endpoint.
5. Restore the original activation and check recovery. Record every attempted pair, failures, exclusions, and sensitivity analyses.

Existing checkpoints can support reproduction, but may lack auditable training membership. Without it, describe evaluation positions as held out from this evaluation's fitting/tuning, not proven absent from model training. Training-novelty claims require Stage B's controlled data.

### Stage B: controlled training and generalization

Train small models from scratch on generated games with logged seeds and exact splits. Deduplicate board states as well as histories and control for board symmetries. Prespecify a structural holdout, such as selected combinations of capture directions, and verify its absence from training. Report both ordinary held-out and structural-holdout performance.

The first training comparison keeps architecture and next-token objective fixed while varying whether the training stream includes explicit board-state annotations. Use a token-matched auxiliary-data control and report both training tokens and number of game positions seen; these quantities cannot be assumed equivalent when annotations add tokens. This tests a specific effect of added state supervision, not every possible origin of the mechanism.

A later objective comparison may contrast next-token prediction with an auxiliary state-prediction loss on the same underlying games. Explicitly recognize that this also adds supervisory information. Any claim isolating objective from information exposure needs an additional matched-supervision design before execution.

Use multiple training seeds, retain intermediate checkpoints, and preregister budgets and analyses after a feasibility pilot but before confirmatory runs. No sample size, compute budget, or power claim has yet been established.

## Required comparisons and controls

| Comparison | Purpose |
| --- | --- |
| Unmodified model and sham intervention | Establish baseline and detect instrumentation effects. |
| Targeted edit versus equal-norm random and unrelated-feature edits at the same site | Separate semantic specificity from generic activation disruption. |
| Correct donor versus mismatched donor | Test whether the direction of change follows the proposed state information. |
| Intervention followed by restoration | Check reversibility and implementation integrity; recovery alone is not proof of semantic mediation. |
| Unchanged features and unaffected legal moves | Measure collateral damage rather than reporting only targeted success. |
| Probe training on shuffled labels and independent evaluation splits | Detect probe fitting artifacts and leakage. |
| Simple sequence-frequency baselines and a rules-engine oracle | Bound performance relative to surface predictors and known correct answers. |

Primary behavioral measures should include probability mass on oracle-legal moves, probability changes on moves whose legality changes between the paired states, and changes on unaffected moves. Report denominators and uncertainty, clustering by game and distinguishing training seeds from repeated positions. Freeze layers and edit strengths before final evaluation. Register null and inconclusive outcomes alongside positive ones.

## What different outcomes would mean

- **Specific, predicted changes exceeding matched controls, with recovery and limited collateral damage:** supports a causal role for the tested representation in this model/task. It does not show that this is the only mechanism or establish human-like understanding.
- **Accurate decoding without specific intervention effects:** the representation is readable, but the proposed causal role remains unestablished. An ineffective intervention is another explanation; validate manipulation before interpreting a null.
- **Targeted and random edits behave similarly:** generic disruption is sufficient to explain the experiment; the semantic interpretation is unsupported by that comparison.
- **Causal effects on ordinary cases but poor structural transfer:** a useful mechanism can coexist with limited generalization.
- **The mechanism develops without explicit state annotations:** consistent with acquisition through predictive training; not independence from training data or objectives.
- **Adding annotations changes mechanism or transfer:** evidence for that training intervention under the matched conditions, not proof that unannotated training cannot produce it.

Successful activation editing can impose information the model would not normally use. Converging evidence from selective disruption, donor controls, recovery, and held-out performance is therefore needed. No outcome here settles phenomenal consciousness or whether humans implement the same process; a human comparison would be a separate study.

## Available materials, instructions, and missing prerequisites

Available here: the proposal and source links only. Available upstream: code, environment instructions, and README links to datasets, model checkpoints, and probe checkpoints. Those external downloads have not been retrieved or hash-verified by this project.

To begin after making the reproduction fork:

```sh
git clone https://github.com/likenneth/othello_world.git
cd othello_world
git checkout f23bb5696cf30b93bd8af8a391ee33fc3aac417e
conda env create -f environment.yml
conda activate othello
```

These are starting instructions adapted from upstream, not commands validated in this project. Follow the pinned README to obtain weights/data and select the intervention notebook. Preserve raw notebook outputs and fail on execution errors; a completed notebook file is not proof of a successful run.

Missing prerequisites: verified downloads and licenses; a reproducible environment lock; a GPU-memory/runtime smoke test; audited oracle and pairing generator; auditable training splits; intervention controls; a noninteractive runner; registered metrics, seeds, exclusions, and budget. Upstream's documented default training setup is substantial; this proposal makes no claim that full training is a quick or inexpensive run. Begin with checkpoint-based reproduction and measure requirements before committing to retraining.

## First useful contribution

Produce a bounded reproduction audit for one checkpoint: confirm which files can be obtained, record hashes and environment, execute the unmodified baseline and one intervention with sham and equal-norm random controls, and retain complete per-case outputs. Report missing prerequisites or unsuccessful reproduction as such.

The contribution should include the experimental repository and exact commit, setup/run commands, chosen cases and selection procedure, hardware/runtime, outputs for every arm, and a short explanation of what the comparison does and does not establish. This audit is a feasibility contribution; it must not be relabeled a confirmatory finding about novel inference.

## Publication and ingestion

This file is the source-of-record proposal. Cite its repository URL, stable path, and full publication commit. Future results should be separately identified and linked, with their own execution/code/data references; preserve this proposal's history. Peer Review may ingest it as proposed research, not as an accepted canonical claim. No new DragonForge schema is introduced.
