# Overview

- Machine learning builds systems which learn patterns from data instead of receiving every rule explicitly

- Deep learning is machine learning using neural networks with multiple learned layers

- A model is a parameterized function. Training adjusts parameters to reduce an objective on examples

- Main learning settings:

    Supervised: Learn from input-label pairs

    Unsupervised/self-supervised: Learn structure or prediction targets from the data itself

    Reinforcement learning: Learn actions from rewards in an environment

- Main workflow:

    Define problem -> Collect data -> Split data -> Build baseline -> Train -> Validate -> Tune -> Test once -> Deploy -> Monitor

- A model is only useful relative to a metric, dataset distribution, latency/cost limit, and failure requirement

# Mathematical Foundations

- A scalar is one number, a vector is a one-dimensional collection, a matrix is two-dimensional, and a tensor generalizes to more dimensions

- Shape describes tensor dimensions, such as `(batch, sequence, embedding)`

- Matrix multiplication combines linear transformations. Element-wise multiplication multiplies matching positions

- A neuron computes a weighted sum plus bias followed by an activation:

    `y = activation(Wx + b)`

- Common activation functions:

    ReLU: `max(0, x)`, simple and common in hidden layers

    GELU: Smooth activation common in transformers

    Sigmoid: Maps to `(0, 1)`, useful for independent binary outputs

    Tanh: Maps to `(-1, 1)`

    Softmax: Converts a vector of logits into a categorical probability distribution

- A loss function measures prediction error

    Mean Squared Error: Regression

    Binary Cross-Entropy: Binary/multi-label classification

    Cross-Entropy: Multi-class classification and language modeling

- A gradient measures how a small parameter change affects the loss

- Backpropagation applies the chain rule backward through the computation graph to calculate gradients

- An optimizer uses gradients to update parameters

    Gradient descent: `parameter = parameter - learning_rate * gradient`

    SGD: Updates from a batch rather than the complete dataset

    Adam/AdamW: Adaptive momentum-based optimizers commonly used in deep learning

# Data Splits and Generalization

- Training set: Used to update model parameters

- Validation set: Used to choose architecture, hyperparameters, and stopping point

- Test set: Used once for an unbiased final estimate

- Data leakage occurs when training receives information which would not be available at real prediction time

- Split related users, documents, time periods, or duplicates together when independent random rows would leak information

- Underfitting means the model cannot learn the training pattern. Overfitting means it learns training-specific details which do not generalize

- Regularization techniques include weight decay, dropout, data augmentation, early stopping, smaller models, and more representative data

- Distribution shift means production data differs from training/evaluation data

- Class imbalance can make accuracy misleading. Use stratification, class weighting, sampling, and class-sensitive metrics as appropriate

# Training Vocabulary

- Parameter: Value learned by the model

- Hyperparameter: Setting chosen outside normal gradient training, such as learning rate, layer count, or batch size

- Sample/example: One training item

- Feature: Input value

- Label/target: Expected output

- Batch: Group processed in one forward/backward step

- Epoch: One pass over the training set

- Step/iteration: One optimizer update

- Logit: Raw model score before probability conversion

- Checkpoint: Saved model and training state

- Learning-rate scheduler: Changes learning rate over training

- Gradient accumulation: Combines gradients from several small batches before updating

- Gradient clipping: Limits unusually large gradients

# PyTorch Fundamentals

- PyTorch is a tensor and deep-learning library with automatic differentiation, neural network modules, data tools, accelerators, compilation, and distributed training

    ```python
    import torch

    x = torch.tensor([[1.0, 2.0], [3.0, 4.0]])
    weights = torch.randn(2, 1, requires_grad=True)
    prediction = x @ weights
    loss = prediction.pow(2).mean()
    loss.backward()

    print(weights.grad)
    ```

- Important tensor attributes:

    `.shape`: Dimensions

    `.dtype`: Numeric representation such as `float32` or `int64`

    `.device`: CPU, CUDA GPU, MPS, or another accelerator

    `.requires_grad`: Whether autograd records operations for gradients

