# Case 1 Analysis

## Q1 — Generate

The analysis compares three ways to rank customers for a retention contact list. The **contract rule** is the simple baseline: it calculates the training-data churn rate for each `Contract` category and assigns that rate to customers in the same category. **Logistic regression** combines the seven inputs into a churn probability after standardizing the numeric variables and one-hot encoding the categorical variables. The supplied model uses `C=1.0`. **Boosted trees** combine 100 shallow decision trees, each with maximum depth 2, using a learning rate of 0.1. Each successive tree helps correct patterns not captured by the preceding trees.

The seven inputs shared by logistic regression and boosted trees are `tenure`, `MonthlyCharges`, `TotalCharges`, `Contract`, `InternetService`, `PaperlessBilling`, and `PaymentMethod`. The first three are numeric and the remaining four are categorical. The contract-rule baseline uses `Contract` alone. `customerID` is an identifier, and `Churn` is the outcome; neither is used as a predictor.

The commands were run in the required order:

```bash
python VD1_analysis.py compare --csv churn.csv --out outputs
```

After reviewing the validation results and recording boosted trees as the choice:

```bash
python VD1_analysis.py evaluate --csv churn.csv --out outputs --choice trees
```

The retained output files are `validation.csv`, `split_rows.csv`, `compare_run.json`, `test_metrics.csv`, `intervals.csv`, `probability_groups.csv`, `scenarios.csv`, `test_predictions.csv`, and `evaluate_run.json`. The run records preserve the source-file fingerprint, Python and package versions, split seed, features, partition sizes, cleaning rule, and recorded method choice. Reproduction requires Python 3.11; the completed runs used Python 3.11.3, NumPy 1.24.2, pandas 2.0.0, and scikit-learn 1.2.2. The instructor-supplied `churn.csv` should be placed beside the script and should not be publicly uploaded unless the course requires it.

Codex helped read the assignment files, verify the data, establish the required environment, run the unchanged supplied script, audit the saved outputs, explain the methods and metrics, and organize the Q1–Q4 evidence. The AI transcript and `AI_USE.md` document that assistance, including unsuccessful setup attempts and the staged validation-choice process. I personally checked the `TotalCharges` treatment and verified that each source row appears in exactly one partition and that fitting uses training rows only.

