
# Task1: Transformer Decoder KVCahe

Most LLMs based on *Transformer*, such as Llama and ChatGLM, they're decoder-only frame, and use **causal language modeling(CLM)** as model to do pre-training.
- auto-regressive.
	- *Prefilling*
		- LLM input a *complete*, *unseen* query sequence. these tokens are not calculated attention.
		- model do one forward, and return the probability distribution of next token.
	- *Decoding*
		- After, to generate *tokens*, we will use new generated tokens as new input to LLM, then it do attention with now token and all token before.
- 所以可以 *Prefilling*阶段开始对这些 key 和 value 进行缓存。

