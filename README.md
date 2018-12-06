<h1 align="center">
  bert-as-service
</h1>

<p align="center">Using BERT model as a sentence encoding service, i.e. mapping a variable-length sentence to a fixed-length vector.</p>

<p align="center">
  <a href="https://github.com/hanxiao/bert-as-service/stargazers">
    <img src="https://img.shields.io/github/stars/hanxiao/bert-as-service.svg"
         alt="GitHub stars">
  </a>
  <a href="https://github.com/hanxiao/bert-as-service/releases">
      <img src="https://img.shields.io/github/release/hanxiao/bert-as-service.svg"
           alt="GitHub release">
  </a>
  <a href="https://github.com/hanxiao/bert-as-service/issues">
        <img src="https://img.shields.io/github/issues/hanxiao/bert-as-service.svg"
             alt="GitHub issues">
  </a>
  <img src="https://img.shields.io/badge/Python->=3.6-brightgreen.svg" alt="Python: >=3.6">
  <img src="https://img.shields.io/badge/Tensorflow->=1.10-brightgreen.svg" alt="Tensorflow: >=1.10">
  <a href="https://github.com/hanxiao/bert-as-service/blob/master/LICENSE">
        <img src="https://img.shields.io/github/license/hanxiao/bert-as-service.svg"
             alt="GitHub license">
  </a>
  <a href="https://twitter.com/intent/tweet?text=Wow:&url=https%3A%2F%2Fgithub.com%2Fhanxiao%2Fbert-as-service">
  <img src="https://img.shields.io/twitter/url/https/github.com/hanxiao/bert-as-service.svg?style=social" alt="Twitter">
  </a>      
</p>

<p align="center">
  <a href="#highlights">Highlights</a> •
  <a href="#what-is-it">What is it</a> •
  <a href="#requirements">Requirements</a> •
  <a href="#usage">Usage</a> •
  <a href="#faq">FAQ</a> •
  <a href="#benchmark">Benchmark</a> •
  <a href="#advance-usage">Advance Usage</a>
</p>

<p align="center">
    <img src=".github/demo.gif" width="700">
</p>

<h6 align="center">Made by Han Xiao • 
 <a href="https://hanxiao.github.io">:globe_with_meridians: https://hanxiao.github.io</a></h6>

## What is it

