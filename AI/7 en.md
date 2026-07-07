# Transformers Deep Dive

## Why Transformers — The Problems with RNNs | ✅

## Self-Attention from Scratch | ✅
## Multi-Head Attention | ✅

## The Full Transformer — Encoder + Decoder | ✅

<br/>

<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## BERT — Masked Language Modeling

Unlike GPT, which predicts the next word, BERT learns by predicting missing words in a sentence

The model's job is to guess the original word using the words before and after it. For example, in the sentence "The [MASK] is barking loudly," BERT can easily predict that the missing word is "dog" because it can see both the left and right context. This is called Masked Language Modeling (MLM) 

BERT is built using only the Transformer Encoder, which means every word can pay attention to every other word in the sentence through bidirectional self-attention. Because it looks at both past and future words simultaneously.

<br/>

<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## GPT — Causal Language Modeling

After GPT predicts the next token, it must decide which token to output. This process is called decoding or sampling, and different strategies produce different behaviors:

1. **Greedy Decoding** always picks the token with the highest probability. It is deterministic and commonly used for tasks like code completion.

2. **Temperature Sampling** controls randomness

3. **Top-k Sampling limits** the choice to the top k most likely tokens before randomly selecting one.

4. **Top-p (Nucleus) Sampling** selects the smallest set of tokens whose combined probability exceeds a threshold (such as 0.9), adapting dynamically to the probability distribution.

5. **Min-p Sampling**, introduced more recently, removes extremely unlikely tokens based on their probability relative to the most likely token, often producing more natural responses than Top-p.

6. **Speculative Decoding** uses a small, fast model to propose several tokens at once, which are then verified by the larger model. This reduces generation latency significantly while maintaining output quality.



<br/>

<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## T5, BART — Encoder-Decoder Models

## Vision Transformers (ViT)

## Audio Transformers — Whisper Architecture


<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## Mixture of Experts (MoE) 

Mixture of Experts (MoE) model. Instead of making the AI use its entire brain for every word, it activates only a small group of specialized parts. This makes the model much larger and smarter without making every prediction extremely expensive.

To understand why this was needed, first look at a dense transformer, which is how earlier models like GPT-3 or Llama 2 worked. In a dense model, every parameter (every learned weight) is used every time the model predicts a token (word or part of a word).


Mixture of Experts solves this problem by dividing one large neural network layer into many **smaller expert** networks. Each expert is simply a small feed-forward network (FFN) that specializes in certain kinds of information. Instead of sending every token through every expert, the model uses a **router**. The router is a tiny neural network whose only job is to decide **which experts should process each token**.

> **It means that instead of having a few big experts, the model has many small experts that can be mixed together in different combinations.**


Router examines the hidden representation of every token and assigns a score to each expert. The highest-scoring experts are selected. Their outputs are combined using weights based on these scores, producing the final output for that layer. Every token makes its own decision independently, so different words in the same sentence may be processed by different experts.

However, this creates another problem called **load balancing**.   
**solution**: Instead of changing the training objective, they adjust a small bias value for each expert. If an expert is selected too often, its bias is slightly reduced so it becomes less likely to be chosen. If it is selected too rarely, its bias is increased. Over time, this naturally balances the workload without hurting the model's learning.

MoE also has a major **disadvantage**: memory. Even though only a few experts are active at a time, all experts must still be stored in memory because any expert might be selected.


## KV Cache, Flash Attention & Inference Optimization
Think of a large language model (LLM) like a person writing one word at a time. Before writing each new word, it looks back at everything written so far to decide what comes next. This "looking back" process is called self-attention. During training, the model can see the entire sentence at once, so it processes many words in parallel. During inference (when the model is generating text for you), it only knows the words it has already generated, so it must generate one token at a time. This difference is why training and inference have completely different performance bottlenecks. Training is mainly limited by **computation (FLOPs)**, while inference is mainly limited by memory movement.


The model performs a lot of unnecessary work during inference. The model has already generated the sentence, "The weather today is very". When it wants to generate the next word, it calculates attention over all previous tokens. it again recalculates attention for "The", "weather", "today", "is", and "very", even though none of those earlier tokens have changed.