The handout identifies the data as the public Telco customer-churn example distributed through the [scikit-learn/churn-prediction dataset mirror](https://huggingface.co/datasets/scikit-learn/churn-prediction). The mirror declares CC BY 4.0; the handout records that this was checked September 9, 2026. This is a classroom example, not verified Summit Telecom data, and `Churn = Yes` means that the customer left.

## Q2 — Validate the analysis

The assigned extract has 7,043 rows and 21 columns. It contains 5,174 `No` and 1,869 `Yes` churn outcomes. All 7,043 customer IDs are present and unique. The only raw blanks are 11 whitespace-only `TotalCharges` entries, all at `tenure = 0`. The script converts `TotalCharges` to numeric, confirms that those 11 blanks are the only conversion failures, fills them with `0.0`, and removes no rows. It then checks that every analysis input is nonmissing and every numeric value is finite.

The split is stratified and uses random seed 0. The first split assigns 60% to training and holds out 40%; the second divides that remainder equally between validation and final test. Rounding to whole rows produces 4,225 training rows, 1,409 validation rows, and 1,409 final-test rows. An audit of `split_rows.csv` found 7,043 records with 7,043 unique source-row numbers covering 0 through 7042, no duplicates, no missing rows, and no unexpected partition labels.

Fitting is restricted to training rows. The contract-category rates and fallback rate use `y[train]` and `df.iloc[train]`. For logistic regression and boosted trees, the scaler, one-hot encoder, and model are placed in one pipeline, and `pipe.fit` receives only `df.iloc[train]` and `y[train]`. Validation and test rows are passed later to the fitted prediction functions; they do not teach the preprocessing or models.

The complete final-test contact-list results are:

| Method | Final-test AUC | Contact-list size | Top-20% observed churn rate | Overall test churn rate |
|---|---:|---:|---:|---:|
| Contract rule | 0.7372652354749543 | 281 | 0.398576512455516 | 0.2654364797728886 |
| Logistic regression | 0.8472073677956031 | 281 | 0.693950177935943 | 0.2654364797728886 |
| Boosted trees | 0.8496654524787517 | 281 | 0.693950177935943 | 0.2654364797728886 |

AUC measures overall ranking: it is the probability that a randomly selected churner receives a higher score than a randomly selected non-churner, with ties receiving half credit. The top-20% observed churn rate answers the operational contact-list question more directly because it reports the historical churn concentration among the 281 customers the team would actually contact. Both multivariable models concentrated substantially more churn in that list than the contract rule. Neither AUC nor list churn establishes that contacting a customer will prevent churn.

## Q3 — Assess uncertainty and value

The supplied 95% bootstrap intervals for each fixed model are:

| Method | Final-test AUC | 95% bootstrap interval |
|---|---:|---|
| Contract rule | 0.7372652354749543 | [0.7162586607994378, 0.7557311497055941] |
| Logistic regression | 0.8472073677956031 | [0.8254417044753704, 0.8700303965284283] |
| Boosted trees | 0.8496654524787517 | [0.8282673461945358, 0.8731642295413213] |

The paired differences are:

| Comparison | AUC difference | 95% paired interval | Includes zero? |
|---|---:|---|---|
| Logistic minus contract | 0.10994213232064887 | [0.09315303703783528, 0.12850513058579374] | No |
| Trees minus contract | 0.11240021700379743 | [0.09589134628060791, 0.13111680606778364] | No |
| Trees minus logistic | 0.0024580846831485648 | [-0.004874855754479845, 0.010192741827438694] | Yes |

For each of 1,000 bootstrap repetitions, the code samples 1,409 test rows with replacement and evaluates every method on the same sampled indices. It subtracts the two AUCs within each shared sample, preserving the paired comparison, and uses the 2.5th and 97.5th percentiles as interval endpoints. All 1,000 draws were valid. The positive model-versus-contract intervals support better ranking by both models in this test sample. The trees-minus-logistic interval includes zero, so the direction of their small difference remains uncertain; including zero does not prove that the methods are equal.

These intervals hold the fitted predictions fixed. They do not include uncertainty from retraining on another sample, future customer or market changes, another company, model selection, or the causal effectiveness of an offer. They also assume row independence; unmodeled household, geographic, temporal, or other dependence could make the intervals too narrow. The three comparisons are exploratory and have no multiple-comparison adjustment.

For boosted trees, the overall mean predicted probability was 0.2704734491434011 and the observed test churn rate was 0.2654364797728886, an overprediction of 0.0050369693705125. Within its 281-customer top-20% list, the mean prediction was 0.6478654375314629 while observed churn was 0.693950177935943, an underprediction of 0.0460847404044801. A close overall average therefore did not eliminate mismatch in the selected high-risk list.

| Boosted-tree probability group | n | Mean prediction | Observed churn rate |
|---|---:|---:|---:|
| 0.0 to 0.2 exclusive | 724 | 0.07964254964211705 | 0.0718232044198895 |
| 0.2 to 0.4 exclusive | 272 | 0.2977914418379485 | 0.2536764705882353 |
| 0.4 to 0.6 exclusive | 236 | 0.49625647107850734 | 0.5211864406779662 |
| 0.6 to 0.8 exclusive | 142 | 0.6768545957787547 | 0.6901408450704225 |
| 0.8 to 1.0 inclusive | 35 | 0.834478055632186 | 0.9142857142857143 |

The 0.2–0.4 group overpredicted churn by 0.0441149712497132. The 0.8–1.0 group underpredicted by 0.0798076586535283, but it contains only 35 customers and is too small for a confident conclusion. No group-level confidence intervals were supplied.

The value analysis uses the fixed boosted-tree list and its historical observed churn rate, `r = 0.693950177935943`. The fictional assumptions are $66 net value per additional retention, $6.20 cost for every contact, and save rates of 10%, 15%, and 20%. The same 281 customers remain selected in every scenario.

| Assumed save rate | Historical list churn rate | Hypothetical net value per 1,000 contacts | Break-even save rate |
|---:|---:|---:|---:|
| 0.10 | 0.693950177935943 | -$1,619.9288256227762 | 0.1353690753690754 |
| 0.15 | 0.693950177935943 | $670.1067615658358 | 0.1353690753690754 |
| 0.20 | 0.693950177935943 | $2,960.142348754448 | 0.1353690753690754 |

**Independently checked calculation — 15% scenario:**

```text
r × s = 0.693950177935943 × 0.15 = 0.10409252669039145
Value per contact = 0.10409252669039145 × $66 = $6.8701067615658357
Net per contact = $6.8701067615658357 − $6.20 = $0.6701067615658357
Net per 1,000 = 1,000 × $0.6701067615658357 = $670.1067615658357
```

The saved value is $670.1067615658358; the final decimal-place difference is binary floating-point representation. The break-even calculation is `$6.20 / (0.693950177935943 × $66) = 0.1353690753690754`, or **13.5369%**. These are hypothetical planning values, not measured profit. The dataset observes historical churn but contains no randomized offer, contact indicator, or counterfactual outcome. It cannot establish whether the offer causes customers to stay, and it does not estimate the save rate.

## Q4 — Explain your choice

### Validation-stage choice record

Recorded on September 27, 2026, before viewing the final-test results.

**Selected method:** Trees (boosted trees)

The validation AUC for this method was 0.8455578806. The other validation AUCs were 0.8384174223 for logistic regression and 0.7427239143 for the contract rule.

My reason for carrying boosted trees forward is that it had the highest validation AUC of the three methods. It also produced the highest observed churn rate in its top-20% validation contact list at 0.6512455516, compared with 0.6120996441 for logistic regression and 0.4163701068 for the contract rule. I recognize that its AUC advantage over logistic regression was small—about 0.00714—but I am following the validation results and selecting the numerical leader.

The original validation-stage choice remains unchanged after viewing the final test. In the table, **pilot candidate** means the selected method for possible randomized testing, **benchmark** means the simple comparator retained for evaluation, and **deferred option** means a credible alternative not selected for the current pilot.

| Method | Final-test AUC | Delta AUC vs. contract | 95% interval | Carry forward? | Reason |
|---|---:|---:|---|---|---|
| Contract rule | 0.7372652354749543 | — | AUC interval: [0.7162586607994378, 0.7557311497055941] | Benchmark | Required simple baseline; it had the lowest AUC and a top-20% observed churn rate of 0.398576512455516. |
| Logistic regression | 0.8472073677956031 | 0.10994213232064887 | Paired delta interval: [0.09315303703783528, 0.12850513058579374] | Deferred option | It substantially exceeded the contract rule and produced the same top-20% observed churn rate as trees, but it was not the recorded validation choice. |
| Boosted trees | 0.8496654524787517 | 0.11240021700379743 | Paired delta interval: [0.09589134628060791, 0.13111680606778364] | Pilot candidate | It preserves the recorded choice, had the highest validation and final-test AUCs, and its paired advantage over the contract rule was entirely positive. |

Evidence that could change this choice includes a larger or later sample showing that logistic regression ranks as well or better, especially because the final trees-minus-logistic difference was only 0.0024580846831485648 and its interval crossed zero. The two methods also had the same final-test top-20% observed churn rate of 0.693950177935943, so operational simplicity or stability could favor logistic regression. For the broader campaign decision, evidence of weaker future list concentration, persistent calibration problems, higher actual costs, or an experimentally measured save rate below the 13.5369% break-even threshold would weaken the case. A randomized pilot that measures incremental retention and total costs, with a save-rate interval above break-even, would strengthen it.
