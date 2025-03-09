# Muon: An optimizer for the hidden layers of neural networks

This repo contains an implementation of the `Muon` optimizer originally described in [this thread](https://x.com/kellerjordan0/status/1842300916864844014) and [this writeup](https://kellerjordan.github.io/posts/muon/).

## Installation

```
pip install git+https://github.com/KellerJordan/Muon
```

## Usage

Muon is intended to optimize only the internal ≥2D parameters of a network.
Embeddings, classifier heads, and scalar or vector parameters should be optimized using AdamW.

```python
# optimizer = torch.optim.AdamW(model.parameters(), lr=3e-4, betas=(0.90, 0.95), weight_decay=0.01)

from muon import Muon
# Find ≥2D parameters in the body of the network -- these should be optimized by Muon
muon_params = [p for p in model.body.parameters() if p.ndim >= 2]
# Find everything else -- these should be optimized by AdamW
adamw_params = ([p for p in model.body.parameters() if p.ndim < 2]
              + [*model.head.parameters(), *model.embed.parameters()])
# Create the optimizer
optimizers = [Muon(muon_params, lr=0.02, momentum=0.95),
              torch.optim.AdamW(adamw_params, lr=3e-4, betas=(0.90, 0.95), weight_decay=0.01)]
...

# in the training step
for opt in optimizers:
    opt.step()
```

You'll have to replace `model.body`, `model.head`, and `model.embed` with whatever subset is appropriate for your model.
E.g., for a ConvNet, `muon_params` should be all the convolutional filters, and `adamw_params` should be everything else.

## Example usage

[Example use in the NanoGPT speedrun](https://github.com/KellerJordan/modded-nanogpt/blob/d700b8724cbda3e7b1e5bcadbc0957f6ad1738fd/train_gpt.py#L519)

[Example use in the CIFAR-10 speedrun](https://github.com/KellerJordan/cifar10-airbench/blob/28bff5f5b31e95aa45b5b20e1f48baf1ed98d5f6/airbench94_muon.py#L362)

## Hyperparameter tuning

Typically, the default values of momentum (0.95), nesterov (True), and ns_steps (5) work well. The only hyperparameter which must be tuned is the learning rate.
It should have constant muP scaling, that is, as you scale up the model size, you shouldn't need to retune the learning rate.

## Benchmarks

For a comparison between AdamW, Shampoo, SOAP, and Muon for training a 124M-parameter transformer, see [here](https://github.com/KellerJordan/modded-nanogpt/tree/master/records/102924_Optimizers).

## Accomplishments

* [Lowered the record for training to 94% on CIFAR-10 from 3.3 A100-seconds to 2.6 A100-seconds](https://github.com/KellerJordan/cifar10-airbench)
* [Used to train a transformer to GPT-2 (XL) performance in $175 of compute](https://x.com/kellerjordan0/status/1850995958697308307)
* [Improved the training speed record for attaining GPT-2 (small) performance by a factor of 1.35x](https://x.com/kellerjordan0/status/1842300916864844014)
* [Adopted by the Kimi.ai lab for scaled LLM training](https://x.com/Kimi_Moonshot/status/1893379158472044623)

## More learning resources about Muon

* [Blog post on Muon by Jialin Su (the creator of RoPE)](https://kexue.fm/archives/10592)
* [Blog post by Jeremy Bernstein on theoretical background of Muon](https://jeremybernste.in/writing/deriving-muon)
* [Tech report by Kimi.ai on using Muon for scaled training](https://arxiv.org/abs/2502.16982v1)

## Citation

```bibtex
@misc{jordan2024muon,
  author       = {Keller Jordan and Yuchen Jin and Vlado Boza and You Jiacheng and
                  Franz Cesista and Laker Newhouse and Jeremy Bernstein},
  title        = {Muon: An optimizer for hidden layers in neural networks},
  year         = {2024},
  url          = {https://kellerjordan.github.io/posts/muon/}
}
```
