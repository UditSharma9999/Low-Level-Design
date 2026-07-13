A Flask + Transformers server struggles with LLMs because of three fundamental bottlenecks:


- **Autoregressive inference**: LLMs generate text one token at a time, and each token depends on the previous one. This means a single request cannot be parallelized across tokens, so faster GPUs mainly improve throughput (more concurrent requests) rather than reducing individual response time.

- **Variable output lengths**: Different requests produce very different numbers of tokens (e.g., 20 vs. 2000). With static batching, short requests finish early while the GPU waits for long ones, wasting compute. Modern LLM servers solve this using continuous batching, which dynamically replaces finished requests with new ones.

- **KV cache memory**: During generation, each request stores a Key-Value (KV) cache that often becomes the main limit on concurrency, sometimes more than the model weights themselves. Naively allocating large contiguous KV cache blocks wastes GPU memory due to fragmentation. Techniques like PagedAttention improve memory utilization and allow more concurrent requests.

For high traffic (~30 RPS), it's better to self-host an open-weight LLM using vLLM on 2× H100 GPUs with tensor parallelism. vLLM improves performance through continuous batching (keeps the GPU busy by dynamically batching requests) and PagedAttention (efficiently manages the KV cache to avoid memory fragmentation). The real limit on concurrency is KV-cache memory per request, not GPU FLOPs or model weights. For low traffic (~1 RPS), using a hosted LLM API is more cost-effective. A practical crossover point is around 5–10 sustained RPS, beyond which self-hosting often becomes worthwhile.

> 💡Key interview takeaway: Mentioning that "KV-cache memory, not weights or FLOPs, is the main concurrency bottleneck" demonstrates a deeper understanding of LLM inference than simply naming vLLM.

**The maximum number of requests an LLM can handle at once is usually limited by the GPU memory needed for each request's KV cache, not by the model weights or GPU compute. Once the KV cache fills up, no more concurrent requests can be served.**


KV cache stores the Keys (K) and Values (V) generated for every previous token during text generation. Instead of recomputing them for each new token, the model reuses this cached information, making autoregressive decoding much faster.

The KV cache dominates GPU memory because every active request has its own cache, and it grows with the number of generated tokens. Model weights are a fixed memory cost, but the KV cache is a per-request memory cost, so it is usually the main factor limiting how many concurrent users an LLM server can handle

**Problem with static batching:**
In LLMs, requests generate different numbers of tokens (e.g., 20 vs. 2000). Static batching keeps all requests in the batch until the longest one finishes, so completed requests sit idle, wasting GPU compute and KV-cache memory.

**Continuous batching:**
After every decoding step (every generated token), the server removes finished requests and immediately adds new waiting requests. This keeps the GPU busy almost all the time, improving throughput by 2–3× on typical chat workloads (and even higher at very high concurrency).

### Which serving framework should you pick?

- **vLLM (default choice)**: Best general-purpose open-source serving framework. It supports most open-weight models, provides an OpenAI-compatible API, and includes Continuous Batching and PagedAttention by default. It's the safest and most common interview answer.
- **TensorRT-LLM**: Best when you're fully committed to NVIDIA GPUs and need the highest throughput. It uses optimized TensorRT kernels and can be faster than vLLM, but requires model compilation and adds operational complexity.

- **SGLang**: Best for agent workloads or applications with shared prompt prefixes. Its RadixAttention reuses KV caches for common prefixes, giving 2–5× speedups when prompts or conversation history are reused.

> 💡Interview-ready answer (20–30 sec)  
> "I'd start with vLLM because it's the open-source default, supports most models, and provides Continuous Batching and PagedAttention out of the box. If we're heavily invested in NVIDIA hardware and need maximum throughput, I'd benchmark TensorRT-LLM. For agentic workloads with lots of shared prompt prefixes, I'd consider SGLang because RadixAttention can significantly improve performance through KV-cache reuse."

**Speculative decoding** is a technique that speeds up LLM text generation by making better use of the GPU during the decode phase. LLM inference has two phases: **prefill**, where the model reads the entire prompt at once, and decode, where it generates one token at a time. Prefill is **compute-bound** because many tokens are processed in parallel, keeping the GPU fully busy. Decode is **memory-bound** because, for every new token, the model has to repeatedly read its weights from GPU memory while doing very little computation. As a result, much of the GPU's compute power sits idle.

