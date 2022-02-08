## This repo describes how to pre-train SimCTG on a large-scale pre-training corpus.
****
### Catalogue:
* <a href='#data_preparation'>1. Data Preparation</a>
* <a href='#train_simctg'>2. Train SimCTG</a>
* <a href='#generate_results'>3. Generate Result with English SimCTG using Different Decoding Methods</a>
    * <a href='#contrastive_search'>3.1. Contrastive Search</a>
    * <a href='#diverse_contrastive_search'>3.2. Diverse Contrastive Search</a>
    * <a href='#nucleus_sampling'>3.3. Nucleus Sampling</a>
    * <a href='#greedy_search'>3.4. Greedy Search</a>
    * <a href='#beam_search'>3.5. Beam Search</a>


****
<span id='data_preparation'/>

#### 1. Data Preparation:
To download the pre-training Wikipedia corpus, please follow the instructions [[here]](https://github.com/yxuansu/SimCTG/tree/main/data).

> **** The dataset contains the following files:

    .
    ├── wikipedia                   
        ├── train_english_wikipedia.txt       # Training set
        └── dev_english_wikipedia.txt         # Validation set

**Data Format**: In the files, each line represents a complete wikipedia article.

****

<span id='train_simctg'/>

#### 2. Train SimCTG:
To train a SimCTG model on LCCC dataset, please run the following commands:
```yaml
chmod +x ./train.sh
./train.sh
```
The arguments are as follows:
* `--model_name`: The name of huggingface pre-trained gpt model (e.g. gpt2).
* `--train_path`: The file path of training set.
* `--dev_path`: The file path of validation set.
* `--margin`: The contrastive margin $\rho$.
* `--seqlen`: The length of every training sample.
* `--number_of_gpu`: The number of available GPUs.
* `--batch_size_per_gpu`: The batch size for each GPU.
* `--gradient_accumulation_steps`: How many forward computations between two gradient updates.
* `--effective_batch_size`: The overall batch size. It equals to batch_size_per_gpu x gradient_accumulation_steps x number_of_gpu.
* `--total_steps`: The number of total gradient update steps.
* `--print_every`: Have many steps to show the intermediate results.
* `--save_every`: How many steps to save one checkpoint.
* `--learning_rate`: The learning rate.
* `--save_path_prefix`: Where to save the checkpoints.

****
<span id='generate_results'/>

#### 3. Generate Result with English SimCTG using Different Decoding Methods:
Here, we show how to use SimCTG to generate dialogue response with different decoding methods.
```python
import torch
from simctgdialogue import SimCTGDialogue
# load model
model_path = r'cambridgeltl/simctg_lccc'
eos_token, pad_token = '[SEP]', '[PAD]'
model = SimCTGDialogue(model_path, eos_token, pad_token)
tokenizer = model.tokenizer
model.eval()

# prepare dailogue context
context_list = ['都有什么好玩的哇', '没啥好玩的、一点儿意思都没有', '那跟沈阳差不多，还是大连好']
```
<span id='contrastive_search'/>

##### 3.1. Contrastive Search:
```python
# use contrastive search to generate the result
beam_width, alpha, decoding_len = 3, 0.6, 64
print (model.contrastive_search(context_list, beam_width, alpha, decoding_len))
# '哈哈，我觉得沈阳比大连好玩多了'
```
The arguments are as follows:
* `--context_list`: A list of utterances, e.g. [utterance_1, utterance_2, ..., utterance_n].
* `--beam_width`: k in the contrastive search, which is typically set within the range of [3, 10].
* `--alpha`: alpha in the contrastive search, which is typically set within the range of [0.5, 0.8].
* `--decoding_len`: Number of tokens to generate.

<span id='diverse_contrastive_search'/>

##### 3.2. Diverse Contrastive Search:
We also provide a stochastic version of contrastive search which can generate diverse results by combining nucleus sampling and contrastive search. More details can be found in Appendix E of the [paper]().
```python
# use diverse contrastive search to generate the result
sample_step, nucleus_p = 1, 0.95
beam_width, alpha, decoding_len = 3, 0.6, 64
print (model.diverse_contrastive_search(context_list, sample_step, nucleus_p, beam_width, alpha, decoding_len))
# '沈阳也不错啊'
```
The arguments are as follows:
* `--context_list`: A list of utterances, e.g. [utterance_1, utterance_2, ..., utterance_n].
* `--sample_step`: The number of tokens sampled with nucleus sampling at the start of generation process.
* `--nucleus_p`: The probability in nucleus sampling.
* `--beam_width`: k in the contrastive search, which is typically set within the range of [3, 10].
* `--alpha`: alpha in the contrastive search, which is typically set within the range of [0.5, 0.8].
* `--decoding_len`: The total number of tokens to generate. It equals to the number of sampled tokens + the number of tokens generated by contrastive search.

<span id='nucleus_sampling'/>

##### 3.3. Nucleus Sampling:
```python
nucleus_p, decoding_len = 0.95, 64
print (model.nucleus_sampling(context_list, nucleus_p, decoding_len))
# '你说沈阳，我就说大连了'
```
The arguments are as follows:
* `--context_list`: A list of utterances, e.g. [utterance_1, utterance_2, ..., utterance_n].
* `--nucleus_p`: The probability in nucleus sampling.
* `--decoding_len`: Number of tokens to generate.

<span id='greedy_search'/>

##### 3.4. Greedy Search:
```python
decoding_len = 64
print (model.greedy_search(context_list, decoding_len))
# '我也觉得'
```
The arguments are as follows:
* `--context_list`: A list of utterances, e.g. [utterance_1, utterance_2, ..., utterance_n].
* `--decoding_len`: Number of tokens to generate.

<span id='beam_search'/>

##### 3.5. Beam Search:
```python
beam_width, decoding_len = 10, 64
print (model.beam_search(context_list, beam_width, decoding_len))
# '哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈'
```
The arguments are as follows:
* `--context_list`: A list of utterances, e.g. [utterance_1, utterance_2, ..., utterance_n].
* `--beam_width`: The beam width of beam search.
* `--decoding_len`: Number of tokens to generate.

