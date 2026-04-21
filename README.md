# Apertus LLM Serving Trace

This repository contains the anonymised production trace from [**Public AI's**](https://openreview.net/pdf?id=uQW9yeormT) deployment of [Apertus](https://arxiv.org/abs/2509.14233), an open-source multilingual language model developed by the Swiss AI Initiative. The trace accompanies the paper:

> **Where the Time Goes: Analysis of a Public LLM Serving System**  
> Büşra Karatay Demiray, Ehsan Yousefzadeh-Asl-Miandoab, Pınar Tözün, Benoît Garbinato, Pamela Delgado  
> EuroMLSys 2026  
> DOI: [10.1145/3805621.3807650](https://doi.org/10.1145/3805621.3807650)

## Overview

The trace covers **five months** (September 2025 - January 2026) of production traffic from a public, free-access LLM inference service. It contains **337,155 successfully completed requests** of two model variants: Apertus-8B-Instruct-2509 and Apertus-70B-Instruct-2509 served through an open-source serving stack (Open WebUI (or dev workloads via Zuplo) → LiteLLM → vLLM), deployed and maintained by the Public AI team. Their serving framework is available at: https://github.com/forpublicai/chat.publicai.co.git

Together with the openly available Apertus model weights, this constitutes an end-to-end open dataset for LLM serving research: open model, open serving stack, and open trace.

## Contents

- **`apertus_trace.csv`** : one row per request, 10 raw columns (described below)
- **`trace_analysis.ipynb`** : Jupyter notebook that loads the CSV, computes derived columns, and reproduces the paper's figures
- **`README.md`**

## Dataset Schema

The CSV contains only the raw logged fields. All category-based columns used in the paper (e.g., `request_scale`, `ttft_category`) are derived deterministically in the notebook; they are not shipped in the CSV to keep the source data separate from the analysis decisions.

| Column | Type | Description |
|---|---|---|
| `request_id` | string | Anonymised unique identifier per request |
| `model` | string | `Apertus-8B` or `Apertus-70B` |
| `start_time` | datetime (UTC) | Request arrival timestamp |
| `end_time` | datetime (UTC) | Request completion timestamp |
| `completion_start_time` | datetime (UTC) | Time when the first output token was generated |
| `prompt_tokens` | int | Number of input tokens |
| `completion_tokens` | int | Number of output tokens |
| `total_tokens` | int | Sum of `prompt_tokens` and `completion_tokens` |
| `cache_hit` | string | LiteLLM cache status: `True`, `False`, or `None` |
| `conversation_turns` | int | Number of user messages in the conversation (see note below) |

### Derived columns computed in the notebook

These columns are computed from the raw fields using the thresholds in Table 2 of the paper:

| Derived column | Values | Computation |
|---|---|---|
| `total_duration_sec` | float | `end_time − start_time` |
| `time_to_first_token_sec` | float | `completion_start_time − start_time` |
| `generation_duration_sec` | float | `end_time − completion_start_time` |
| `tokens_per_sec` | float | `completion_tokens / generation_duration_sec` |
| `request_scale` | Tiny (<1K), Small (1K-5K), Medium (5K-10K), Large (>10K) | Based on `total_tokens` |
| `duration_scale` | Instant (<1s), Fast (1-5s), Moderate (5-10s), Slow (10-30s), Very slow (>30s) | Based on `total_duration_sec` |
| `ttft_category` | Instant start (<0.5s), Fast start (0.5-3s), Queued (3-10s), Heavily queued (>10s) | Based on `time_to_first_token_sec` |
| `prefill_fraction` | float in [0, 1] | `time_to_first_token_sec / total_duration_sec` |
| `phase_type` | Prefill-heavy, Decode-heavy | `prefill_fraction ≥ 0.5` → prefill-heavy |
| `is_anomalous` | bool | `total_duration_sec < 0.1` AND `total_tokens > 100` |

## Key Statistics

| Property | Value                    |
|---|--------------------------|
| Total requests | 337,155                  |
| Time span | Sep 2025 - Jan 2026      |
| Apertus-8B requests | 261,308 (77.7%)          |
| Apertus-70B requests | 75,847 (22.3%)           |
| Median duration | 5.0 s                    |
| P99 duration | 142.6 s                  |
| Missing days | 22 (logging disruptions) |

## Important Notes on the Data

- **Anonymisation.** User and request identifiers have been replaced. Timestamps are preserved to enable temporal analysis.
- **Cache timing.** LiteLLM-level caching was enabled on **December 18, 2025**. Before that date, `cache_hit` is always `False` or `None`. The overall `True` rate (5.3%) is diluted by the pre-caching months; in the December 18 - January window it is approximately 23%.
- **Conversation turns.** The `conversation_turns` field was populated starting **November 19, 2025**. Earlier requests show 0. 
- **Filtering provenance.** The raw logs contained 365,562 requests. We removed 140 failed requests (0 tokens) and 28,267 requests with invalid duration. The published trace contains the 337,155 remaining successful requests.

## Reproducing the Paper's Figures

```bash
# Install dependencies
pip install pandas numpy matplotlib scipy jupyter

# Launch the notebook
jupyter notebook trace_analysis.ipynb
```

The notebook is organised by figure number and is meant to be read alongside the paper. Each cell is self-contained and prints the numerical results that appear in the paper text.

## Citation

If you use this trace in your research, please cite:

```bibtex
coming soon...
```

## License

The trace data is released under [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/).

## Acknowledgements
We thank Public AI for providing access to the serving infrastructure and trace data, and for their work advancing idea of open AI for benefiting public interest. We also thank Héctor F. Satizábal for his valuable feedback and CSCS for providing information on the backend infrastructure. This work was supported by the Swiss National Science Foundation (SNSF) under Grant 10001932 (DEEP).

## Contact
If you have any questions, feedback, or encounter issues, feel free to reach out via [email](mailto:busra.karatayd@hes-so.ch).