Speculative decoding takes advantage of this idle compute by introducing a small, fast **draft model** that predicts several future tokens at once. Instead of the large model generating one token at a time, it verifies all of the draft model's predicted tokens in a **single forward pass**. This increases the amount of computation done for each time the model weights are loaded, improving GPU efficiency. If the draft model's predictions are correct, the tokens are accepted; if not, they are rejected and replaced using **rejection sampling**, which ensures the final output is **identical in quality and probability distribution** to normal decoding.

The speedup depends on the acceptance rate—the more accurate the draft model, the more tokens are accepted and the bigger the performance gain. However, speculative decoding **works best for single-user or small-batch requests**.....

---

## Production GenAI systems reduce costs using five main cost levers.

- **Model routing (cascading)**: Simple requests are sent to smaller, cheaper models, while only complex queries go to expensive frontier models. This can reduce inference costs by 40–70% for routed traffic.

- **Semantic caching**: The system stores answers to previous questions using embeddings. If a new question has the same meaning as a cached one, it returns the cached response instead of calling the LLM. This provides millisecond responses, almost zero inference cost, and typical cache hit rates of 20–40%.

- **Prompt compression**: Before sending a prompt to the LLM, unnecessary or repetitive content is removed. Reducing a prompt from, say, 5000 tokens to 1500 tokens cuts input token costs by 2–5×.

- **Batch API**: For non-urgent tasks like overnight document summarization, requests are grouped and processed asynchronously using batch APIs, which typically reduce costs by about 50% at the expense of higher latency.

- **Output minimization**: Applications often need only structured or concise outputs instead of long explanations. Using JSON mode, schemas, stop sequences, and output limits reduces output tokens, saving 30–80% because output tokens are the most expensive.

### Why are output tokens more expensive? 
Input tokens are processed together during the prefill phase, making GPU usage highly efficient. Output tokens are generated one at a time during the decode phase, which is memory-bandwidth-bound and requires repeatedly reading model weights from GPU memory. Because decoding uses GPU resources less efficiently, providers typically charge 3–5× more for output tokens, making output minimization one of the most effective cost-saving techniques.


### 1. Model Routing (Cascading) 
A **router** decides which model to use. There are three common approaches:
 
- **Heuristic router**: Uses simple if-else rules (e.g., keywords or query length). It's fast and cheap but doesn't understand meaning.

- **Embedding similarity router**: Converts queries into embeddings and compares them with labeled examples. It understands semantic meaning, achieves around 85–95% accuracy, and is the default choice for most production systems.

- **Tiny LLM router**: A small trained model (1B–7B parameters) classifies requests as Mini or Frontier. It offers the highest accuracy (95–98%) but requires training, fine-tuning, and additional infrastructure.


The biggest challenge is **misrouting**:

- `Easy → Frontier`: Only wastes money.

- `Hard → Mini`: Produces poor answers, reduces user trust, and may require retries or human intervention. This is much more costly and is called misroute asymmetry.

To reduce this risk, systems use a **confidence-based fallback**. The router first sends the request to the mini model. If the model is confident, its answer is returned. If confidence is low, the request is automatically retried using the frontier model. Although a few requests are processed twice, the overall cost remains much lower than always using the expensive model.

**When not to use routing**: Routing is only useful when the workload has a mix of easy and hard requests. If all requests are similarly complex (e.g., invoice extraction or code review), the router has little to decide, and the added complexity isn't worth it. In those cases, techniques like output **minimization or Batch APIs** usually provide better value.

### 2. Semantic caching and the Batch API 
Semantic caching and the Batch API are two powerful cost-saving techniques because they often **avoid expensive LLM calls altogether**. Unlike model routing, which still calls a cheaper model, semantic caching reuses previous answers, and the Batch API processes non-urgent work at a lower price.

**Semantic caching** works by recognizing the meaning of a user's question instead of looking for an exact text match. Each query is converted into an embedding (a numerical vector), and its embedding is compared with previously cached embeddings using cosine similarity. If the similarity is above a chosen threshold, the cached answer is returned in 5–10 ms instead of making an LLM call that could take 500 ms to 3 seconds. If no similar question is found, the LLM generates a new answer, which is then stored in the cache for future use. This significantly reduces latency, API costs, and GPU usage.

