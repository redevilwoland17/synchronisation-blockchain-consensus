# Physical synchronisation for anomaly detection in epidemic blockchain consensus

Simulation and analysis code accompanying the paper published in *Scientific Reports*.

> Mordarski, M., Magri, L. & Knottenbelt, W. **Physical synchronisation for anomaly detection in epidemic blockchain consensus.** *Scientific Reports* (2026). https://doi.org/10.1038/s41598-026-61354-4

The paper formalises the analogy between gossip-based blockchain consensus and coupled-oscillator synchronisation: under a phase encoding, the dynamics of the Blockchain Epidemic Consensus Protocol (BECP) reduce, at leading order and in a weak-coupling regime, to the Kuramoto model on the peer-to-peer graph. Consensus disruptions then appear as losses of phase coherence, giving a candidate anomaly signal. On real Bitcoin data — where continuous phase dynamics are not observable — a static phase *proxy* built from per-pool block-attribution statistics identifies the March 2013 chain fork at 5.1σ.

## Citation

If you use this code, please cite both the paper and the archived software release:

```bibtex
@article{mordarski2026synchronisation,
  title   = {Physical synchronisation for anomaly detection in epidemic blockchain consensus},
  author  = {Mordarski, Marcel and Magri, Luca and Knottenbelt, William},
  journal = {Scientific Reports},
  year    = {2026},
  doi     = {10.1038/s41598-026-61354-4}
}
```

A `CITATION.cff` file is included, so GitHub's "Cite this repository" button produces the same reference. The software DOI is shown in the badge at the top of this file once the first release is archived.

## Repository layout

The repository is deliberately split in two. Scientific integrity and readability pull in different directions; rather than compromise on either, both are provided.

```
original_scripts/    exact code used to produce the published results
cleaned_pipeline/    refactored, documented equivalent
```

### `original_scripts/` — the record

These are the files as they were run, unmodified. They are the authoritative reference for anything in the paper. If a result in the manuscript is ever in question, this is the code that produced it. Nothing here has been tidied, renamed or restructured, and the notebooks retain their embedded figure outputs so that the published plots can be inspected without re-running anything.

The directory contains several superseded iterations (`kuramoto_blockchain_v2.py`, `_corrected`, `_combined`, `_extended`) alongside the authoritative `kuramoto_blockchain_v3.py`. They are kept because they document how the implementation evolved, but **`kuramoto_blockchain_v3.py` is the version behind the published results**.

### `cleaned_pipeline/` — the usable version

A reorganised package with the same numerical behaviour, intended as the starting point for anyone building on this work. The scientific content is unchanged: no logic, mathematical operation or data transformation has been altered, shortened or reordered in any way that affects results.

```
cleaned_pipeline/
├── becp/
│   ├── config.py      SimulationConfig, MetricsSnapshot, literature reference values
│   ├── networks.py    BA / Watts–Strogatz / Erdős–Rényi coupling graphs
│   ├── models.py      KuramotoModel, BECPModel, shared coupling weight K_ij
│   └── controls.py    swappable coupling kernels for the discriminating control
├── scripts/
│   ├── run_mapping_validation.py   order-parameter comparison (Fig. 1)
│   └── run_negative_control.py     coupling-kernel control (Supplementary Note 9)
└── requirements.txt
```

## Environment setup

Python 3.9 or newer.

```bash
git clone https://github.com/<user>/<repo>.git
cd <repo>/cleaned_pipeline

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

pip install -r requirements.txt
```

## Running the pipeline

From inside `cleaned_pipeline/`:

```bash
# Order-parameter correspondence between BECP and epidemic Kuramoto
python scripts/run_mapping_validation.py --seeds 5

# Discriminating control: does the correspondence survive a wrong coupling?
python scripts/run_negative_control.py --seeds 10
```

The first completes in under a minute; the second runs eight kernels across ten seeds and takes a few minutes.

Or use the package directly:

```python
from becp import SimulationConfig, create_network, BECPModel, KuramotoModel

cfg = SimulationConfig(n_nodes=100, topology='BA', coupling_func='f2',
                       coupling_prob=0.5, coupling_strength=1.5, n_steps=300, seed=42)
G = create_network(cfg)

r_becp = [s.order_parameter for s in BECPModel(G.copy(), cfg).run()]
r_kuramoto = [s.order_parameter for s in KuramotoModel(G.copy(), cfg).run()]
```

⚠️ Both models reseed NumPy's global generator in their constructor. Construct and run one model fully before constructing the next; building both first and running them afterwards gives the second a different random stream.

### Empirical Bitcoin analyses

`historical_validation.py` reconstructs the March 2013 fork analysis and `granger_causality.py` tests the causal direction between the order parameter and chain-quality metrics. Both are standalone command-line tools.

The block cache they operate on (`block_cache_2013_v2.json`, blocks 224,900–225,900) is **not redistributed here**. `historical_validation.py` rebuilds it from the public Blockchair API on first run, or accepts a TSV dump via `--from-tsv`; the block range and pool-attribution signatures are specified in the script and in the paper, so the dataset is fully reconstructible.

```bash
python historical_validation.py          # fetches and caches block data
python granger_causality.py              # requires the cache above
```

## Use of AI-assisted technologies

As stated in the paper's Acknowledgements, AI-assisted technologies were used to confirm the literature gap, implement code, and suggest text improvements. For transparency, this extended to the code in this repository: large language models (Anthropic's Claude) were used to assist in writing portions of the original implementation, and to carry out the refactoring of `original_scripts/` into `cleaned_pipeline/`.

## Licence

MIT — see [`LICENSE`](LICENSE). The paper itself is published open access under CC BY 4.0.

## Contact

Marcel Mordarski — marcel.mordarski25@imperial.ac.uk
Department of Computing, Imperial College London
