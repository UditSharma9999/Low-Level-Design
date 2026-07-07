# Phase 06: Speech & Audio
## Bag of Words, TF-IDF, and Text Representation

**Bag of Words (BoW)** throws away order. For each document, count how many times each vocabulary word appears. Vector length is the vocabulary size. Position i is the count of word i.

**TF-IDF** reweights BoW. A word that appears in every document is uninformative, so scale it down. A word rare across the corpus but frequent in a single document is signal, so scale it up.

```text
TF-IDF(w, d) = TF(w, d) * IDF(w)
             = count(w in d) / |d| * log(N / df(w))
```
Where `TF` is term frequency in the document, `df` is document frequency (how many docs contain the word), `N` is total documents. The `log` keeps the weight bounded for ubiquitous words.

> Key property: both produce sparse vectors with interpretable axes. 

<br/>

<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## Word Embeddings — Word2Vec from Scratch

### The Problem
TF-IDF knows dog and puppy are different words. It does not know they mean nearly the same thing. 


### The Concept

**Distributional hypothesis**: "You shall know a word by the company it keeps."


**Word2Vec** comes in two flavors, both exploiting that idea.

1. **Skip-gram**. Given a center word, predict the surrounding words. `cat -> (the, sat, on)` with window size 2.

2. **CBOW (continuous bag of words)**. Given surrounding words, predict the center. `(the, sat, on) -> cat`.

Skip-gram is slower to train but handles rare words better. It became the default.

The network has one hidden layer with no nonlinearity. Input is a one-hot vector over the vocabulary. Output is a softmax over the vocabulary. After training, you throw away the output layer. The hidden layer weights are the embeddings.

`The trick`: softmax over 100k words is prohibitively expensive. Word2Vec uses negative sampling to turn it into a binary classification task. Predict "did this context word appear near this center word, yes or no". Sample a handful of negative (non-co-occurring) words per training pair instead of computing softmax over the whole vocabulary.

<br/>

<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## GloVe, FastText, and Subword Embeddings

### The Problem
models like Word2Vec tried to learn meanings by analyzing a huge table of how often words appeared together.

A second problem was dealing with new or rare words. Word2Vec and GloVe could only learn words they had already seen during training.

### The Concept
- **GloVe (Global Vectors)**. Build the word-word co-occurrence matrix `X` where `X[i][j]` is how often word j appears in the context of word `i`

- **FastText**. A word is the sum of its character n-grams plus the word itself. `where` becomes `<wh, whe, her, ere, re>, <where>`. The word vector is the sum of those component vectors. Train as Word2Vec.

- **BPE (Byte-Pair Encoding)**. Start with a vocabulary of individual bytes (or characters). Count every adjacent pair in the corpus. Merge the most frequent pair into a new token. Repeat for `k` iterations. Result: a vocabulary of `k + 256` tokens where frequent sequences (`ing, tion, the`) are single tokens and rare words are broken into familiar pieces. Every sentence tokenizes into something.



<br/>

<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## Sentiment Analysis

### The Concept
Classical sentiment is a two-step recipe.

1. **Represent**. Turn the text into a feature vector. BoW, TF-IDF, or n-grams.

2. **Classify**. Fit a linear model (Naive Bayes, logistic regression, SVM) on labeled examples.


<br/>

<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## Named Entity Recognition

## POS Tagging and Syntactic Parsing




<br/>

<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## CNNs and RNNs for Text

### The Problem
TF-IDF and Word2Vec produced flat vectors that ignored word order. A classifier built on them could not `tell dog bites man` from `man bites dog`. Word order sometimes carries the signal.


### The Concept

1. **Convolutional nets for text (TextCNN 2014)** : each word is converted into an embedding (a vector). Then, 1D convolution filters slide over groups of consecutive words (like 2-word or 3-word phrases) to detect important patterns. Global max-pooling keeps only the strongest match from each filter, no matter where it appears in the sentence. The outputs from multiple filter sizes are combined and passed to a classifier. This works well because each filter learns to recognize useful n-gram patterns (such as "not good"), and max-pooling makes the model insensitive to where those patterns occur, while allowing fast parallel training.

