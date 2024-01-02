# config docs for stable audio tools

**base format**

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

```py
{
    "model_type": "autoencoder",
    . . .
    "model": {
        "encoder": {},
        "decoder": {},
        "bottleneck: {},
        "latent_dim": 32,
        "downsampling_ratio": 2048,
        "io_channels": 1
    }
}
```

model definitions have `type` and `config`. `config` fields vary by type

```py
"encoder": {
    "type": "dac",
    "config": {
        "in_channels": 2,
        "latent_dim": 128,
        "d_model": 128,
        "strides": [2,4,8,8]
    }
}
```

```py
"bottleneck": {
    "type": "dac_rvq",
    "config": {
        "input_dim": 128,
        "n_codebooks": 9,
        "codebook_dim": 8,
        "codebook_size": 2048,
        "quantizer_dropout": 0.5
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
