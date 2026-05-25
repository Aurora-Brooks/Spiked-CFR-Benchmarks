# IHDP-post and ACIC-post Benchmarks

This repository releases the processed **IHDP-post** and **ACIC-post** benchmarks introduced in our ICML 2026 paper:

**Spiked-CFR: Causal Representation Learning from LLMs via Wasserstein Projection Pursuit**

These benchmarks are text-based versions of the standard IHDP and ACIC semi-synthetic treatment-effect datasets. They are designed for evaluating causal effect estimation methods from natural-language inputs.

## Overview

IHDP-post and ACIC-post are generated from the original tabular IHDP and ACIC records using the prompts provided in the Appendix of our paper. We use **GPT-4o-mini** to convert each tabular instance into a natural-language post or clinical-style note, and then extract the variables used by our Spiked-CFR pipeline.

In our Spiked-CFR pipeline, these released files correspond to the output of the **LLM-based Extraction** stage:

- `post` is the GPT-4o-mini-generated natural-language post or clinical-style note, corresponding to the raw textual input.
- `x` is the extracted pre-treatment covariate text, used as input to the frozen LLM encoder.
- `treatment` is the treatment assignment.
- `y_factual` is the observed factual outcome.
- `y_counterfactual` is the held-out counterfactual outcome, provided only for evaluation.

The counterfactual outcome is not included in the generated post and should not be used as model input.

## Repository Structure

```text
Spiked-CFR/
  acic_post/
    acic_post.csv

  ihdp_post/
    ihdp_post.csv
```

## Dataset Statistics

| Dataset | Source | Number of Instances | Treatment | Outcome | Unit-Level Counterfactuals |
|---|---:|---:|---|---|---|
| IHDP-post | IHDP | 747 | Home visit / early intervention | IQ score | Yes |
| ACIC-post | ACIC | 4,802 | Simulated binary treatment | Synthetic continuous outcome | Yes |

## File Format

Both CSV files use UTF-8 encoding and contain the following columns:

| Column | Description |
|---|---|
| `unit_id` | Unique instance identifier within the dataset. |
| `x` | Extracted pre-treatment covariate text. This is the text input used by the frozen LLM encoder in our Spiked-CFR pipeline. It is intended to exclude treatment and outcome information. |
| `treatment` | Treatment assignment. In IHDP-post, this is represented as `Yes`/`No`; in ACIC-post, this is represented as `1`/`0`. |
| `y_factual` | Observed factual outcome. |
| `post` | GPT-4o-mini-generated natural-language post or clinical-style note from the original tabular record. |
| `y_counterfactual` | Held-out counterfactual outcome. This is provided for evaluation only and should not be used as model input. |

## Intended Use

These benchmarks can be used for research on:

- text-based causal effect estimation;
- causal representation learning from natural-language covariates;
- treatment-effect estimation from LLM representations;
- CATE / ITE evaluation with unit-level counterfactual ground truth;
- PEHE, ATE error, and related treatment-effect evaluation metrics.

In our Spiked-CFR pipeline, the `x` column is encoded by a frozen language model, while `treatment` and `y_factual` are used for treatment-effect learning. The `y_counterfactual` column is used only for evaluation.

## Evaluation Convention

For each instance, the factual and counterfactual outcomes can be converted into the two potential outcomes, `y0` and `y1`, according to the treatment assignment.

If `treatment = 1` or `treatment = Yes`:

```text
y1 = y_factual
y0 = y_counterfactual
```

If `treatment = 0` or `treatment = No`:

```text
y0 = y_factual
y1 = y_counterfactual
```

The individual treatment effect (ITE) is:

```text
tau = y1 - y0
```

For CATE evaluation, PEHE can be computed by comparing the estimated treatment effect with the ground-truth individual treatment effect `tau`.

## Loading Example

```python
import pandas as pd

ihdp = pd.read_csv("ihdp_post/ihdp_post.csv")
acic = pd.read_csv("acic_post/acic_post.csv")

print(ihdp.head())
print(acic.head())
```

Example conversion from factual/counterfactual outcomes to potential outcomes:

```python
import pandas as pd

def add_potential_outcomes(df):
    df = df.copy()

    t = df["treatment"]
    if t.dtype == object:
        treated = t.astype(str).str.lower().isin(["yes", "1", "true", "treated"])
    else:
        treated = t.astype(int) == 1

    df["y1"] = df["y_factual"].where(treated, df["y_counterfactual"])
    df["y0"] = df["y_counterfactual"].where(treated, df["y_factual"])
    df["tau"] = df["y1"] - df["y0"]
    return df

ihdp = add_potential_outcomes(ihdp)
acic = add_potential_outcomes(acic)
```

## Citation

If you use IHDP-post or ACIC-post, please cite our ICML 2026 paper:

```bibtex
@inproceedings{wang2026spikedcfr,
  title={Spiked-CFR: Causal Representation Learning from LLMs via Wasserstein Projection Pursuit},
  author={Wang, Fan and Yue, Hengyu and Yu, Bowen and Liu, Weiming and Yang, Zongxin and Zhang, Xuyun and Zheng, Xiaolin and Chen, Chaochao and Deng, Shuiguang},
  booktitle={Proceedings of the 43rd International Conference on Machine Learning},
  year={2026}
}
```

Please also cite the original IHDP and ACIC sources:

```bibtex
@article{hill2011bayesian,
  title={Bayesian Nonparametric Modeling for Causal Inference},
  author={Hill, Jennifer L.},
  journal={Journal of Computational and Graphical Statistics},
  volume={20},
  number={1},
  pages={217--240},
  year={2011},
  doi={10.1198/jcgs.2010.08162}
}
```

```bibtex
@article{dorie2019automated,
  title={Automated versus Do-It-Yourself Methods for Causal Inference: Lessons Learned from a Data Analysis Competition},
  author={Dorie, Vincent and Hill, Jennifer and Shalit, Uri and Scott, Marc and Cervone, Dan},
  journal={Statistical Science},
  volume={34},
  number={1},
  pages={43--68},
  year={2019},
  doi={10.1214/18-STS667}
}
```

## License

The processed IHDP-post and ACIC-post benchmark files are released under the Creative Commons Attribution-NonCommercial 4.0 International License (CC BY-NC 4.0). See `LICENSE` for details.

This license applies only to the processed benchmark files released in this repository. Users are responsible for complying with the terms and conditions of the original IHDP and ACIC datasets.

## Limitations

These benchmarks are semi-synthetic and are intended for research evaluation. They should not be interpreted as real clinical evidence or used for medical decision-making.

The generated text is produced by GPT-4o-mini from structured records and may reflect artifacts of the prompting process. Researchers should consider this when interpreting results obtained on these benchmarks. 
