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

"model" is the model definition. for autoencoders, it defines the encoder, decoder, and bottleneck. for diffusion, it defines the pretransform (aka compression model aka paste the autoencoder model definition), the conditioning, and the actual diffusion transformer shape.

"training" defines (some of- the rest are passed to `train.py`) the training hyperparams (unless LR is set in the model definition, like dac does), losses (including any adversarial stuff), and demo generation params.

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
        "conditioning": {
            "configs": [],
            "cond_dim": 512  # match the diffusion models `cond_token_dim` (??)
        },
        "diffusion": {
            "type": "dit",
            "config": {},
            "cross_attention_cond_ids": [] # specific to dit?
        },
        "io_channels": 64 # latent dimension?? 
    }
}
```

DiT model definition params:

```py
"config": {
    "io_channels": 64,     # encoders latent_dim (?)
    "embed_dim": 1536,     # ...
    "depth": 24,           # ...
    "num_heads": 24,       # pretty self-explanatory. usually in multiples of 8
    "cond_token_dim": 768  # size of the input tensor, defined in "conditioning"
}
```

```py
"cross_attention_cond_ids": [
    "...", # list each config ID defined in "configs"
],
```

## training

```py
"training": {
    "learning_rate": 1e-4,
    "loss_configs": {},
    "demo": {}
}
```

demo:

```py
"demo": {
    "demo_every": 2000,

    # FOR CONDITIONAL DIFFUSION MODELS:
	"demo_steps": 250,  # 250 is overkill, 50 or 100 is alright
    "num_demos": 4,     # run 4 test prompts
    "demo_cond": [
        {
            "prompt": "test prompt",
            "(conditioning ID)", "(value)", # ...
        },
        ... (x3 more)
    ],
    "demo_cfg_scales": [3, 6, 9] # run each demo at each of these 3 scales. it will use the same seed. generally wont have to touch this.
}
```
