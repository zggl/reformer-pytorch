# Reformer, the Efficient Transformer, in Pytorch

<img src="./lsh_attention.png" width="500">

This is a Pytorch implementation of Reformer https://openreview.net/pdf?id=rkgNKkHtvB

It includes LSH attention, reversible network, and chunking. It has been validated with a toy auto-regressive task.

## Install

```bash
pip install reformer_pytorch
```

## Usage

The full Reformer

```python
# should fit in ~ 5gb - 8k tokens

import torch
from reformer_pytorch import Reformer

model = Reformer(
    emb = 512,
    depth = 12,
    max_seq_len = 8192,
    num_tokens= 20000,
    heads = 8,
    lsh_dropout = 0.1,
    causal = True,        # auto-regressive or not
    bucket_size = 64,     # average size of qk per bucket, 64 was recommended in paper
    n_hashes = 8,         # should keep at 8 per paper
    ff_chunks = 200,      # number of chunks for feedforward layer
    weight_tie = False,   # tie parameters of each layer for no memory per additional depth
    attn_chunks = 8,        # process lsh attention in chunks, only way for memory to fit when scaling to 16k tokens
    use_full_attn = False   # use full self attention, for comparison
).cuda()

x = torch.randint(0, 20000, (1, 8192)).long().cuda()
y = model(x)
```

Self Attention with LSH

```python
import torch
from reformer_pytorch import LSHSelfAttention

attn = LSHSelfAttention(
    emb = 128,
    heads = 8,
    bucket_size = 64,
    n_hashes = 8,
    causal = False
)

x = torch.randn(10, 1024, 128)
y = attn(x)
```

LSH (locality sensitive hashing) Attention

```python
import torch
from reformer_pytorch import LSHAttention

attn = LSHAttention(
    bucket_size = 64,
    n_hashes = 16,
    causal = True
)

qk = torch.randn(10, 1024, 128)
v = torch.randn(10, 1024, 128)

attn_out, buckets = attn(qk, v)

# buckets will contain the bucket number (post-argmax) of each token of each batch
```

## Todo

1. Make it so Reformer can be used as decoder where queries only attend to fed key/values
2. Recurrence like Transformer XL
3. All-attention learned memory key values