**BERT**: [Developed by Google](https://github.com/google-research/bert), BERT is a method of pre-training language representations. It leverages an enormous amount of plain text data publicly available on the web and is trained in an unsupervised manner. Pre-training a BERT model is a fairly expensive yet one-time procedure for each language. Fortunately, Google released several pre-trained models where [you can download from here](https://github.com/google-research/bert#pre-trained-models).


**Sentence Encoding/Embedding**: sentence encoding is a upstream task required in many NLP applications, e.g. sentiment analysis, text classification. The goal is to represent a variable length sentence into a fixed length vector, e.g. `hello world` to `[0.1, 0.3, 0.9]`. Each element of the vector should "encode" some semantics of the original sentence.

**Finally, this repo**: This repo uses BERT as the sentence encoder and hosts it as a service via ZeroMQ, allowing you to map sentences into fixed-length representations in just two lines of code. 

## Highlights

- :telescope: **State-of-the-art**: build on pretrained 12/24-layer BERT models released by Google AI, which is considered as a milestone in the NLP community.
- :hatching_chick: **Easy-to-use**: require only two lines of code to get sentence encodes.
- :zap: **Fast**: 900 sentences/s on a single Tesla M40 24GB with `max_seq_len=20`. See [benchmark](#Benchmark).
- :octopus: **Scalable**: scale nicely and smoothly on multiple GPUs and multiple clients without worrying about concurrency. See [benchmark](#speed-wrt-num_client).

## Install
You can install the server and client *separately* or even on *different* machines via:
```bash
pip install -U bert-serving-server  # install server
pip install -U bert-serving-client  # install client
```

Note that the server MUST be run on Python >= 3.5 and Tensorflow >= 1.10 (*one-point-ten*). The server does not support Python 2.

:point_up: The client can be run on both Python 2 and 3 [for the following consideration](#q-can-i-run-it-in-python-2).

## Usage

#### 1. Download a Pre-trained BERT Model
Download a model listed below, then uncompress the zip file into some folder, say `/tmp/english_L-12_H-768_A-12/`

<details>
 <summary>List of released pretrained BERT models (click to expand)</summary>


<table>
<tr><td><a href="https://storage.googleapis.com/bert_models/2018_10_18/uncased_L-12_H-768_A-12.zip">BERT-Base, Uncased</a></td><td>12-layer, 768-hidden, 12-heads, 110M parameters</td></tr>
<tr><td><a href="https://storage.googleapis.com/bert_models/2018_10_18/uncased_L-24_H-1024_A-16.zip">BERT-Large, Uncased</a></td><td>24-layer, 1024-hidden, 16-heads, 340M parameters</td></tr>
<tr><td><a href="https://storage.googleapis.com/bert_models/2018_10_18/cased_L-12_H-768_A-12.zip">BERT-Base, Cased</a></td><td>12-layer, 768-hidden, 12-heads , 110M parameters</td></tr>
<tr><td><a href="https://storage.googleapis.com/bert_models/2018_10_18/cased_L-24_H-1024_A-16.zip">BERT-Large, Cased</a></td><td>24-layer, 1024-hidden, 16-heads, 340M parameters</td></tr>
<tr><td><a href="https://storage.googleapis.com/bert_models/2018_11_23/multi_cased_L-12_H-768_A-12.zip">BERT-Base, Multilingual Cased (New)</a></td><td>104 languages, 12-layer, 768-hidden, 12-heads, 110M parameters</td></tr>
<tr><td><a href="https://storage.googleapis.com/bert_models/2018_11_03/multilingual_L-12_H-768_A-12.zip">BERT-Base, Multilingual Cased (Old)</a></td><td>102 languages, 12-layer, 768-hidden, 12-heads, 110M parameters</td></tr>
<tr><td><a href="https://storage.googleapis.com/bert_models/2018_11_03/chinese_L-12_H-768_A-12.zip">BERT-Base, Chinese</a></td><td>Chinese Simplified and Traditional, 12-layer, 768-hidden, 12-heads, 110M parameters</td></tr>
</table>

</details>


> **Optional:** fine-tuning the model on your downstream task. [Why is it optional?](#q-are-you-suggesting-using-bert-without-fine-tuning)

#### 2. Start a BERT service
After installing the server, you should be able to use `bert-serving-start` CLI as follows:
```bash
bert-serving-start -model_dir /tmp/english_L-12_H-768_A-12/ -num_worker=4 
```
This will start a service with four workers, meaning that it can handle up to four **concurrent** requests. More concurrent requests will be queued in a load balancer. Details can be found in our [FAQ](#q-what-is-the-parallel-processing-model-behind-the-scene) and [the benchmark on number of clients](#speed-wrt-num_client)

<details>
 <summary>Start a BERT Service in a Docker Container (click to expand)</summary>

One may also run BERT Service in a container:

```bash
docker build -t bert-as-service -f ./docker/Dockerfile .
NUM_WORKER=1
PATH_MODEL=<path of your model>
docker run --runtime nvidia -dit -p 5555:5555 -p 5556:5556 -v $PATH_MODEL:/model -t bert-as-service $NUM_WORKER
```
</details>


#### 3. Use Client to Get Sentence Encodes
Now you can use BERT to encode sentences simply as follows:
```python
from bert_serving.client import BertClient
bc = BertClient()
bc.encode(['First do it', 'then do it right', 'then do it better'])
```
It will return a `ndarray`, in which each row is the fixed representation of a sentence. You can also let it return a pure python object with type `List[List[float]]`.

As a feature of BERT, you may get encodes of a pair of sentences by concatenating them with ` ||| `, e.g.
```python
bc.encode(['First do it ||| then do it right'])
```

Getting the token-based embedding [is also possible](#q-how-can-i-get-word-embedding-instead-of-sentence-embedding).

#### Use BERT Service Remotely
One may also start the service on one (GPU) machine and call it from another (CPU) machine as follows:

```python
# on another CPU machine
from bert_serving.client import BertClient
bc = BertClient(ip='xx.xx.xx.xx')  # ip address of the GPU machine
bc.encode(['First do it', 'then do it right', 'then do it better'])
```

> :bulb: **Checkout some advance usages below:**
> - [Using `BertClient` with `tf.data` API](#using-bertclient-with-tfdata-api)
> - [Building a text classifier using BERT features and `tf.estimator` API](#building-a-text-classifier-using-bert-features-and-tfestimator-api)
> - [Save to and load from TFRecord data](#save-to-and-load-from-tfrecord-data)
> - [Asynchronous encoding](#asynchronous-encoding)
> - [Broadcasting to multiple clients](#broadcasting-to-multiple-clients)

 
## Server and Client Configurations

### Server-side configs

The server-side is a CLI called `bert-serving-start`, you can specify its arguments via:
```bash
bert-serving-start -model_dir [-max_seq_len] [-num_worker] [-max_batch_size] [-port] [-port_out] [-pooling_strategy] [-pooling_layer]
```

| Argument | Type | Default | Description |
|--------------------|------|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `model_dir` | str |  | folder path of the pre-trained BERT model. |
| `max_seq_len` | int | `25` | maximum length of sequence, longer sequence will be trimmed on the right side. |
| `num_worker` | int | `1` | number of (GPU/CPU) worker runs BERT model, each works in a separate process. |
| `max_batch_size` | int | `256` | maximum number of sequences handled by each worker, larger batch will be partitioned into small batches. |
| `port` | int | `5555` | port for pushing data from client to server |
| `port_out` | int | `5556`| port for publishing results from server to client |
| `pooling_strategy` | str | `REDUCE_MEAN` | the pooling strategy for generating encoding vectors, valid values are `NONE`, `REDUCE_MEAN`, `REDUCE_MAX`, `REDUCE_MEAN_MAX`, `CLS_TOKEN`, `FIRST_TOKEN`, `SEP_TOKEN`, `LAST_TOKEN`. Explanation of these strategies [can be found here](#q-what-are-the-available-pooling-strategies). To get encoding for each token in the sequence, please set this to `NONE`.|
| `pooling_layer` | int | `-2` | the encoding layer that pooling operates on, where `-1` means the last layer, `-2` means the second-to-last, etc.|
| `gpu_memory_fraction` | float | `0.5` | the fraction of the overall amount of memory that each GPU should be allocated per worker |

### Client-side configs

Client-side configs are summarized below, which can be found in [`client.py`](service/client.py) as well.
 
| Argument | Type | Default | Description |
|----------------------|------|-----------|-------------------------------------------------------------------------------|
| `ip` | str | `localhost` | IP address of the server |
| `port` | int | `5555` | port for pushing data from client to server, *must be consistent with the server side config* |
| `port_out` | int | `5556`| port for publishing results from server to client, *must be consistent with the server side config* |
| `output_fmt` | str | `ndarray` | the output format of the sentence encodes, either in numpy array or python List[List[float]] (`ndarray`/`list`) |
| `show_server_config` | bool | `True` | whether to show server configs when first connected |


## FAQ

##### **Q:** Where is the BERT code come from?

**A:** [BERT code of this repo](server/bert_serving/server/bert/) is forked from the [original BERT repo](https://github.com/google-research/bert) with necessary modification, [especially in extract_features.py](server/bert_serving/server/bert/extract_features.py).

##### **Q:** How large is a sentence vector?
In general, each sentence is translated to a 768-dimensional vector. Depending on the pretrained BERT you are using, `pooling_strategy` and `pooling_layer` the dimensions of the output vector could be different. 

##### **Q:** How do you get the fixed representation? Did you do pooling or something?

**A:** Yes, pooling is required to get a fixed representation of a sentence. In the default strategy `REDUCE_MEAN`, I take the second-to-last hidden layer of all of the tokens in the sentence and do average pooling.

##### **Q:** Are you suggesting using BERT without fine-tuning?

**A:** Yes and no. On the one hand, Google pretrained BERT on Wikipedia data, thus should encode enough prior knowledge of the language into the model. Having such feature is not a bad idea. On the other hand, these prior knowledge is not specific to any particular domain. It should be totally reasonable if the performance is not ideal if you are using it on, for example, classifying legal cases. Nonetheless, you can always first fine-tune your own BERT on the downstream task and then use `bert-as-service` to extract the feature vectors efficiently. Keep in mind that `bert-as-service` is just a feature extraction service based on BERT. Nothing stops you from using a fine-tuned BERT.

##### **Q:** Can I get a concatenation of several layers instead of a single layer ?

**A:** Sure! Just use a list of the layer you want to concatenate when calling the server. Example:

```bash
bert_serving_start -pooling_layer -4 -3 -2 -1 -model_dir /tmp/english_L-12_H-768_A-12/
```

##### **Q:** What are the available pooling strategies?

**A:** Here is a table summarizes all pooling strategies I implemented. Choose your favorite one by specifying `bert_serving_start -pooling_strategy`.

|Strategy|Description|
|---|---|
| `NONE` | no pooling at all, useful when you want to use word embedding instead of sentence embedding. This will results in a `[max_seq_len, 768]` encode matrix for a sequence.|
| `REDUCE_MEAN` | take the average of the hidden state of encoding layer on the time axis |
| `REDUCE_MAX` | take the maximum of the hidden state of encoding layer on the time axis |
| `REDUCE_MEAN_MAX` | do `REDUCE_MEAN` and `REDUCE_MAX` separately and then concat them together on the last axis, resulting in 1536-dim sentence encodes |
| `CLS_TOKEN` or `FIRST_TOKEN` | get the hidden state corresponding to `[CLS]`, i.e. the first token |
| `SEP_TOKEN` or `LAST_TOKEN` | get the hidden state corresponding to `[SEP]`, i.e. the last token |

##### **Q:** Why not use the hidden state of the first token as default strategy, i.e. the `[CLS]`?

**A:** Because a pre-trained model is not fine-tuned on any downstream tasks yet. In this case, the hidden state of `[CLS]` is not a good sentence representation. If later you fine-tune the model, you may use `[CLS]` as well.

##### **Q:** BERT has 12/24 layers, so which layer are you talking about?

**A:** By default this service works on the second last layer, i.e. `pooling_layer=-2`. You can change it by setting `pooling_layer` to other negative values, e.g. -1 corresponds to the last layer.

##### **Q:** Why not the last hidden layer? Why second-to-last?

**A:** The last layer is too closed to the target functions (i.e. masked language model and next sentence prediction) during pre-training, therefore may be biased to those targets. If you question about this argument and want to use the last hidden layer anyway, please feel free to set `pooling_layer=-1`.

##### **Q:** Could I use other pooling techniques?

**A:** For sure. Just follows [`get_sentence_encoding()` I added to the modeling.py](server/bert_serving/server/bert/extract_features.py#L96). Note that, if you introduce new `tf.variables` to the graph, then you need to train those variables before using the model. You may also want to check [some pooling techniques I mentioned in my blog post](https://hanxiao.github.io/2018/06/24/4-Encoding-Blocks-You-Need-to-Know-Besides-LSTM-RNN-in-Tensorflow/#pooling-block).

##### **Q:** Can I start multiple clients and send requests to one server simultaneously?

**A:** Yes! That's the purpose of this repo. In fact you can start as many clients as you want. One server can handle all of them (given enough time).

##### **Q:** How many requests can one service handle concurrently?

**A:** The maximum number of concurrent requests is determined by `num_worker` in `bert_serving_start`. If you a sending more than `num_worker` requests concurrently, the new requests will be temporally stored in a queue until a free worker becomes available.

##### **Q:** So one request means one sentence?

**A:** No. One request means a list of sentences sent from a client. Think the size of a request as the batch size. A request may contain 256, 512 or 1024 sentences. The optimal size of a request is often determined empirically. One large request can certainly improve the GPU utilization, yet it also increases the overhead of transmission. You may run `python example1.py` for a simple benchmark.

##### **Q:** How about the speed? Is it fast enough for production?

**A:** It highly depends on the `max_seq_len` and the size of a request. On a single Tesla M40 24GB with `max_seq_len=40`, you should get about 470 samples per second using a 12-layer BERT. In general, I'd suggest smaller `max_seq_len` (25) and larger request size (512/1024).

##### **Q:** Did you benchmark the efficiency?

**A:** Yes. See [Benchmark](#Benchmark).

To reproduce the results, please run [`python benchmark.py`](benchmark.py).

##### **Q:** What is backend based on?

**A:** [ZeroMQ](http://zeromq.org/).

##### **Q:** What is the parallel processing model behind the scene?

<img src=".github/bert-parallel-pipeline.png" width="600">

##### **Q:** Why does the server need two ports?
One port is for pushing text data into the server, the other port is for publishing the encoded result to the client(s). In this way, we get rid of back-chatter, meaning that at every level recipients never talk back to senders. The overall message flow is strictly one-way, as depicted in the above figure. Killing back-chatter is essential to real scalability, allowing us to use `BertClient` in an asynchronous way. 

##### **Q:** Do I need Tensorflow on the client side?

**A:** No. Think of `BertClient` as a general feature extractor, whose output can be fed to *any* ML models, e.g. `scikit-learn`, `pytorch`, `tensorflow`. The only file that client need is [`client.py`](service/client.py). Copy this file to your project and import it, then you are ready to go.

##### **Q:** Can I use multilingual BERT model provided by Google?

**A:** Yes.

##### **Q:** Can I use my own fine-tuned BERT model?

**A:** Yes. In fact, this is suggested. Make sure you have the following three items in `model_dir`:
                             
- A TensorFlow checkpoint (`bert_model.ckpt`) containing the pre-trained weights (which is actually 3 files).
- A vocab file (`vocab.txt`) to map WordPiece to word id.
- A config file (`bert_config.json`) which specifies the hyperparameters of the model.

##### **Q:** Can I run it in python 2?

**A:** Server side no, client side yes. This is based on the consideration that python 2.x might still be a major piece in some tech stack. Migrating the whole downstream stack to python 3 for supporting `bert-as-service` can take quite some effort. On the other hand, setting up `BertServer` is just a one-time thing, which can be even [run in a docker container](#run-bert-service-on-nvidia-docker). To ease the integration, we support python 2 on the client side so that you can directly use `BertClient` as a part of your python 2 project, whereas the server side should always be hosted with python 3.

##### **Q:** How can I get word embedding instead of sentence embedding?

**A:** To get word embedding please set `pooling_strategy = NONE`. This will omit the pooling operation on the encoding layer, resulting in a `[max_seq_len, 768]` matrix for every sequence. To get the word embedding corresponds to every token, you can simply use slice index.

> :children_crossing: NOTE: no matter how long your original sequence is, the service will always return a `[max_seq_len, 768]` matrix for every sequence. Beware of the special tokens padded to the sequence, e.g. `[CLS]`, `[SEP]`, `0_PAD`, when getting the word embedding.

Example:
```python
# max_seq_len = 25
# pooling_strategy = NONE

bc = BertClient()
x = ['hey you', 'whats up?']

bc.encode(x)  # [2, 25, 768]
bc.encode(x)[0]  # [1, 25, 768], sentence embeddings for `hey you`
bc.encode(x)[0][0]  # [1, 1, 768], word embedding for `[CLS]`
bc.encode(x)[0][1]  # [1, 1, 768], word embedding for `hey`
bc.encode(x)[0][2]  # [1, 1, 768], word embedding for `you`
bc.encode(x)[0][3]  # [1, 1, 768], word embedding for `[SEP]`
bc.encode(x)[0][4]  # [1, 1, 768], word embedding for padding symbol
bc.encode(x)[0][25]  # error, out of index!
```

##### **Q:** Do I need to do segmentation for Chinese?

No, if you are using [the pretrained Chinese BERT released by Google](https://github.com/google-research/bert#pre-trained-models) you don't need word segmentation. As this Chinese BERT is character-based model. It won't recognize word/phrase even if you intentionally add space in-between. To see that more clearly, this is what the BERT model actually receives after tokenization:

```python
bc.encode(['hey you', 'whats up?', '你好么？', '我 还 可以'])
```

```
tokens: [CLS] hey you [SEP]
input_ids: 101 13153 8357 102 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
input_mask: 1 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0

tokens: [CLS] what ##s up ? [SEP]
input_ids: 101 9100 8118 8644 136 102 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
input_mask: 1 1 1 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0

tokens: [CLS] 你 好 么 ？ [SEP]
input_ids: 101 872 1962 720 8043 102 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
input_mask: 1 1 1 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0

tokens: [CLS] 我 还 可 以 [SEP]
input_ids: 101 2769 6820 1377 809 102 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
input_mask: 1 1 1 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
```

That means the word embedding is actually the character embedding for Chinese-BERT.


##### **Q:** Why my (English) word is tokenized to `##something`?

Because your word is out-of-vocabulary (OOV). The tokenizer from Google uses a greedy longest-match-first algorithm to perform tokenization using the given vocabulary.

For example:
```python
input = "unaffable"
tokenizer_output = ["un", "##aff", "##able"]
```

##### **Q:** I encounter `zmq.error.ZMQError: Operation cannot be accomplished in current state` when using `BertClient`, what should I do?

**A:** This is often due to the misuse of `BertClient` in multi-thread/process environment. Note that you can’t reuse one `BertClient` among multiple threads/processes, you have to make a separate instance for each thread/process. For example, the following won't work at all:

```python
# BAD example
bc = BertClient()

# in Proc1/Thread1 scope:
bc.encode(lst_str)

# in Proc2/Thread2 scope:
bc.encode(lst_str)
```

Instead, please do:

```python
# in Proc1/Thread1 scope:
bc1 = BertClient()
bc1.encode(lst_str)

# in Proc2/Thread2 scope:
bc2 = BertClient()
bc2.encode(lst_str)
```

##### **Q:** The cosine similarity of two sentence vectors is unreasonably high (e.g. always > 0.8), what's wrong?

**A:** A decent representation for a downstream task doesn't mean that it will be meaningful in terms of cosine distance. Since cosine distance is a linear space where all dimensions are weighted equally. if you want to use cosine distance anyway, then please focus on the rank not the absolute value. Namely, do not use:
```
if cosine(A, B) > 0.9, then A and B are similar
```
Please consider the following instead:
```
if cosine(A, B) > cosine(A, C), then A is more similar to B than C.
```

##### **Q:** I'm getting bad performance, what should I do?

**A:** This often suggests that the pretrained BERT could not generate a descent representation of your downstream task. Thus, you can fine-tune the model on the downstream task and then use `bert-as-service` to serve the fine-tuned BERT. Note that, `bert-as-service` is just a feature extraction service based on BERT. Nothing stops you from using a fine-tuned BERT.

## Benchmark

The primary goal of benchmarking is to test the scalability and the speed of this service, which is crucial for using it in a dev/prod environment. Benchmark was done on Tesla M40 24GB, experiments were repeated 10 times and the average value is reported.

To reproduce the results, please run
```bash
python benchmark.py
```

Common arguments across all experiments are:

| Parameter         | Value |
|-------------------|-------|
| num_worker        | 1,2,4 |
| max_seq_len       | 40    |
| client_batch_size | 2048  |
| max_batch_size    | 256   |
| num_client        | 1     |

#### Speed wrt. `max_seq_len`

`max_seq_len` is a parameter on the server side, which controls the maximum length of a sequence that a BERT model can handle. Sequences larger than `max_seq_len` will be truncated on the left side. Thus, if your client want to send long sequences to the model, please make sure the server can handle them correctly.

Performance-wise, longer sequences means slower speed and  more chance of OOM, as the multi-head self-attention (the core unit of BERT) needs to do dot products and matrix multiplications between every two symbols in the sequence.

<img src=".github/max_seq_len.png" width="600">

| `max_seq_len` | 1 GPU | 2 GPU | 4 GPU |
|---------------|-------|-------|-------|
| 20            | 903   | 1774  | 3254  |
| 40            | 473   | 919   | 1687  |
| 80            | 231   | 435   | 768   |
| 160           | 119   | 237   | 464   |
| 320           | 54    | 108   | 212   |

#### Speed wrt. `client_batch_size`

`client_batch_size` is the number of sequences from a client when invoking `encode()`. For performance reason, please consider encoding sequences in batch rather than encoding them one by one. 

For example, do:
```python
# prepare your sent in advance
bc = BertClient()
my_sentences = [s for s in my_corpus.iter()]
# doing encoding in one-shot
vec = bc.encode(my_sentences)
```

DON'T:
```python
bc = BertClient()
vec = []
for s in my_corpus.iter():
    vec.append(bc.encode(s))
```

It's even worse if you put `BertClient()` inside the loop. Don't do that.

<img src=".github/client_batch_size.png" width="600">

| `client_batch_size` | 1 GPU | 2 GPU | 4 GPU |
|---------------------|-------|-------|-------|
| 1                   | 75    | 74    | 72    |
| 4                   | 206   | 205   | 201   |
| 8                   | 274   | 270   | 267   |
| 16                  | 332   | 329   | 330   |
| 64                  | 365   | 365   | 365   |
| 256                 | 382   | 383   | 383   |
| 512                 | 432   | 766   | 762   |
| 1024                | 459   | 862   | 1517  |
| 2048                | 473   | 917   | 1681  |
| 4096                | 481   | 943   | 1809  |



#### Speed wrt. `num_client`
`num_client` represents the number of concurrent clients connected to the server at the same time.

<img src=".github/num_clients.png" width="600">

| `num_client` | 1 GPU | 2 GPU | 4 GPU |
|--------------|-------|-------|-------|
| 1            | 473   | 919   | 1759  |
| 2            | 261   | 512   | 1028  |
| 4            | 133   | 267   | 533   |
| 8            | 67    | 136   | 270   |
| 16           | 34    | 68    | 136   |
| 32           | 17    | 34    | 68    |

As one can observe, 1 clients 1 GPU = 381 seqs/s, 2 clients 2 GPU 402 seqs/s, 4 clients 4 GPU 413 seqs/s. This shows the efficiency of our parallel pipeline and job scheduling, as the service can leverage the GPU time  more exhaustively as concurrent requests increase.


#### Speed wrt. `max_batch_size`

`max_batch_size` is a parameter on the server side, which controls the maximum number of samples per batch per worker. If a incoming batch from client is larger than `max_batch_size`, the server will split it into small batches so that each of them is less or equal than `max_batch_size` before sending it to workers.

<img src=".github/max_batch_size.png" width="600">

| `max_batch_size` | 1 GPU | 2 GPU | 4 GPU |
|------------------|-------|-------|-------|
| 32               | 450   | 887   | 1726  |
| 64               | 459   | 897   | 1759  |
| 128              | 473   | 931   | 1816  |
| 256              | 473   | 919   | 1688  |
| 512              | 464   | 866   | 1483  |


#### Speed wrt. `pooling_layer`

`pooling_layer` determines the encoding layer that pooling operates on. For example, in a 12-layer BERT model, `-1` represents the layer closed to the output, `-12` represents the layer closed to the embedding layer. As one can observe below, the depth of the pooling layer affects the speed.

<img src=".github/pooling_layer.png" width="600">

| `pooling_layer` | 1 GPU | 2 GPU | 4 GPU |
|-----------------|-------|-------|-------|
| [-1]            | 438   | 844   | 1568  |
| [-2]            | 475   | 916   | 1686  |
| [-3]            | 516   | 995   | 1823  |
| [-4]            | 569   | 1076  | 1986  |
| [-5]            | 633   | 1193  | 2184  |
| [-6]            | 711   | 1340  | 2430  |
| [-7]            | 820   | 1528  | 2729  |
| [-8]            | 945   | 1772  | 3104  |
| [-9]            | 1128  | 2047  | 3622  |
| [-10]           | 1392  | 2542  | 4241  |
| [-11]           | 1523  | 2737  | 4752  |
| [-12]           | 1568  | 2985  | 5303  |


## Advance Usage

### Using `BertClient` with `tf.data` API

The [`tf.data`](https://www.tensorflow.org/guide/datasets) API enables you to build complex input pipelines from simple, reusable pieces. One can also use `BertClient` to encode sentences on-the-fly and use the vectors in a downstream model. Here is an example:

```python
batch_size = 256
num_parallel_calls = 4
num_clients = num_parallel_calls * 2  # should be at least greater than `num_parallel_calls`

# start a pool of clients
bc_clients = [BertClient(show_server_config=False) for _ in range(num_clients)]


def get_encodes(x):
    # x is `batch_size` of lines, each of which is a json object
    samples = [json.loads(l) for l in x]
    text = [s['raw_text'] for s in samples]  # List[List[str]]
    labels = [s['label'] for s in samples]  # List[str]
    # get a client from available clients
    bc_client = bc_clients.pop()
    features = bc_client.encode(text)
    # after use, put it back
    bc_clients.append(bc_client)
    return features, labels


ds = (tf.data.TextLineDataset(train_fp).batch(batch_size)
        .map(lambda x: tf.py_func(get_encodes, [x], [tf.float32, tf.string]),  num_parallel_calls=num_parallel_calls)
        .map(lambda x, y: {'feature': x, 'label': y})
        .make_one_shot_iterator().get_next())
```

The trick here is to start a pool of `BertClient` and reuse them one by one. In this way, we can fully harness the power of `num_parallel_calls` of `Dataset.map()` API.  

The complete example can [be found example4.py](example4.py). There is also [an example in Keras](https://github.com/hanxiao/bert-as-service/issues/29#issuecomment-442362241). 

### Building a text classifier using BERT features and `tf.estimator` API

Following the last example, we can easily extend it to a full classifier using `tf.estimator` API. One only need minor change on the input function as follows:

```python
estimator = DNNClassifier(
    hidden_units=[512],
    feature_columns=[tf.feature_column.numeric_column('feature', shape=(768,))],
    n_classes=len(laws),
    config=run_config,
    label_vocabulary=laws_str,
    dropout=0.1)

input_fn = lambda fp: (tf.data.TextLineDataset(fp)
                       .apply(tf.contrib.data.shuffle_and_repeat(buffer_size=10000))
                       .batch(batch_size)
                       .map(lambda x: tf.py_func(get_encodes, [x], [tf.float32, tf.string]), num_parallel_calls=num_parallel_calls)
                       .map(lambda x, y: ({'feature': x}, y))
                       .prefetch(20))

train_spec = TrainSpec(input_fn=lambda: input_fn(train_fp))
eval_spec = EvalSpec(input_fn=lambda: input_fn(eval_fp), throttle_secs=0)
train_and_evaluate(estimator, train_spec, eval_spec)
```

The complete example can [be found example5.py](example5.py), in which a simple MLP is built on BERT features for predicting the relevant articles according to the fact description in the law documents. The problem is a part of the [Chinese AI and Law Challenge Competition](https://github.com/thunlp/CAIL/blob/master/README_en.md).


### Save to and load from TFRecord data
The TFRecord file format is a simple record-oriented binary format that many TensorFlow applications use for training data. You can also pre-encode all your sequences and store their encodings to a TFRecord file, then later load it to build a `tf.Dataset`. For example, to write encoding into a TFRecord file:

```python
bc = BertClient()
list_vec = bc.encode(lst_str)
list_label = [0 for _ in lst_str]  # a dummy list of all-zero labels

# write to tfrecord
with tf.python_io.TFRecordWriter('tmp.tfrecord') as writer:
    def create_float_feature(values):
        return tf.train.Feature(float_list=tf.train.FloatList(value=values))

    def create_int_feature(values):
        return tf.train.Feature(int64_list=tf.train.Int64List(value=list(values)))

    for (vec, label) in zip(list_vec, list_label):
        features = {'features': create_float_feature(vec), 'labels': create_int_feature([label])}
        tf_example = tf.train.Example(features=tf.train.Features(feature=features))
        writer.write(tf_example.SerializeToString())
```

Now we can load from it and build a `tf.Dataset`:
```python
def _decode_record(record):
    """Decodes a record to a TensorFlow example."""
    return tf.parse_single_example(record, {
        'features': tf.FixedLenFeature([768], tf.float32),
        'labels': tf.FixedLenFeature([], tf.int64),
    })

ds = (tf.data.TFRecordDataset('tmp.tfrecord').repeat().shuffle(buffer_size=100).apply(
    tf.contrib.data.map_and_batch(lambda record: _decode_record(record), batch_size=64))
      .make_one_shot_iterator().get_next())
```

The complete example can [be found example6.py](example6.py). 

To save word/token-level embedding to TFRecord, one needs to first flatten `[max_seq_len, num_hidden]` tensor into an 1D array as follows:
```python
def create_float_feature(values):
    return tf.train.Feature(float_list=tf.train.FloatList(value=values.reshape(-1)))
```
And later reconstruct the shape when loading it:
```python
name_to_features = {
    "feature": tf.FixedLenFeature([max_seq_length * num_hidden], tf.float32),
    "label_ids": tf.FixedLenFeature([], tf.int64),
}
    
def _decode_record(record, name_to_features):
    """Decodes a record to a TensorFlow example."""
    example = tf.parse_single_example(record, name_to_features)
    example['feature'] = tf.reshape(example['feature'], [max_seq_length, -1])
    return example
```
Be careful, this will generate a huge TFRecord file.

### Asynchronous encoding

`BertClient.encode()` offers a nice synchronous way to get sentence encodes. However,   sometimes we want to do it in an asynchronous manner by feeding all textual data to the server first, fetching the encoded results later. This can be easily done by:
```python
# an endless data stream, generating data in an extremely fast speed
def text_gen():
    while True:
        yield lst_str  # yield a batch of text lines

bc = BertClient()

# get encoded vectors
for j in bc.encode_async(text_gen(), max_num_batch=10):
    print('received %d x %d' % (j.shape[0], j.shape[1]))
```

The complete example can [be found example2.py](example2.py).

### Broadcasting to multiple clients

The encoded result is routed to the client according to its identity. If you have multiple clients with same identity, then they all receive the results! You can use this *multicast* feature to do some cool things, e.g. training multiple different models (some using `scikit-learn` some using `tensorflow`) in multiple separated processes while only call `BertServer` once. In the example below, `bc` and its two clones will all receive encoded vector.

```python
# clone a client by reusing the identity 
def client_clone(id, idx):
    bc = BertClient(identity=id)
    for j in bc.listen():
        print('clone-client-%d: received %d x %d' % (idx, j.shape[0], j.shape[1]))

bc = BertClient()
# start two cloned clients sharing the same identity as bc
for j in range(2):
    threading.Thread(target=client_clone, args=(bc.identity, j)).start()

for _ in range(3):
    bc.encode(lst_str)
```
The complete example can [be found in example3.py](example3.py).
