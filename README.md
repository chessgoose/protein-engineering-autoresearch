# PETase Autoresearch







This repository is a sketch for adapting [karpathy/autoresearch](https://github.com/karpathy/autoresearch) to PETase engineering, with the [Align Foundation 2025 PETase Tournament](https://alignbio.org/get-involved/competitions/2025-petase-tournament/) as the concrete motivating use case.

On March 25, 2026, the competition is already in the generative phase:

- Predictive phase ended on February 20, 2026
- Generative phase round 1 began on March 9, 2026
- Generative phase round 2 begins on July 20, 2026
- Winners are scheduled to be announced on November 20, 2026

That means the most useful framing for this repo is not a generic protein-design sandbox, but an autonomous PETase design workflow that can:

- rank PETase variants
- generate new PETase sequences for the tournament
- learn from round-1 assay feedback before round 2

The core idea is the same:

- let an AI agent run many small, reviewable research cycles
- give it a tight objective, a constrained toolchain, and fixed evaluation rules
- keep the human focused on defining the problem, approving risk, and deciding what is worth taking into the lab

Instead of editing `train.py` to improve a language model, the agent edits experiment plans, prompts, configs, candidate sets, and analysis code to improve a PETase design campaign.

## Competition Fit

The Align tournament is especially well-suited to an `autoresearch`-style setup because the task is already organized around repeated evaluation.

The official evaluation criteria are:

- Predictive phase: teams are ranked by average NDCG across property-prediction tasks
- Generative phase: teams are ranked by experimentally measured improvement in specific activity relative to a benchmark, while maintaining threshold expression

The tournament also provides:

- sequence-based prediction tasks
- training data on natural PETase sequences and mutational variants
- two iterative generative rounds with experimental feedback between rounds

That is almost exactly the kind of loop autonomous research systems need.

## What This Looks Like For PETase

An autonomous protein-engineering loop should look less like "invent biology from scratch" and more like:

1. define a PETase-specific objective and tournament metric
2. gather prior PETase sequences, structures, mutants, and benchmark enzymes
3. generate PETase variants or focused redesigns
4. score them with existing computational tools
5. rank, cluster, and prune
6. propose the next tournament submission batch
7. repeat with a permanent memory of what worked and failed

This is closer to an AI research organization wrapped around established protein-design software than a single monolithic model.

## The Right Scope

For this repo, the best scope is something like:

- improving PETase specific activity under the tournament assay
- maintaining expression while pushing activity upward
- preserving fold and catalytic machinery while exploring sequence space
- using round-1 assay feedback to improve round-2 submissions

This works poorly if the objective is vague, the assay is undefined, or the agent is allowed to optimize proxy metrics with no reality check.

## Mapping `autoresearch` to Protein Engineering

In `autoresearch`, the loop is roughly:

- edit code
- train for a fixed budget
- evaluate on a fixed metric
- keep or discard changes

In protein engineering, the equivalent loop becomes:

- edit the campaign plan, prompts, configs, and candidate-generation policy
- generate a fixed-size batch of candidates
- run a fixed computational evaluation stack
- compare against baseline and archive all results
- keep or discard the policy change

### Analogy Table

| `autoresearch` concept | Protein engineering analogue |
| --- | --- |
| `program.md` | campaign charter, agent SOPs, scoring rubric, safety constraints |
| `train.py` | candidate-generation prompts, workflow configs, ranking logic, analysis scripts |
| fixed 5-minute run | fixed compute budget per design round |
| validation loss | campaign score: weighted multi-objective score |
| best checkpoint | best candidate set / best experimental batch proposal |

## Existing Protein Engineering Tools To Use

The point is not to replace mature tools. The point is to orchestrate them.

### Search, Retrieval, and Context

- [MMseqs2](https://github.com/soedinglab/MMseqs2): fast sequence search and clustering
- [Foldseek](https://github.com/steineggerlab/foldseek): fast structure search for analogs and novelty checks
- BLAST / UniProt / PDB / AlphaFold DB: prior-art and homolog retrieval

### Sequence and Structure Foundation Models

- [ESM](https://github.com/facebookresearch/esm): embeddings, variant scoring, inverse folding, structure-related priors
- [ColabFold](https://github.com/sokrypton/ColabFold): practical structure prediction and complex prediction pipeline
- AlphaFold-class tools: structure confidence, interface hypotheses, and fold plausibility

### Generative Design

- [ProteinMPNN](https://github.com/dauparas/ProteinMPNN): sequence design conditioned on backbone structure
- [RFdiffusion](https://github.com/RosettaCommons/RFdiffusion): de novo backbone generation, motif scaffolding, binder-oriented design
- task-specific LMs or fine-tuned models for local mutational proposals

### Physics and Energy-Based Evaluation

- [Rosetta](https://github.com/RosettaCommons/rosetta) / [PyRosetta](https://www.pyrosetta.org/): packing, interface scoring, relax, mutation scans
- FoldX: fast mutation energy estimates
- MD packages such as OpenMM or GROMACS: heavier-weight stability checks when justified

### Optimization and Experiment Planning

- Bayesian optimization tools such as [BoTorch](https://github.com/pytorch/botorch)
- simple evolutionary search, beam search, or bandit-style allocation
- active learning around assay results once wet-lab data exists

## Recommended Agent Workflow

The AI should behave like a small computational protein-design team.

### Agent 1: Research Lead

Responsibilities:

- define the target product profile
- write the scoring rubric
- decide what experiment class to run next
- summarize evidence and uncertainty

### Agent 2: Retrieval Scientist

Responsibilities:

- find homologs, structures, motifs, and known mutations
- assemble competitor and literature context
- build reusable design context for the other agents

### Agent 3: Design Scientist

Responsibilities:

- propose mutations, sequence libraries, or de novo backbones
- call ProteinMPNN, RFdiffusion, ESM-based generators, or template-based design workflows

### Agent 4: Evaluation Scientist

Responsibilities:

- run structure prediction
- run stability/interface/novelty/developability checks
- produce a standardized scorecard for every candidate

### Agent 5: Lab Planner

Responsibilities:

- select the next wet-lab batch
- optimize for diversity, information gain, and cost
- write the final recommended experiment set

## PETase Tournament Loop

For the Align PETase tournament, I would structure the loop like this:

1. **Campaign setup**
   - Define the tournament objective, constraints, and submission budget.
   - Example: "Increase PETase specific activity over the benchmark while maintaining expression threshold and preserving catalytic competence."

2. **Baseline context build**
   - Pull homologs with MMseqs2.
   - Pull PETase structures or predicted structures.
   - Build an initial mutation map, active-site map, substrate-contact map, and conserved-residue mask.

3. **Hypothesis generation**
   - Ask the agent to produce several explicit design hypotheses.
   - Example: active-site reshaping, surface-charge tuning, loop rigidification, consensus-guided mutations, expression-rescuing compensatory mutations.

4. **Candidate generation**
   - Use the appropriate generator.
   - Existing PETase backbone + sequence redesign -> ProteinMPNN
   - Local sequence exploration around known PETase scaffolds -> ESM/LM-guided mutagenesis
   - Backbone perturbation or motif-preserving redesign -> RFdiffusion only if the tournament rules and design risk justify it

5. **Fast computational triage**
   - remove duplicates and near-duplicates
   - filter by sequence liabilities
   - cluster for diversity
   - reject obvious catalytic or folding disruptions

6. **Structure-based evaluation**
   - run ColabFold or another structure pipeline
   - compare predicted fold to PETase reference structures
   - estimate confidence, catalytic geometry retention, substrate-pocket shape, and packing quality

7. **Energy and developability scoring**
   - Rosetta/FoldX metrics
   - aggregation heuristics
   - expression proxies
   - thermostability proxies
   - mutation-load penalties so the agent does not drift too far too early

8. **Batch selection**
   - choose a small, diverse batch
   - include benchmark-like controls and exploratory outliers
   - preserve enough diversity to learn from failures

9. **Memory update**
   - store all prompts, configs, scores, failures, and decisions
   - update the agent's beliefs about what changes are helping PETase activity versus expression

10. **Repeat**
   - the next round should modify the PETase design policy, not just sample more of the same candidates

## A Practical Scoring Function

A protein-engineering `val_loss` equivalent should be explicit and hard to game.

Example PETase composite score before real assay data:

```text
total_score =
  0.30 * structure_confidence
  0.20 * catalytic_site_preservation
  0.15 * substrate_pocket_quality
  0.15 * stability_proxy
  0.10 * expression_proxy
  0.05 * developability_score
  0.05 * diversity_bonus
  - penalties for liabilities, motif drift, and excessive mutation load
```

Important rule: do not let the agent optimize a single proxy like pLDDT or Rosetta energy in isolation. That is how autonomous loops become confidently wrong.

After round-1 tournament feedback, this should shift to a data-driven surrogate model trained directly on the measured PETase outputs that Align releases to participants.

## Suggested Repo Layout

If we were to build this repo in the spirit of `autoresearch`, I would keep it small and opinionated:

```text
README.md
program.md                  # the campaign charter and agent instructions
campaigns/
  petase_tournament/
    objective.yaml
    constraints.yaml
    known_data/
    results/
workflows/
  retrieve_homologs.sh
  run_proteinmpnn.sh
  run_rfdiffusion.sh
  run_colabfold.sh
  run_scoring.py
  select_batch.py
memory/
  experiment_log.jsonl
  accepted_hypotheses.md
  rejected_hypotheses.md
reports/
  latest_summary.md
```

## What `program.md` Should Contain

This is the most important file in an `autoresearch`-style setup.

It should tell the agent:

- the biological objective
- what success means
- which residues or motifs are untouchable
- what tools it may call
- the exact evaluation order
- what gets logged every round
- when to stop
- when human approval is required

Example rules:

- never mutate catalytic residues unless the campaign explicitly allows it
- always include wild type and best-known positive control in each selected batch
- never recommend synthesis from a single metric alone
- preserve campaign diversity; avoid 20 nearly identical variants
- summarize uncertainty, not just top scores

## Example Workflows

### 1. Enzyme Thermostability Campaign

Input:

- one enzyme sequence
- optional structure or AlphaFold model
- activity assay

Loop:

- retrieve homologs and consensus positions
- propose conservative stabilizing mutations
- score fold retention and stability
- select diverse low-risk variants
- incorporate assay results into the next round

Best tools:

- MMseqs2
- ESM
- ColabFold
- Rosetta / FoldX
- Bayesian optimization after first assay rounds

### 2. Binder Design Campaign

Input:

- target structure and epitope definition

Loop:

- use RFdiffusion to scaffold binders around the target geometry
- use ProteinMPNN for sequence design
- use ColabFold for complex prediction
- rank by interface confidence, clash-free packing, and diversity

Best tools:

- RFdiffusion
- ProteinMPNN
- ColabFold
- Rosetta interface analysis
- Foldseek for novelty / analog checks

### 3. Mutational Scan Triage

Input:

- existing protein and a large mutational design space

Loop:

- prioritize singles and pairs with model-based scores
- cluster by mechanism
- propose compact batches that maximize information gain

Best tools:

- ESM variant scoring
- Rosetta/FoldX mutation scans
- active learning with assay feedback

## Where AI Helps Most

AI is most useful for:

- turning literature and database context into explicit design hypotheses
- choosing which computational tool to run next
- synthesizing conflicting evidence across many scoring outputs
- updating search strategy after each batch
- writing standardized reports and preserving memory across rounds

AI is less trustworthy for:

- claiming real-world function from proxy scores alone
- inventing undocumented biology
- skipping controls
- making biosafety-sensitive decisions without oversight

## Guardrails

A useful autonomous system needs constraints.

Minimum guardrails:

- human approval before any synthesis or wet-lab escalation
- explicit biosafety and dual-use review
- immutable experiment logs
- strict versioning of prompts, tool configs, and score weights
- mandatory controls in every selected batch
- rejection of unsupported claims like "likely active" without assay evidence

## The Most Important Design Principle

Do not ask the agent to "design the best protein."

Ask it to:

- operate a tight, auditable design loop
- generate competing hypotheses
- run established tools consistently
- learn from results faster than a human manually could

That is the protein-engineering equivalent of `autoresearch`.

## If We Build This Repo Further

The next logical steps would be:

1. write `program.md` for the PETase tournament
2. add thin wrappers for MMseqs2, ProteinMPNN, RFdiffusion, ColabFold, and scoring
3. define one PETase-specific scorecard schema
4. add a memory store for all tournament rounds
5. add a report generator that proposes the next submission batch

## PETase-Specific Strategy

If we were entering this competition, I would not start with fully unconstrained de novo design. I would start with a conservative PETase improvement stack:

1. build a PETase retrieval set from known sequences, homologs, and public structures
2. identify conserved catalytic residues and positions that should almost never be touched
3. separate mutation ideas into buckets:
   - likely activity-improving
   - likely expression-improving
   - likely stability-improving
   - high-risk exploratory
4. train or prompt a ranking model to predict the three tournament-relevant properties:
   - activity
   - expression
   - thermostability if available in the provided data
5. use the generative agent to propose variants that are diverse across mechanisms, not just pointwise top-scoring
6. reserve a fraction of each batch for exploration so the system keeps learning

## What The AI Should Optimize In This Competition

The tournament setup suggests two distinct autonomous modes.

### Predictive Mode

Goal:

- rank provided PETase variants well on the measured properties

Best AI workflow:

- build ensembles from sequence models, structure-aware models, and simple baselines
- calibrate predictions for ranking, not only for absolute regression accuracy
- optimize directly for NDCG-like behavior

### Generative Mode

Goal:

- submit PETase sequences that improve measured specific activity while maintaining expression thresholds

Best AI workflow:

- use predictive models as filters, not truth
- generate multiple mechanistic families of candidates
- choose submissions by expected value plus diversity plus information gain
- treat round 1 as both a performance round and a data-acquisition round for round 2

## Tournament Notes

Relevant official details from the Align Foundation page:

- Predictive phase ranking uses average NDCG across properties
- Generative phase ranking is based on experimentally measured specific activity relative to benchmark, while maintaining expression thresholds
- The tournament provides training data on natural PETase sequences and mutational variants
- Round 2 includes experimental feedback on each team's submitted designs
- Top-performing teams are de-anonymized at the end of the tournament

This makes PETase a much better first target for this repo than a generic "protein engineering" placeholder because the benchmark, timeline, and feedback loop are all already defined.

## References

- [karpathy/autoresearch](https://github.com/karpathy/autoresearch)
- [Align Foundation 2025 PETase Tournament](https://alignbio.org/get-involved/competitions/2025-petase-tournament/)
- [ProteinMPNN](https://github.com/dauparas/ProteinMPNN)
- [RFdiffusion](https://github.com/RosettaCommons/RFdiffusion)
- [ESM](https://github.com/facebookresearch/esm)
- [ColabFold](https://github.com/sokrypton/ColabFold)
- [MMseqs2](https://github.com/soedinglab/MMseqs2)
- [Foldseek](https://github.com/steineggerlab/foldseek)
- [Rosetta](https://github.com/RosettaCommons/rosetta)
- [PyRosetta](https://www.pyrosetta.org/)
- [BoTorch](https://github.com/pytorch/botorch)