During self-attention, every token is converted into three vectors called Query (Q), Key (K), and Value (V). When a new token is generated, only its Query changes. The Keys and Values of all previous tokens never change because those tokens are already fixed. Therefore, instead of recomputing all previous Keys and Values every generation step, the model stores them in memory. This stored memory is called the **KV cache**.

**Storing consumes memory**:
The cache size increases linearly with three factors: the number of generated tokens, the batch size, and the number of transformer layers.

One important optimization is **Grouped Query Attention (GQA)**. Normally, every attention head has its own Key and Value vectors. GQA allows multiple Query heads to share the same Keys and Values.

**The problem**: Standard attention creates a huge N×N matrix of token relationships, which consumes lots of GPU memory and requires repeated data movement. The GPU spends more time moving data between memory (HBM) than doing calculations, making attention slow and memory-bound.

**Flash Attention** solves this problem by changing the order of computation rather than changing the mathematical result. Instead of creating one enormous attention matrix, Flash Attention divides the sequence into small blocks called **tiles**. Each tile easily fits inside the GPU.

The difficult part of Flash Attention is the softmax calculation. Normally, softmax needs to see all attention scores before computing probabilities. Flash Attention instead maintains a running maximum and a running sum while processing one tile after another. This clever mathematical trick allows it to produce exactly the same final answer without ever storing the complete score matrix. The output is mathematically equivalent to standard attention, except for tiny floating-point rounding differences.

**prefix caching**. Many users may share the same beginning of a prompt. Instead of recomputing the KV cache for that shared prefix every time, the system stores the cache once and reuses it across multiple requests.

**PagedAttention**, introduced by vLLM. Instead of storing the KV cache as one large continuous block of memory, it divides the cache into small fixed-size blocks.

**speculative decoding**. Here, a small, fast model predicts several future tokens at once. The large, accurate model then checks whether those predictions are correct. If most predictions are accepted, the expensive model effectively generates multiple tokens in a single forward pass instead of one token at a time. This reduces latency without sacrificing output quality.

<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## Scaling Laws

**if you have a fixed amount of computing power, should you spend it on making the model bigger (more parameters) or training it on more data (more tokens)?**

> `Compute ≈ 6 × Parameters × Tokens`  
> This means you have a limited budget. If you increase the number of parameters, you may have fewer tokens left. If you increase tokens, you may need a smaller model. The challenge is finding the best balance.

The mathematical idea behind scaling laws is that model loss depends on both model size and training data:

**Loss = model size error + data error + unavoidable error**

If the model is too small, it cannot learn enough patterns. If the dataset is too small, the model does not get enough examples. The goal is to reduce both errors.

data quality: Not all tokens are equally useful.



<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## Build a Transformer from Scratch — The Capstone

<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## Attention Variants — Sliding Window, Sparse, Differential


### Sliding Window Attention (SWA)
Instead of looking at the entire previous text, each token only looks at the most recent few tokens.

This reduces the cost from: Full attention: `O(N²)` Every token connects to every token to Sliding window: `O(N × W)`: Where W is the window size.

This also reduces the KV cache size. The downside is that the model may forget very old information. To solve this, modern models mix sliding window layers with some full attention layers. For example, a model might use: 
- 5 layers of sliding window attention
- 1 layer of full attention

The sliding layers handle local understanding, while the full attention layers allow the model to access distant information.

### Sparse Attention
Sparse attention is another approach where the model chooses only certain token connections instead of all possible connections.

Instead of creating an N×N attention matrix, sparse attention creates only the important connections.

- Local attention: A token looks at nearby tokens.

- Global attention: Some special tokens can see everything and everything can see them.

- Strided attention:A token looks at every few tokens from the past.

This gives the model both local information and some long-range understanding.

### Differential Attention

**Attention sink problem**: In normal attention, the softmax function forces all attention scores to add up to 1. Sometimes the model gives unnecessary attention to the first few tokens, even when those tokens are not important.

Differential Attention uses two attention calculations:

- **First attention**:"Which tokens are important?"

- **Second attention**: "Which tokens are receiving unnecessary attention?"

Then it subtracts the second from the first.    
**Useful attention - unwanted attention = cleaner attention**

The result is that the model focuses more on meaningful tokens and reduces wasted attention.


<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## Speculative Decoding — Draft, Verify, Repeat .....

