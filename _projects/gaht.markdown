---
title: "Genetic Algorithm for Hyperparameter Tuning"
status: "Completed"
tags: [Python, Optimization, Genetic Algorithms]
github: https://github.com/GabrielePintus/GMOO-Project
---

Hyperparameter optimization often involves a complex mix of discrete choices (e.g., number of layers, activation function) and continuous variables (e.g., learning rate, dropout). Standard search methods frequently struggle with this hybrid space: Grid Search fails to scale, while Bayesian Optimization, though powerful, can be sensitive to initialization and struggle with non-continuous variables.

## Problem formulation

Given a function $f(x)$ approximated by a neural network $\tilde{f}(x)$, let $g$ be the function mapping the hyperparameter space $\mathcal{H}$ to the validation loss. The goal is to find:

$$h^* = \arg\min_{h \in \mathcal{H}} g(h)$$

The model is a Multi-Layer Perceptron (MLP), and the hyperparameter space is:

$$\mathcal{H} = \mathbb{N}_{1,5} \times \mathbb{N}_{16,128} \times \mathbb{N}_{1,6} \times \mathbb{R}_{0,0.5}$$

That is: number of layers, hidden size, activation function (6 choices), and dropout rate.

## The GAHT implementation

I developed a Genetic Algorithm for Hyperparameter Tuning (GAHT) specifically designed to navigate this mixed discrete-continuous search space without requiring gradients or probabilistic surrogates.

The GA runs with a **population of 5** individuals for up to **20 generations**, with the following mapping:
- **Genotype**: vector of hyperparameters
- **Phenotype**: trained neural network
- **Fitness**: MSE on the validation set

**Operators:**

- **Tournament selection** (τ = 3, "mid pressure"): selects the best candidate from random subsets of size 3, balancing convergence speed with population diversity
- **Two-point crossover**: combines parent configurations at two indices to explore the search space more effectively than single-point methods
- **Adaptive mutations**: separate operators per variable type:
  - Number of layers: $n \leftarrow \text{clip}(n + \delta,\ 1,\ 5),\quad \delta \sim \text{Unif}(-2, 2)$
  - Neurons per layer: $n \leftarrow \text{clip}(n + \delta,\ 16,\ 128),\quad \delta \sim \text{Unif}(-16, 16)$
  - Dropout: $d \leftarrow \text{clip}(d + \epsilon,\ 0,\ 0.5),\quad \epsilon \sim \mathcal{N}(0, 0.15)$
  - Activation function: tree-based probabilistic mutation (see below)

### Activation function mutation

Mutating a categorical variable like the activation function is non-trivial. A naive approach — enumerate all functions, apply uniform mutation — implicitly assigns arbitrary distances between them, leading to inconsistent behavior.

Instead, I defined a hierarchy of activation functions as a tree and derived a distance function from it:

<div style="max-width: 640px; margin: 1.25em auto;">
<div class="mermaid">
%%{init: {
  'theme': 'base',
  'flowchart': {
    'curve': 'monotoneY',
    'nodeSpacing': 50,
    'rankSpacing': 60,
    'padding': 12
  },
  'themeVariables': {
    'fontSize': '16px',
    'fontFamily': 'ui-sans-serif, -apple-system, BlinkMacSystemFont, Segoe UI, sans-serif',
    'lineColor': '#6b7280',
    'textColor': '#111827',
    'primaryColor': '#e5e7eb',
    'primaryBorderColor': '#374151'
  }
}}%%
graph TD
    root(Activation Function)

    root --> unbounded(Unbounded)
    root --> bounded(Bounded)

    unbounded --> straight(Straight)
    unbounded --> round(Round)

    bounded --> tanh(tanh)
    bounded --> sigmoid(sigmoid)

    straight --> relu(relu)
    straight --> leakyrelu(leakyrelu)

    round --> elu(elu)
    round --> gelu(gelu)

    classDef root fill:#dbeafe,stroke:#1d4ed8,color:#0f172a,stroke-width:2px;
    classDef internal fill:#f3f4f6,stroke:#4b5563,color:#111827,stroke-width:1.5px;
    classDef leaf fill:#ecfdf5,stroke:#047857,color:#064e3b,stroke-width:1.5px;

    class root root;
    class unbounded,bounded,straight,round internal;
    class tanh,sigmoid,relu,leakyrelu,elu,gelu leaf;
</div>
</div>

The distance between two activation functions $l_1, l_2$ is the logarithm of the number of hops between them in the tree:

$$d_b(l_1, l_2) = \log_b(\text{hops}(l_1, l_2))$$

This induces a probability distribution:

$$\tilde{p}_b(l_1, l_2) = 2^{-d_b(l_1, l_2)}, \quad p_b(l_1, l_2) = \frac{\tilde{p}_b(l_1, l_2)}{\sum_{i \in L} \tilde{p}_b(l_1, i)}$$

The base $b$ controls the exploration-exploitation tradeoff: higher $b$ means more explorative mutations.

## Implementation

The algorithm is implemented in Python using **PyTorch Lightning** for model training and **Weights & Biases** for experiment tracking and visualization.

Since each fitness evaluation requires training a full neural network, assessments are expensive but mutually independent — making the algorithm inherently easy to parallelize. Two strategies are possible:
- **Population level**: train all individuals simultaneously, each using a fraction of available resources
- **Individual level**: train each individual sequentially with full resources (used here via PyTorch Lightning)

For large-scale settings where compute is geographically distributed, an **Island model** (multiple co-evolving populations) is a natural extension.

## Results

Both GAHT and standard Bayesian Optimization were benchmarked on a **Sinewave** dataset — a custom toy problem with 5 sinewaves as covariates and a random linear combination as target.

![Top 5 MSE comparison on Sinewave](/assets/images/gaht/sinewave-top5.png)

GA consistently outperforms BO across all top-5 models. The best GA configuration achieves MSE = $4.05 \times 10^{-4}$ vs BO's $5.25 \times 10^{-4}$: a **13.4% reduction in MSE** and **46.4% lower variance** across runs. The variance reduction is the standout result — the GA explores more broadly and converges more consistently than BO, which is sensitive to its surrogate model assumptions.

## Code

The full implementation and benchmark scripts are available on [GitHub](https://github.com/GabrielePintus/GMOO-Project).