The **cosine similarity threshold** is important because it controls the balance between accuracy and cache hits. A low threshold (e.g., 0.85) may incorrectly treat related questions (like "reset password" and "reset MFA") as identical, returning the wrong answer. A very high threshold (e.g., 0.99) is too strict, resulting in very few cache hits. A practical default is 0.92–0.93, which provides good savings while keeping incorrect matches rare.


The effectiveness of semantic caching depends on the application. Enterprise FAQ and customer-support bots often achieve 15–40% cache hit rates because many users ask similar questions. Coding assistants usually see less than 10% because most queries are unique. Cached responses can become outdated, a problem called **cache staleness**. This is addressed using **TTL (Time To Live)** to expire entries after a set period, **cache versioning** to invalidate caches when source documents change (especially in RAG systems), and negative caching, which temporarily stores responses like "I don't know" or "I can't answer that" to avoid repeatedly calling the LLM for impossible or forbidden questions.

The **Batch API** is designed for tasks that do not require immediate responses. Instead of processing requests in real time, providers execute them later (often within 24 hours) when GPUs are less busy, typically reducing costs by about 50%. It is ideal for document classification, bulk PDF extraction, nightly chat summaries, evaluation and regression testing, embedding backfills, and content moderation queues. Many teams overlook Batch API savings because they focus on their chatbot, but a significant portion of their LLM usage often comes from these offline tasks.

### 3. Output minimization
Output tokens cost 3–5× more than input tokens for two reasons. First, technically, input tokens are processed during the prefill phase, where thousands of tokens are handled in parallel, making the GPU highly efficient. Output tokens are generated during the decode phase, one token at a time, making decoding **memory-bandwidth-bound** and much slower. Second, **providers intentionally price output tokens higher** because decoding is the real bottleneck: output generation consumes KV-cache memory and decode slots, which limit how many users can be served concurrently. The extra price is partly a business decision to ration this scarce resource.

Because of this, **reducing output length is one of the highest-impact cost optimizations**. For example, reducing a response from 500 tokens to 100 tokens saves 5× output tokens, and since output tokens are about 3× more expensive, the savings on the output portion of the bill are even more significant.

There are four main **output minimization techniques**:


- **JSON mode / Structured outputs**: Force the model to return only structured JSON following a schema. This eliminates unnecessary explanations, reduces output tokens by around 70%, and makes the response easy to parse.

- **Stop sequences**: Define strings (e.g., `"Sources:"` or `"</answer>"`) where generation should stop. This prevents the model from adding unnecessary filler and can reduce output by 20–40%.

- **Length limits**: Instruct the model to respond briefly (e.g., "Respond in under 50 words. No preamble.") and combine this with a `max_tokens` limit to enforce a hard cap.

- **Don't request explanations you don't need**: If your application only needs a classification or JSON result, don't ask the model to explain its reasoning. Extra explanations create expensive output tokens without adding value. Reserve detailed reasoning only for debugging or evaluation workflows.

The first step in reducing LLM costs is to understand where the money is being spent. Analyze one week of logs to measure:

- Input vs. output tokens per request.
- Repeat query rate (using embedding similarity).
- Interactive vs. batch-eligible requests.
- Frontier vs. mini model usage.

**Streaming is not a cost optimization**. It only displays tokens as they are generated, improving perceived latency but not reducing the number of tokens or the bill.

### 4. Prompt compression

Prompt compression reduces input tokens by removing low-value or repetitive text before sending the prompt to the LLM. Tools like LLMLingua use a small model to identify and remove unimportant tokens, typically achieving 2–5× compression with less than a 2 percentage-point quality drop on RAG workloads.

Prompt compression is most useful for:

- Long-context RAG applications.
- High-volume extraction, classification, or summarization pipelines.
- Applications using very large context windows (e.g., 128k tokens).

It is not recommended for:

- Code assistants, where every token may be important.
- Short prompts (under ~500 tokens), where the compression overhead outweighs the savings.
- Latency-sensitive applications, since compression adds about 50–150 ms and requires an extra small-model call.

#### Which cost lever should you use first?

A good default order is:

1. Output minimization – highest ROI, easiest to implement.
2. Semantic caching – excellent when many questions repeat.
3. Model routing – high savings but more engineering effort.
4. Batch API – ideal for offline workloads.
5. Prompt compression – most useful only for long-context prompts.

