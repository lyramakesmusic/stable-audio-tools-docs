# config docs for stable audio tools

### overview

stable audio tools lets you:
- train custom audio autoencoders (either [dac](https://github.com/descriptinc/descript-audio-codec), [encodec](https://github.com/facebookresearch/audiocraft/blob/main/docs/ENCODEC.md), or oobleck- their in-house autoencoder architecture)
- train generative audio diffusion models (via [diffusion transformers](https://arxiv.org/pdf/2212.09748.pdf))
- train [musicgen](https://github.com/facebookresearch/audiocraft/blob/main/docs/MUSICGEN.md) (less support)

basic documentation for the train command etc is [in their repo](https://github.com/Stability-AI/stable-audio-tools/blob/main/README.md). the configs are (currently) poorly documented outside of [a few examples](https://github.com/Stability-AI/stable-audio-tools/tree/main/stable_audio_tools/configs)

### baseline model config format

most config fields change per model type, but every config has the following fields:

```py
{
    "model_type": "autoencoder", 
    "sample_size": 65536,
    "sample_rate": 44100,
    "model": {},
    "training": {},  
}
```

## autoencoders

encoders compress audio from a `[channels, sample_length]` audio tensor to a smaller `[n_codebooks, latent_dim, length]` (quantized) or `[latent_dim, length]` (latent) tensor.

```py
{
    "model_type": "autoencoder",
    . . .
    "model": {
        "encoder": {},               # described below
        "decoder": {},               # described below
        "bottleneck: {},             # described below
        "latent_dim": 32,            # ... match the latent_dim from the model config, described below
        "downsampling_ratio": 2048,  # ...
        "io_channels": 1             # 1 = mono
    }
}
```

model definitions have `type` and `config`. `config` fields vary by type

```py
"encoder | decoder": {
    "type": "dac | oobleck",
    "config": {}
}
```

`dac` type encoders config like this:

```py
"config": {
    "in_channels": 2,             # 2 = stereo
    "latent_dim": 128,            # ...
    "d_model": 128,               # ...
    "strides": [2,4,8,8]          # ...
}
```

`oobleck` type encoders config like this:

```py
"config": {
    "in_channels": 1,             # 1 = mono
    "channels": 128,              # ...
    "c_mults": [1, 2, 4, 8, 16],  # ...
    "strides": [4, 4, 8, 8, 8],   # ...
    "latent_dim": 128,            # ...
    "use_snake": true             # ...
}
```

### bottlenecks

latents only (for oobleck-type encoders):

```py
"bottleneck": {
    "type": "vae"
}
```

quantized (for dac-type encoders):

```py
"bottleneck": {
    "type": "dac_rvq",
    "config": {
        "input_dim": 128,          # match the encoder/decoder latent dim
        "n_codebooks": 9,          # ...
        "codebook_dim": 8,         # ...
        "codebook_size": 2048,     # ...
        "quantizer_dropout": 0.5   # you don't really need to touch this
    }
},
```

## diffusion

```py
{
    "model_type": "diffusion_cond", # conditional diffusion, todo: others
    . . .
    "model": {
        "pretransform": { "autoencoder model config here" },
        "conditioning": {},
        "diffusion": {
            "type": "dit",
            "config": {},
            "cross_attention_cond_ids": [] # specific to dit?
        },
        "io_channels": 64
    }
}
```

## training

```py
"training": {
    "learning_rate": 1e-4,
    "loss_configs": {},
    "demo": {}
}
```
