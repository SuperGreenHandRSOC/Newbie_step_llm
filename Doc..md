## Architecture (Step 1 & 2)
### Overview
- about Vanilla transformer
	- 6 encoder-layers
		- 2 sub-layers each: multi-head self-attention layer, position-wise fully connected feed-forward network
	- 6 decoder-layers
		- 3 sub-layer: masked multi-head self-attention + encoder
- **3 classifications** according to encoder decoder schma
	- **Encoder-only**
		- **attention layers can access all the words in the initial sentences**
		- BERT
		- good for: full sentences NLU
	- **Decoder-only**
		- alias: auto-regressive models
		- **attention layers can only access the words positioned before that in the sentences**
		- GPT
		- good for: text generation
	- **Encoder-Decoder**: the original Transformer for machine translation
		- alias: sequence-to-sequence model
		- **encoder can access all the words in the initial sentences while decoder only access the words positions before a given word in the input**
		- good for: generating new sentences conditioned on a given input

![[attachments/Pasted image 20240718202812.png#C|400x400]]

### Basic Idea
#### Attention
Summary: 相关信息的加权和
##### Seq2seq Model with Attention layer (基操)

![[attachments/Pasted image 20230828115138.png#C|400]]
Attention Embedding: weighted sum of every hidden state from the encoder, weighted based on the decoder hidden state.

1. calculating **alignment scores** -- similarity between the decoder hidden state (previous hidden state in the decoder) and **each** encoder hidden state $$\large e_{ij} = v_a^\top \tanh{\left(W_a s_{i-1} + U_a h_j\right)}$$Note that the dimension of decoder state and encode states are different because there are several encoder states but only **one** hidden state from the decoder. Three weight matrices there for computing alignment scores: $\mathbf v_a$,$\mathbf W_a$ and $\mathbf U_a$.
2. apply SoftMax function to get weight matrix that really determine the influences from encoder hidden states. `weight = softmax(e_ij)`
3. multiply the encoder states with the corresponding weight `weighted_score = encoder_states * weight`
4. Sum up weighted alignment vectors (weighted sum) to get the context vector and return it. `context = np.sum(weighted_scores, axis=0)`

##### QKV Attention
- **Q - query**: matched to a key
- **K - key**: matched to a query
- **V - value**: assigned to a key
- **Alignment**: the similarity of a Q-K pair, computed by the model

![[../../Courses/into_NLP/NLP_first_insight_Andrew/4. with Attention model/attachments/Pasted image 20230828164128.png#C|500]]
1. **MatMul**: multiply packed queries with packed keys to get alignment scores.
2. **Scale (Regularization)**: using the square root of the *key vector dimension*
3. **Mask**
4. **Softmax**: sum to one
5. **MatMul**: get the attention vectors for each query

##### Multi-head (QKV) Attention
a head can learn different relationships between word from another head

![[attachments/Pasted image 20240902161603.png]]

1. process Q, K, V with some linear layers to get different sets of them.
2. perform QKV in parallel to multiple sets and get multiple heads, which mean **multiple representations of the attention information**
3. **concatenate** them and then multiply again by a matrix with dimention *dim-of-each-heads by num heads --- dim of each head*

- Multi-Headed models attend to information from different representations
- Parallel computations
- Similar computational cost to single-head attention

![[attachments/Pasted image 20230831211448.png|300]]
##### Masked self-attention / casual attention
- Here we have 3 main ways of Attention as following:
	- in **Encoder-Decoder Attention**:
		- Queries from one sentence, keys and values from another. 
		- E.g. from Franch and English
	- in **Self-Attention**:
		- Queries, keys and values come from the same sentence.
		- the weight matrix - Meaning of each word within the sentence, contextual representations of the words
	- in **Masked Self-Attention**:
		- Queries, keys and values come from the same sentence. **Queries don't attend to future positions.**
		- predictions at each position depend only on the known outputs
		- to implement Masked Attention we only need to **add a mask** in the **softmax function.**


![[attachments/Pasted image 20230831180051.png#|200]]![[attachments/Pasted image 20230831180108.png|200]]![[attachments/Pasted image 20230831180124.png|200]]


![[attachments/Pasted image 20230831180603.png#C|500]]
##### Coding: Attention
#### Encoder & Decoder
About the access to the input, the difference between encoder and decoder oriented in the different attention machanisms they use. Encoder mainly uses fully visible attention, and decoder mainly uses causal attention (masked self-attention).
![[attachments/Pasted image 20230904231941.png#C|500]]
![[attachments/Pasted image 20230903103645.png#C|500]]
- **left**: basic **encoder-decoder representation**. We have fully **visible attention** in the encoder and then casual attention in the decoder. ***<-Best btw ?***
- **middle**: Language model, which consists of a single transformer layer stack. use **causal masking** throughout because it's being fed the concatenation of the inputs and the target. Like GPT2, CTRL.
- **Right**: prefix language model which corresponds to allowing fully **visible masking** over the inputs and **casual masking** in the rest. like UniLM

From experiments scientists noticed that the encoder-decoder structure outperforms the others. So it is also the structure of T5.
### Comprehensive version
#### Encoder-only: BERT
- 3 modules
	- a embedding module (to embedding vector)
	- a stack of Transformer encoders (to contextual representation vectors)
	- a fully connected layers (to one-hot)
- **input object**
	- **position embeddings**
	- **segment embeddings**: which sentence is the word belonged
	- **token embeddings**: special tokens `<CLS>`, `<EOS>`, `<SEP>`
	- Sum up the embeddings to get an input 
		- *Note*: Through extensive training process on a massive dataset, the model will be able to extract all the information without loss. Just imagine a complex high-dimensional, huge vector space. Embeddings won't mess up.
	- Trough the transformer blocks to get $T_1$ to $T_N$, $T_1'$ to $T_M'$

- **output visualization**
	- can predict the masked word via a simple softmax 
	- The `C` token indicates which kind of tasks is going to be performed

![[../../../../wandering/attachments/Pasted image 20230902231142.png#C|400]]

##### Train a BERT
- 2 steps in the BERT framework
	- **pre-training**
		- unlabeled data
		- different pre-training tasks.
		- 2 tasks
			- MLM
			- Next Sentence Prediction
	-  fine-tuning
		- labeled data from the downstream tasks


###### MLM
- Choose 15% of the tokens at random: mask them 80% of the time, replace them with a random token 10% of the time, or keep as is 10% of the time. 
- There could be multiple masked spans in a sentence
- Next sentence prediction is also used when pre-training.
- Loss function: Cross Entropy Loss

E.g: To a sentence *my dog is cute*
- "my dog is \[MASK\]" with probability 80%,
- "my dog is happy" with probability 10%,
- "my dog is cute" with probability 10%.
![[attachments/Pasted image 20230903104401.png|500]]


#### Decoder-only: GPT
![[attachments/Pasted image 20240718153418.png]]
##### History: How GPTs make progress
https://iq.opengenus.org/gpt2-vs-gpt3-vs-gpt35-vs-gpt4/
##### Train a GPT
#### Encoder-Decoder
![[attachments/Pasted image 20240718202812.png#C|400x400]]
#### Comparation
#### Encoder-only vs. have-a-Decoder
##### Architecture
Encoder: Fully visible masking
Decoder: Casual masking

Encoder-only models like BERT are designed to understand the input text as a whole rather than sequentially generating new tokens. They lack the ability to model time series or predict the next word in a sequence, which is essential for text generation tasks. Since these models only consist of encoders, they struggle with generating text, especially when there isn't a prompt guiding the generation process.
##### Pre-training
MLM （填空）相比起 next token prediction （下一个）不擅长做生成任务
#### Decoder-only vs. have-a-Encoder
https://www.zhihu.com/question/588325646
模型更简单，更容易做到并行处理

### Coding
## Training Decoders (Step 3)
### Overall
- What's important?
	- Data
	- Scale
	- Managing complexity: exploiting the parameters

- Pre-training: learning language & gaining knowledge
	- tokenization the dataset
	- next-token prediction
- Post-training: follow instructions & tool-use
	- SFT on DPO
- Extentions: multimodal (speech & images)
	- Multi-modal encoder pre-training 
	- Adapters
### Pre-training
- **max-length**: 生成最大长度
	- 限制生成文本长度，适应特定场景需要、鼓励模型减少信息冗余
- **warm up**
	- 用较大的学习率迅速进行早期的梯度下降，提高训练效率

### Post-training
Instruction tuning
https://www.ibm.com/topics/instruction-tuning

