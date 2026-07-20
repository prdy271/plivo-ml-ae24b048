# RUNLOG

## Run 1

### Hypothesis

The baseline model was likely underpowered and lacked several standard transformer training practices. Increasing model capacity while remaining under the 2M parameter limit, together with improved optimisation, should improve generalisation and reduce bits-per-byte (BPB).

### Changes

Optimisation
- Enabled weight tying (`tie_weights=True`).
- Reduced parameter initialisation standard deviation from `0.05` to `0.02`.
- Added cosine learning-rate scheduling with warmup.
- Added gradient clipping (`max_norm=1.0`).

Architecture
- Increased context length:
  - `block_size: 128 -> 256`
- Increased model capacity while remaining below the parameter budget:
  - `n_layer: 4 -> 6`
  - `n_head: 4 -> 5`
  - `n_embd: 128 -> 160`

### Results

Parameters: 1,937,920

Final training loss: 1.9535

Development BPB:

2.6716

### Conclusion

This experiment substantially improved optimisation stability while increasing model capacity.

Increasing the embedding dimension and number of transformer blocks allowed the model to represent longer-range dependencies without exceeding the assignment parameter budget.

Weight tying reduced redundant parameters between the embedding matrix and output projection, allowing more parameters to be allocated to the transformer itself.

Gradient clipping prevented occasional unstable updates, while cosine learning-rate decay provided larger updates early in training and smaller updates during convergence.

This run established a significantly stronger baseline for subsequent experiments.

## Run 2

### Hypothesis

The first run still used relatively small batches and no regularisation. Increasing the batch size should provide a more accurate gradient estimate, while moderate dropout should improve generalisation. Increasing warmup should further stabilise optimisation during the early stages of training.

### Changes

- Batch size:
  - 8 -> 16

- Warmup:
  - 100 -> 200 steps

- Dropout:
  - 0 -> 0.05

### Results

Parameters: 1,937,920

Final training loss:

1.9113

Development BPB:

2.5578

### Comparison

Training loss:

1.9535 -> 1.9113

Development BPB:

2.6716 -> 2.5578

### Conclusion

This was the best-performing configuration.

The larger batch size reduced gradient variance, producing smoother optimisation and consistently lower training loss throughout training.

Increasing the warmup period prevented excessively aggressive parameter updates during the earliest optimisation steps, allowing the cosine learning-rate schedule to transition more smoothly into full learning rate.

Adding a small amount of dropout improved generalisation without noticeably slowing convergence. Unlike heavy regularisation, a dropout rate of 0.05 was sufficient to reduce overfitting while still allowing the model to fit the training distribution effectively.

The simultaneous reduction in both training loss and development BPB indicates that these modifications improved both optimisation quality and generalisation.

## Run 3

### Hypothesis

After improving optimisation, the next objective was to investigate whether architectural refinements inspired by modern GPT implementations would improve language modelling performance.

### Changes

Initialisation

- Replaced the original initialisation routine with GPT-style initialisation.
- Scaled residual projection weights by

0.02 / sqrt(2 * n_layer)

Activation

- Replaced

GELU()

with

GELU(approximate="tanh")

Attention

- Removed biases from

qkv projection

and

output projection

### Results

Parameters:

1,934,080

Final training loss:

1.9354

Development BPB:

2.6174

### Comparison

Training loss:

1.9113 -> 1.9354

Development BPB:

2.5578 -> 2.6174

### Conclusion

The modifications degraded performance.

Although these techniques are commonly used in modern large language models, they appear to be less effective in this small-scale training regime.

Removing biases likely reduced the model's flexibility during the limited 2000 optimisation steps. Large production models are typically trained for billions of tokens, where bias removal provides negligible benefit, whereas smaller models can still benefit from the additional parameters.

The custom initialisation also altered optimisation dynamics relative to the previous run. Since the model was already converging well, changing the initialisation provided no measurable advantage.

The tanh approximation of GELU was expected to have only a negligible numerical effect and was unlikely to explain the observed degradation by itself.

Overall, this experiment demonstrated that architectural techniques successful at large scale do not necessarily transfer directly to small models trained under strict optimisation budgets.

## Run 4

### Hypothesis

The previous experiments suggested that optimisation was already strong. The final experiment therefore attempted to improve language modelling capacity by increasing the available context length while simultaneously reducing the learning rate for potentially better convergence.

### Changes

- Reverted to the Run 2 architecture.
- Retained GELU(tanh).
- Increased context length:

256 -> 384

- Reduced learning rate:

3e-4 -> 2e-4

### Results

Parameters:

1,937,920

Final training loss:

2.0209

Development BPB:

2.7267

### Comparison

Training loss:

1.9113 -> 2.0209

Development BPB:

2.5578 -> 2.7267

### Conclusion

This experiment performed substantially worse than Run 2.

Increasing the context length significantly increased the optimisation difficulty while keeping the number of optimisation steps fixed at 2000. The model therefore spent much of training learning longer contexts instead of sufficiently fitting shorter-range dependencies.

Reducing the learning rate simultaneously slowed optimisation, making the reduced convergence even more pronounced.

Although larger context windows are generally beneficial when models are trained for sufficiently long durations, under the fixed training budget imposed by this assignment they prevented the model from reaching the same optimisation quality as Run 2.

This experiment illustrates that increasing model capability alone does not guarantee better performance when optimisation resources are limited.