- Move tensors and models to the same device with `.to(device)`

- Operations ending in `_` mutate in place, such as `zero_()`

- Broadcasting expands compatible singleton dimensions without manually copying data

- Views such as `reshape`, `view`, `transpose`, `permute`, `squeeze`, and `unsqueeze` change how dimensions are interpreted

- `detach()` removes a tensor from gradient tracking. `clone()` copies values. `torch.no_grad()` disables gradient recording in a scope

# PyTorch Models

- Models subclass `torch.nn.Module`

    ```python
    import torch
    from torch import nn


    class Classifier(nn.Module):
        def __init__(self, input_size: int, hidden_size: int, classes: int):
            super().__init__()
            self.network = nn.Sequential(
                nn.Linear(input_size, hidden_size),
                nn.ReLU(),
                nn.Dropout(0.2),
                nn.Linear(hidden_size, classes),
            )

        def forward(self, x: torch.Tensor) -> torch.Tensor:
            return self.network(x)
    ```

- Assigning layers as attributes registers their parameters

- `forward` defines the computation. Call `model(x)` instead of calling `forward` directly

- Common modules include `Linear`, `Embedding`, `Conv2d`, `LSTM`, `LayerNorm`, `BatchNorm`, `Dropout`, `MultiheadAttention`, and `TransformerEncoder`

- `state_dict()` contains learned parameters and persistent buffers

- `model.train()` enables training behavior such as dropout. `model.eval()` enables evaluation behavior. Neither disables gradients by itself

# Datasets and DataLoaders

- A map-style `Dataset` implements `__len__` and `__getitem__`

    ```python
    from torch.utils.data import Dataset, DataLoader


    class TextDataset(Dataset):
        def __init__(self, examples):
            self.examples = examples

        def __len__(self):
            return len(self.examples)

        def __getitem__(self, index):
            return self.examples[index]


    loader = DataLoader(
        TextDataset(examples),
        batch_size=32,
        shuffle=True,
        num_workers=0,
    )
    ```

- `DataLoader` batches, shuffles, loads in workers, and uses a collate function to combine examples

- Variable-length sequences are normally padded with an attention mask or grouped by similar lengths

- Normalize inputs using statistics computed from training data only

- Seed Python, NumPy, and PyTorch when reproducibility matters, while understanding some accelerator operations may remain nondeterministic

# Training Loop

    ```python
    model = Classifier(input_size=100, hidden_size=64, classes=4).to(device)
    optimizer = torch.optim.AdamW(model.parameters(), lr=3e-4)
    criterion = torch.nn.CrossEntropyLoss()

    for epoch in range(epochs):
        model.train()

        for features, labels in train_loader:
            features = features.to(device)
            labels = labels.to(device)

            optimizer.zero_grad()
            logits = model(features)
            loss = criterion(logits, labels)
            loss.backward()
            torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
            optimizer.step()

        model.eval()
        with torch.inference_mode():
            validate(model, validation_loader)
    ```

- Correct basic order:

    Clear gradients -> Forward pass -> Compute loss -> Backward pass -> Optional clipping -> Optimizer step

- Do not apply softmax before `CrossEntropyLoss`; it expects raw logits

- Validation should not update parameters and should normally use `torch.inference_mode()`

- Track training and validation loss separately. Save the best validation checkpoint, not automatically the final epoch

# Checkpointing and Inference

    ```python
    torch.save({
        "model": model.state_dict(),
        "optimizer": optimizer.state_dict(),
        "epoch": epoch,
        "config": config,
    }, "checkpoint.pt")

    checkpoint = torch.load("checkpoint.pt", map_location=device, weights_only=True)
    model.load_state_dict(checkpoint["model"])
    model.eval()
    ```

- Save architecture/configuration, tokenizer version, model weights, optimizer/scheduler state, epoch/step, metrics, and random-state information needed to resume

