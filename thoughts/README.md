# Fragment Thoughts

1. The Best Regularizer is Data
  - Quick start the business with clearly defined ideas, and get massive real-world data; *they* are the Best Regularizer for our deep learning model.
    - Regularizer: they will solve overfitting and imbalance issues, brining other applications like self-supervised learning.
  - The real-world data is the asset for the business.
***
2. Softmax vs. Sigmoid in label space perspective
- The softmax is a operation that expresses a given logits as probability from an overall perspective
- The sigmoid is a operation that expresses each given logits as a probability based on the logit value itself.
  - At this point, the usage of the sigmoid is wider than softmax, as sigmoid allows more fluid handling of unrelated label spaces.
  - Ex1)
    - classifying one of the 10 states for the patience → Softmax
    - classifying several actions the patience is performing → Sigmoid
      - The patient's behavior can be identified from a multi-label perspective.
***