2. **RNN, LSTM, GRU**

<br/>

<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## Sequence-to-Sequence Models

**The bottleneck**. Everything the encoder learned about the source must be squeezed into that one context vector. Long sentences lose detail. Rare words get blurred. Reordering (chat noir vs. black cat) has to be memorized, not computed.

Attention (lesson 10) fixes this by letting the decoder look at every encoder hidden state, not just the last one. That is the whole pitch.

**Teacher Forcing** :   
During training, instead of feeding the decoder's own previous prediction, we feed it the correct previous word.

**Exposure Bias ⭐ (Very Important)**       
The mismatch between training and inference.    
- During training: Decoder sees the correct previous word.    
- During inference: Decoder only sees its own predictions.  

So it never learned how to recover from its own mistakes.

**Scheduled Sampling**: (A solution to exposure bias.)

Instead of always using the correct previous word:
- Early training → use 100% correct words.
- Later training → sometimes use the model's own prediction.

**Greedy Decoding ⭐**: At every step, choose the word with the highest probability.



<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## Attention Mechanism — The Breakthrough | ✅


<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## Machine Translation

### The three-tier evaluation hierarchy 

1. **Heuristic metrics (BLEU, chrF)**: These are traditional, reference-based metrics that compare the model's translation with a correct human translation. BLEU checks how many words or phrases (n-grams) match the reference, while chrF compares character sequences, making it better at handling spelling changes and languages with rich word forms. They are fast, easy to compute, and useful for tracking whether a new model is better or worse, but they often give low scores to correct translations that use different wording (paraphrases).

2. **Learned metrics (COMET, BLEURT, BERTScore)**: These use neural networks instead of simple word matching. They understand the meaning of sentences and compare the translation with the source and/or reference. For example, if two translations use different words but express the same idea, these metrics usually recognize them as similar. Among them, COMET is widely considered the best choice for production systems because its scores align closely with human judgments.

3. **LLM-as-a-Judge (Reference-free)**: Instead of comparing with a reference translation, a large language model (LLM) evaluates the translation directly. You can ask it to rate fluency, accuracy, tone, grammar, and cultural appropriateness. This is especially useful when no reference translation exists, such as evaluating creative writing or newly translated content. A well-designed evaluation prompt can produce judgments that are often close to those of human reviewers.

production translation systems don't just need fluent translations—they also need safeguards to prevent `hallucinations`, `wrong-language (Off-target generation)`, `inconsistent terminology (Terminology drift)` , `incorrect formality`, and `overly long responses`.


<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## Text Summarization

### The Problem

**Extractive summarization** is a ranking problem. Score every sentence, return the top-k. The output is always grammatical because it is lifted verbatim. The risk is missing content that is distributed across the article.

**Abstractive summarization** is a generation problem. A transformer produces new text conditioned on the input.


### The Concept
- **Extractive**. Treat the article as a graph where nodes are sentences and edges are similarities. Run PageRank (or something like it) over the graph to score sentences by how connected they are to everything else. Highest-scoring sentences are the summary. The canonical implementation is TextRank (Mihalcea and Tarau, 2004).

- **Abstractive**. Fine-tune a transformer encoder-decoder (BART, T5, Pegasus) on document-summary pairs. At inference, the model reads the document and generates the summary token-by-token via cross-attention.

- **ROUGE (Recall-Oriented Understudy for Gisting Evaluation)** is a set of metrics used to evaluate text generation tasks such as summarization by comparing a model's output with a human-written reference summary. `ROUGE-1` measures how many individual words (unigrams) overlap between the generated and reference summaries. `ROUGE-2` measures the overlap of two-word sequences (bigrams), which better reflects whether the model preserves short phrases. `ROUGE-L` measures the Longest Common Subsequence (LCS).

    Always use stemming. Without it, "running" and "run" count as different words and ROUGE undercounts.

Beyond ROUGE:
- BERTScore 
- BARTScore 
- MoverScore  
- more....



<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## Question Answering Systems

**Extractive QA** assumes the answer already exists in a given passage. A transformer model (such as BERT) reads both the question and the passage together, then predicts the start and end positions of the answer within the passage.