- Load untrusted pickle-based checkpoints only with safe loading practices. Model files can be a code-execution boundary

- Inference should validate inputs, use evaluation mode, disable gradients, batch where useful, and measure latency and memory

- `torch.compile` may optimize model execution, but behavior and speedups depend on the model, shapes, and environment. Benchmark the actual workload

# Neural Network Architectures

- Feedforward/MLP networks apply fully connected layers and are common for tabular features and classifier heads

- Convolutional Neural Networks use local shared filters and are common for images and spatial signals

- Recurrent networks such as LSTM and GRU process sequences step by step and retain a state

- Transformers use attention to process token relationships with high parallelism during training

- Encoder-only transformers produce contextual representations and are common for classification and embeddings

- Decoder-only transformers predict following tokens and are common for generative language models

- Encoder-decoder transformers map one sequence to another and are common for translation and structured generation

# Tokenization and BPE

- Language models operate on token IDs rather than raw text

- Character tokenization has a small vocabulary but long sequences

- Word tokenization creates unknown-word and huge-vocabulary problems

- Subword tokenization balances vocabulary size and sequence length

- Byte Pair Encoding (BPE) starts from small units and repeatedly merges the most useful adjacent pair

- Basic BPE training:

    1. Normalize the training text

    2. Represent words as initial byte/character symbols

    3. Count adjacent symbol pairs

    4. Merge the most frequent pair

    5. Repeat until reaching the target vocabulary

    6. Save merge order and token-to-ID vocabulary

- BPE encoding applies learned merges in rank order. Decoding maps token IDs back to byte/text sequences

- Special tokens may represent padding, unknown values, beginning/end, masking, or application roles

- Tokenizer and model are one artifact contract. Changing vocabulary or token IDs without adjusting embeddings invalidates a checkpoint

- Evaluate compression, unknown behavior, multilingual coverage, whitespace/code handling, and round-trip correctness

# Transformer Architecture

- A decoder-only transformer pipeline is:

    Token IDs -> Token embeddings + position information -> Repeated masked transformer blocks -> Final normalization -> Vocabulary logits

- Self-attention projects input representations into queries, keys, and values

    `Attention(Q, K, V) = softmax(QK^T / sqrt(d_k))V`

- Queries ask what each position needs, keys describe what positions offer, and values contain information to combine

- Multi-head attention learns several relationship subspaces in parallel and concatenates their results

- A causal mask prevents a decoder token from attending to future tokens during next-token training

- Padding masks prevent attention to padding positions

- A transformer block normally includes attention, feedforward/MLP layers, residual connections, layer normalization, and dropout

- Residual connections provide a direct path around transformations and help optimization

- Positional information is needed because attention alone does not encode order. Approaches include learned positions, sinusoidal positions, and rotary position embeddings

- The final vocabulary logits have shape `(batch, sequence, vocabulary_size)`

- Next-token loss compares the logits at one position to the token at the following position

# Language Model Training and Generation

- For a token sequence, training inputs omit the final token and labels omit the first token

    ```python
    inputs = tokens[:, :-1]
    targets = tokens[:, 1:]
    logits = model(inputs)
    loss = criterion(
        logits.reshape(-1, logits.size(-1)),
        targets.reshape(-1),
    )
    ```

- Perplexity is `exp(cross_entropy_loss)` and represents how uncertain the model is on average under the tokenization and dataset

- Compare perplexity only when evaluation data and tokenization are compatible

- Generation loop:

    Encode prompt -> Run model -> Select next-token distribution -> Choose token -> Append -> Repeat until stop condition

- Sampling controls:

    Greedy: Always highest probability

    Temperature: Flattens or sharpens probabilities

    Top-k: Sample among the k most likely tokens

    Top-p/nucleus: Sample from the smallest set reaching a probability threshold

    Repetition penalty: Reduces repeated token preference

- Cache prior attention keys/values during autoregressive generation to avoid recomputing the complete prefix

