# NOTES

The best-performing configuration was obtained in Run 2 with a development BPB of 2.5578.

The final model used six transformer layers, five attention heads, an embedding dimension of 160, a context length of 256, weight tying, cosine learning-rate decay with a 200-step warmup, AdamW optimisation, gradient clipping at 1.0, dropout of 0.05 and a batch size of 16.

Increasing the model capacity while remaining below the 2M parameter limit provided the largest improvement over the starter implementation.

Increasing the batch size reduced gradient variance and produced smoother optimisation.

A longer warmup improved optimisation stability during the initial stages of training.

A small amount of dropout improved generalisation without preventing convergence.

Later experiments showed that GPT-style initialisation, removing attention biases and increasing the context length did not improve performance under the limited 2000-step training budget.

These results suggest that optimisation quality and appropriate model capacity were more important than adopting architectural techniques designed for much larger language models.