**Generative (Closed-book) QA** relies entirely on a large language model's internal knowledge. The model answers directly from what it learned during training, without retrieving any external documents.

**Open-domain QA (Retrieval-Augmented QA or RAG)**




<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">



## Information Retrieval and Search

- **Sparse retrieval (BM25)**. Fast, precise on exact matches, terrible on semantics. Run over an inverted index. Sub-10ms per query on millions of documents. Gets you statute references, product codes, error messages, named entities right.

- **Dense retrieval**. Encode query and documents into vectors. Nearest neighbor search. Captures paraphrases and semantic similarity. Misses exact keyword matches that differ by one character. 50-200ms per query with FAISS or a vector DB.

- **Fusion**. Merge the ranked lists from sparse and dense. Reciprocal Rank Fusion (RRF) is the easy default because it ignores raw scores (which live in different scales) and only uses rank positions. Weighted fusion is an option when you know one signal dominates for your domain.

- **Cross-encoder rerank**. Take the top-30 from fusion. Run a cross-encoder (query + document together, scoring each pair). Keep the top-5. Cross-encoders are slower per pair than bi-encoders but far more accurate. You amortize by only running them on the top-30.

<br/>

| Metric                                             | Meaning                                                                                                                                                                                                                   |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Recall@k**                                       | Of all queries where the correct document exists in the database, how often is it found within the top **k** retrieved results.                                                                                           |
| **MRR (Mean Reciprocal Rank)**                     | Measures how early the first correct result appears. For each query, take **1 / rank of first relevant document**, then average across all queries. Higher is better when relevant results appear near the top.           |
| **nDCG@k (Normalized Discounted Cumulative Gain)** | Evaluates ranking quality by giving higher scores to relevant documents that appear higher in the list, and it also handles **graded relevance** (e.g., highly relevant vs somewhat relevant), not just yes/no relevance. |

<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## Topic Modeling — LDA and BERTopic
## Text Generation Before Transformers — N-gram Language Models


<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## Chatbots — Rule-Based to Neural to LLM Agents

> !Chatbot evolution: rule-based → retrieval → neural → agent

- Rule-based chatbots (like ELIZA or DialogFlow systems) work using hand-written rules. 

- Retrieval-based chatbots 

- Neural chatbots (based on encoder-decoder models)

- LLM agents (modern systems like GPT-based assistants) 


<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## Multilingual NLP

## Subword Tokenization — BPE, WordPiece, Unigram, SentencePiece

## Structured Outputs & Constrained Decoding | ✅


## Natural Language Inference — Textual Entailment

<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## Embedding Models — The 2026 Deep Dive

- **Dense embeddings**. One vector per passage (usually 384-3,072 dimensions). Cosine similarity ranks passages by semantic proximity.

- **Sparse embeddings**. SPLADE-style. A transformer predicts a weight for every vocab token, then zeros out most of them. 

- **Multi-vector (late interaction)**. One vector per token. Scoring with MaxSim: for each query token, find the most similar document token, sum the scores. More expensive to store and score, but wins on long queries and domain-specific corpora.


......


<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">

## Chunking Strategies for RAG

### The Concept

1. **Fixed chunking**. Split every N characters or tokens. Simplest baseline. Breaks mid-sentence. Good compression, bad coherence.

2. **Recursive**. LangChain's `RecursiveCharacterTextSplitter`. Try splitting on `\n\n` first, then `\n`, then `.`, then space. Falls back cleanly. The 2026 default.

3. **Semantic**. Embed each sentence. Compute cosine similarity between adjacent sentences. Split where similarity drops below a threshold. Preserves topic coherence. Slower; sometimes produces tiny 40-token fragments that hurt retrieval.

4. **Sentence**. Split on sentence boundaries. One sentence per chunk or a window of N sentences. Matches semantic chunking up to ~5k tokens at a fraction of the cost.

5. **Parent-document**. Store small child chunks for retrieval and the larger parent chunk for context. Retrieve by child; return parent. Degrades gracefully: bad child chunks still return reasonable parents.

6. **Late chunking** Instead, you first feed the entire document into a model that understands long text, so it sees the full context. Only after that do you break it into chunks. This helps because each chunk “knows” what the whole document was about, so meaning is preserved better. The downside is that it is slow and expensive, since the model must process the full document before splitting.