- Stop on end token, maximum length, or an application-specific sequence

# Embeddings

- An embedding model maps content into a fixed-size vector

- Text embeddings support semantic search, clustering, recommendations, duplicate detection, classification features, and retrieval

- Always use the same compatible embedding model and preprocessing for indexed content and queries

- Chunking choices affect retrieval:

    Chunk size: Enough context without mixing unrelated topics

    Overlap: Preserves boundary information but duplicates retrieval/storage

    Structure: Headings, paragraphs, code blocks, tables, and document boundaries

    Metadata: Source, position, owner, permissions, version

- Normalize embeddings when required by the metric/model

- Embedding model changes normally require complete re-embedding or a separately versioned index

- Do not infer that close vectors are true, safe, authorized, or causally related. They are model-based similarity signals

# Vector Search and Retrieval

- Retrieval begins with a query embedding and returns nearest stored embeddings

- `top_k` controls how many candidates return. A larger value may improve recall but increases noise, prompt size, latency, and cost

- Metadata filtering enforces scope and narrows the search space

- Dense semantic retrieval finds similar meaning. Sparse/BM25 retrieval finds shared lexical signals

- Hybrid retrieval combines semantic and lexical strengths

- Rerankers score query-document pairs more precisely after inexpensive first-stage retrieval

- Maximum Marginal Relevance balances query similarity against diversity among selected chunks

- Query rewriting, decomposition, hypothetical documents, and multi-query retrieval may improve difficult searches but add latency and new failure modes

# pgvector

- pgvector is a PostgreSQL extension which stores and searches vectors next to relational data

    ```SQL
    CREATE EXTENSION IF NOT EXISTS vector;

    CREATE TABLE document_chunks (
        id bigserial PRIMARY KEY,
        user_id uuid NOT NULL,
        document_id uuid NOT NULL,
        content text NOT NULL,
        content_hash text NOT NULL,
        embedding vector(1536) NOT NULL,
        UNIQUE (document_id, content_hash)
    );
    ```

- Distance operators:

    `<->`: L2 distance

    `<#>`: Negative inner product

    `<=>`: Cosine distance

    `<+>`: L1 distance

    ```SQL
    SELECT id, content, 1 - (embedding <=> :query_embedding) AS similarity
    FROM document_chunks
    WHERE user_id = :authenticated_user_id
    ORDER BY embedding <=> :query_embedding
    LIMIT 8;
    ```

- Exact search provides perfect recall without an approximate index

- HNSW provides a strong speed/recall tradeoff with higher memory and slower index creation

    ```SQL
    CREATE INDEX document_chunks_embedding_hnsw
    ON document_chunks
    USING hnsw (embedding vector_cosine_ops);
    ```

- IVFFlat partitions vectors into lists, builds faster and uses less memory, but requires representative existing data and tuning

- Create an index using the operator class matching the query distance

- Approximate search trades recall for latency. Measure it against exact results on representative queries

- Relational filters and vector indexes interact. Composite relational indexes, query planning, iterative scans, partitioning, and tenant layout may affect filtered recall and speed

- Apply Row Level Security or server-side ownership conditions so similarity search never crosses users

# Retrieval-Augmented Generation (RAG)

- RAG retrieves external context and includes it in a model request so generation can use application-specific information

- Main stages:

    Ingestion -> Retrieval -> Prompt assembly -> Generation -> Citation/validation -> Evaluation

- Ingestion:

    Parse -> Normalize -> Chunk -> Hash -> Embed -> Store vector + source metadata -> Remove obsolete chunks

- Retrieval:

    Authenticate -> Scope -> Embed query -> Search -> Filter/rerank -> Select context

- Generation:

    State task -> Separate instructions from untrusted context -> Include selected chunks -> Require grounded behavior -> Generate -> Return citations

- A strong RAG system treats retrieved documents as untrusted data. A document saying "ignore previous instructions" is content, not an instruction