Prompt compression reduces the number of input tokens by removing unnecessary information before sending the prompt to the LLM. In RAG applications, retrieved documents often contain formatting, boilerplate, repeated context, and less relevant sentences, while chat history may repeat information the model already has. These extra tokens increase cost without adding much value.

A popular tool is LLMLingua (and LLMLingua-2 and LongLLMLingua). It uses a small model (~1B parameters) to score how important each token is for generating the final answer, then removes low-information tokens.

Another approach is **prompt distillation**, where a long, verbose system prompt is rewritten into a much shorter version that produces the same behavior. For example, a lengthy instruction like "You are a helpful, harmless, honest assistant..." can often be replaced with a single concise sentence after validating on an evaluation set, reducing input tokens without affecting output quality.


#### These three techniques are cost-related, but they are not considered the main GenAI cost levers because they don't directly reduce variable LLM inference costs in the same way as routing, caching, or output minimization.


- **Streaming**: Streaming only improves perceived latency by showing tokens as they are generated. It does not reduce the number of tokens generated or the API bill, so it is a UX optimization, not a cost optimization.

- **Self-hosting**: Running an open-weight model on your own GPUs can become cheaper than using hosted APIs at around 5–10 sustained requests per second (RPS), with much larger savings at higher traffic. However, it changes the pricing model from pay-per-token (variable cost) to GPU rental (fixed cost), so it's an infrastructure decision rather than a cost-reduction lever.

- **Quantization (FP8, INT8, INT4)**: Quantization reduces model memory usage and speeds up inference, but it mainly matters when you self-host models. If you're using a hosted API, the provider already decides the quantization, so you don't control this optimization.

> For cost questions, don't list every optimization. Instead, explain 2–3 major levers in depth (such as model routing, semantic caching, and output minimization), include expected savings, and mention the trade-offs (e.g., routing errors handled with a confidence fallback). This demonstrates a deeper understanding than briefly naming many techniques.

<br/>

**Prompt caching** is not a single standardized feature because different LLM providers implement it differently. While the goal is the same—reuse previously processed prompt prefixes to reduce input token cost and latency—the APIs and engineering approach vary.

- **Anthropic**: Uses cache breakpoints. The developer explicitly marks a point in the prompt using a cache_control block. Everything before that marker is cached (typically for about 5 minutes). Future requests only get a cache hit if the cached prefix is byte-for-byte identical.
- **OpenAI**: Uses automatic prefix caching. The developer doesn't need to configure anything. The system automatically detects the longest matching prompt prefix from recently processed prompts and reuses it.

## Cost Attribution 
Cost attribution means tracking exactly where every LLM cost comes from. Instead of seeing one large monthly API bill, every LLM request is tagged with metadata such as tenant ID, feature, user ID, route, model, and request ID. This allows you to answer questions like:

- Which feature is the most expensive?
- Which customer generates the highest cost?
- Did a recent deployment increase token usage?

Along with these tags, the system also records the input tokens, output tokens, and cost for each request. Because every LLM call is now linked to this metadata, you can later analyze exactly which customer, feature, user, or model is responsible for the cost, turning a single monthly bill into actionable insights that help engineering, product, and finance understand and optimize spending.

### Why tag requests at the beginning?
When a user request first reaches your backend, middleware automatically adds tags (tenant, feature, user, route, model, etc.) before any LLM call is made. Every downstream LLM request automatically inherits these tags, so developers don't have to remember to add them manually.

### Budget and operational alerts

Once attribution is available, automated alerts can be created:

- Feature cost alarms: Notify a team if a feature's daily cost suddenly spikes.

- Tenant budget alarms: Detect enterprise customers exceeding expected usage.

- Heavy-user alarms: Identify users responsible for a large share of total cost.

- Cache hit alarms: Alert engineers if prompt cache hit rates suddenly drop, often indicating an accidental prompt or configuration change.

> 💡 Senior interview takeaway   
> A strong answer is:   
**"Before optimizing costs, I'd first implement cost attribution by tagging every LLM request with tenant, feature, user, route, and model. Then I'd build a daily cost cube so finance, product, and engineering can identify where the money is being spent. Only after measuring costs would I apply optimizations like routing, caching, batching, or prompt optimization to the parts of the system that actually need them."**

<br/>

prompt engineering" is misleading because it makes people think the