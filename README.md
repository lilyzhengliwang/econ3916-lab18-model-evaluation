# econ3916-lab18-model-evaluation

# Model Evaluation: The Curve, and What It Cannot Tell You

## Objective
I compared two classifiers on a simulated rare-outcome dataset using ROC and precision-recall curves, a paired bootstrap, and an expected-cost threshold, to see which differences in performance are real and which depend on a cost judgement the data cannot supply.

## Methodology
- Simulated 8,000 cases of a rare outcome and split them into training and test rows with a fixed seed (42). 11.8% of the 2,400 test rows were positive (283 rows).
- Fit a logistic regression and drew its ROC curve and precision-recall curve on the test rows.
- Fit a boosted-tree model as a second model and compared the two on ROC AUC and average precision.
- Ran a paired bootstrap over 500 resamples of the test rows to put a 95% interval on the AUC difference.
- Built an expected-cost curve in which a missed positive costs ten times a false alarm. I chose the cheapest threshold on the training rows using 5-fold cross-validated probabilities, then priced it once on the test rows.
- Repeated the threshold choice for both models at cost ratios of 2:1, 5:1, 10:1, 20:1, 30:1 and 50:1, and checked which model was cheaper at each ratio.
- Wrote `roc_point(model, X, y, t)`, which flags every row whose predicted probability is `t` or more and returns the false positive rate and true positive rate at that threshold.

## Key Findings
- The logistic regression had ROC AUC 0.8468 and average precision 0.4651. The boosted model had ROC AUC 0.8431 and average precision 0.4529.
- The AUC difference (logistic minus boosted) was +0.0038 on the full test set. The 95% bootstrap interval was [-0.0020, +0.0094], which includes zero, so I cannot say either model ranks better.
- The cheapest threshold on the training rows was 0.0905. It cost 0.4692 per case on the test rows, against 0.9437 at the default 0.5, so the default threshold cost about twice as much. The closed-form threshold 1/(1+10) = 0.0909 cost 0.4683, almost the same.
- Across the cost-ratio sweep, the logistic model was cheaper at 2:1, 10:1, 20:1, 30:1 and 50:1. The boosted model was cheaper only at 5:1 (0.3358 against 0.3417). Most of these gaps are small, and I did not put intervals on them, so I treat the 5:1 result as noise.
- At the chosen threshold of 0.0905, `roc_point()` gave a false positive rate of 0.296 and a true positive rate of 0.823.
- The best threshold and the choice between models both depend on the cost ratio. That number comes from the people who bear the cost of a miss and a false alarm, not from the data, so the metrics alone could not settle the choice.