- Authorization must happen at every source, ingestion, retrieval, and mutation boundary

- Use deterministic chunk IDs and hashes to make ingestion retries safe and skip unchanged embeddings

- Keep source ID, chunk offsets, and document version so answers can cite and be traced to exact evidence

- If evidence is missing, the system should say it cannot support the answer instead of filling gaps from confidence

# RAG Evaluation

- Retrieval metrics:

    Recall@k: Whether relevant material appears in the first k results

    Precision@k: Fraction of returned results which are relevant

    MRR: Reciprocal rank of the first relevant result

    nDCG: Ranking quality with graded relevance

- Generation metrics:

    Faithfulness/groundedness: Claims supported by retrieved context

    Answer relevance: Response addresses the question

    Citation correctness: Citations actually support claims

    Completeness: Required aspects are covered

    Safety: Prompt injection and access boundaries hold

- System metrics include end-to-end latency, embedding/search/generation latency, token use, cache hit rate, empty retrieval rate, and cost

- Build an evaluation set from real expected queries, difficult negatives, permission boundaries, stale documents, and adversarial instructions

- Evaluate retrieval separately from generation. A good answer can hide poor retrieval and a poor generation prompt can hide good retrieval

# Model Evaluation

- Classification metrics:

    Accuracy: Overall fraction correct

    Precision: Fraction of positive predictions which are correct

    Recall: Fraction of true positives found

    F1: Harmonic mean of precision and recall

    ROC-AUC / PR-AUC: Ranking quality across thresholds

- Regression metrics include MAE, MSE/RMSE, and R-squared

- Use confusion matrices and per-class metrics to see which errors are hidden by averages

- Calibration measures whether predicted confidence corresponds to actual correctness

- Compare against simple baselines and report uncertainty across seeds or folds when relevant

# Performance and Scaling

- Batch operations improve accelerator utilization but consume more memory

- Mixed precision uses lower-precision arithmetic where safe to reduce memory and increase throughput

- Gradient accumulation simulates a larger batch without holding every sample simultaneously

- Data parallelism replicates a model and splits batches. Model/tensor/pipeline parallelism splits a model which does not fit one device

- Profile data loading, host-device transfer, model operations, synchronization, and output processing before optimizing

- CPU inference benefits from quantization, batching, thread tuning, model size reduction, and efficient kernels

- Quantization reduces precision and memory, commonly to 8-bit or lower, with possible accuracy tradeoffs

- Distillation trains a smaller student from a larger teacher. Pruning removes parameters or structures

# Safety and Production Practices

- Version data, code, configuration, tokenizer, model, dependencies, and evaluation results

- Validate model input/output at the application boundary

- Moderate relevant user inputs and outputs for the product's risk model

- Apply rate limits, budgets, timeouts, retries with backoff, and concurrency limits

- Treat prompts, retrieved content, model output, and tool arguments as untrusted

- Do not let free-form model output directly perform database mutations. Parse into a strict schema, authorize the requested operation, and apply domain validation

- Monitor data drift, model quality, latency, cost, error rate, safety events, and feedback after deployment

- Keep a rollback path and avoid silently replacing models without evaluation

# Quick Reference

- Need gradients -> `requires_grad=True` and `backward()`

- Train behavior -> `model.train()`

- Evaluation behavior -> `model.eval()` plus `torch.inference_mode()`

- Multi-class loss -> Raw logits plus `CrossEntropyLoss`

- Variable-length batches -> Padding and attention mask

- Save learned state -> `state_dict`

- Generate text -> Autoregressive next-token loop with a decoding strategy

- Semantic retrieval -> Dense embedding search

- Exact names/identifiers -> Sparse, keyword, or hybrid retrieval

- Relational app plus vector search -> pgvector

- Unchanged document ingestion -> Content hash cache

- Multi-user RAG -> Authenticated scope inside the retrieval query

- Better final ranking -> Retrieve broadly then rerank

- Claim should be trusted -> Require supporting source and evaluate grounding