7. **Contextual retrieval** means you improve each chunk by adding a short explanation of where it came from. For example, instead of storing just a paragraph, you store: “This is from the section about refund policies in a customer support document” + the paragraph itself. This extra context helps search systems find the right chunk more accurately. The downside is that you need an LLM to generate these descriptions beforehand, which makes **indexing more costly**.

<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## Coreference Resolution

## Entity Linking & Disambiguation

## Relation Extraction & Knowledge Graph Construction


<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## LLM Evaluation — RAGAS, DeepEval, G-Eval

When you evaluate a RAG (Retrieval-Augmented Generation) system, simple exact matching doesn’t work well. For example, if the system answers “June 29th, 2007” but the reference says “June 29, 2007,” a strict match gives a score of zero, even though a human would say it is correct.

**RAGAS** measures different aspects of RAG quality, such as whether the answer is faithful to the retrieved context, whether the context actually contains the needed information, and whether the answer is relevant to the question. It often uses a combination of NLI models and LLM judges to score outputs.

**DeepEval** works like a testing framework (similar to pytest) but for LLM systems. It allows you to run automated evaluation tests in CI/CD pipelines and includes metrics like hallucination detection, task success, and bias checks.

**G-Eval** is a specific method where a large language model is used as a judge. It evaluates outputs based on clear criteria (like correctness or relevance) and often uses step-by-step reasoning before giving a final score.


### The Concept
**LLM-as-judge (core idea)**: Instead of checking answers with exact string rules, we use another LLM as a grader. We give it:

- the question
- the retrieved context
- the model’s answer
- a rubric (scoring rules)

Then we ask it something like: 
`“Score this answer from 0 to 1 based on faithfulness.”`

Why it can fail:
- Bias
- Silent failures (invalid JSON or parsing breaks)
- Model drift: If you change the judge model version, your scores can change even if your system didn’t improve.

#### The 4 main RAG evaluation dimensions

| Metric              | Question                                                                 | Backend                                                                 |
|---------------------|--------------------------------------------------------------------------|-------------------------------------------------------------------------|
| Faithfulness        | Does each claim in the answer come from the retrieved context?          | NLI-based entailment                                                    |
| Answer relevance    | Does the answer address the question?                                   | Generate hypothetical questions from answer; compare to real question   |
| Context precision    | Of retrieved chunks, what fraction were relevant?                       | LLM-judge                                                               |
| Context recall      | Did retrieval return everything needed?                                 | LLM-judge against gold answer                                           |


<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## Long-Context Evaluation — NIAH, RULER, LongBench, MRCR


<br/>
<hr style="border: none; border-top: 2px dashed #999; height: 0;">


## Dialogue State Tracking

It is the process of keeping track of what a user wants during a multi-turn conversation in a structured way.

In a task like restaurant booking, the user doesn’t give all details at once. They refine their request over multiple messages:

- “I want a cheap restaurant in the north”
- “Actually make it moderate”
- “Add Italian”

Each message updates the system’s internal state, usually stored as a dictionary (slot-value pairs), like:

```JSON
{
  cuisine: Italian,
  area: north,
  price: moderate
}
```

Continuously update this state as the conversation changes. It must correctly add new information, modify old information, and remove or override previous constraints.

### The Concept

DST is defined using a schema, which lists domains (like restaurant or hotel) and their slots (like price, area, or cuisine). Slots can be **closed-vocabulary** (fixed choices like cheap/moderate/expensive) or **open-ended** (like a restaurant name).

There are two main ways to solve DST. The **classification approach** checks each possible slot-value pair and decides whether it is correct (yes/no), which works well for fixed options. The **generation approach** instead directly produces slot values as text, which is more flexible and is the modern standard.

Performance is usually measured using **Joint Goal Accuracy (JGA)**, which is strict: the prediction is only correct if all slots in the dialogue state are correct at the same time. Even one mistake counts as failure, which makes DST challenging.

> The most modern approach combines LLMs with structured output constraints (like Pydantic schemas and constrained decoding) to ensure reliable, valid state updates.


