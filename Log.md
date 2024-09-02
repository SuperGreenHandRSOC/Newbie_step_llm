## Doc.
1. Architecture
	1. simple version & Basic idea
		1. overview
		2. Encoder, Decoder
		3. Attention
			- [x] Coding: Attention
	2. comprehensive version
		1. Encoder-only: BERT
			1. architecture
			2. pre-training
		2. Decoder-only: GPT
			1. Brief history: **What's new, what's the differences**
			2. pre-training
			- [ ] tokenizer, embedding, transformer block, im_head
		3. Encoder-Decoder
		4. Comparation
			- [ ] bert vs. GPT
			- [ ] decoder-only vs. encoder-decoder
		- [ ] Coding
2. Training process

## step 1 transformer 基本框架
- [ ] 结果不同是算法写得不对还是随机数生成不同？
- [ ] 贴一下代码 + 写注释介绍思路

- [ ] 关于Attention反向传播的推导是否、如何有利于下文我对更细致分析的理解
## step 2 tranformer 架构
- [x] transformer 基本框架和架构都是结构的问题，说明起来有什么不同吗（要pre的话要不并在一起讲吧）
	- [ ] 记得包括预训练流程

- [ ] Concepts/MLM more approaches
	- [ ] Next Sentence Prediction
	- [ ] reconstruct corrupted sentence
- [ ] Encoder - Decoder: seems a simple conbination

- [ ] pre-training vs. post training: have general methods for post-training but specific approaches for pre-training according to the architecture itself?
- [ ] comparation: 还有其它原因吗
## step 3 llama3
- [ ] 没细看
- [ ] 报告里到处找不到 `max-length`
- [ ] 了解更多预处理和infrastructure

## Pending
- [x] The following section seems to have mentioned this as well, where's different -  comprehensive the basic architecture of the transformers
- [ ] 记得看一下 flu shot learning 里面的链接的别的学习资料, and check with this one