# Dual-Encoders for IR

This repository implements dual-encoder models for information retrieval. Specifically, the following is supported:

- Training of Transformer-based symmetric or asymmetric dual-encoders
- Indexing corpora from ir-datasets
- Re-ranking using [Fast-Forward indexes](https://github.com/mrjleo/fast-forward-indexes)
- Dense retrieval using [FAISS](https://github.com/facebookresearch/faiss)

## Requirements

Install the packages from `requirements.txt`. Note that [ranking_utils](https://github.com/mrjleo/ranking-utils) has additional dependencies.

## Usage

We use [Hydra](https://hydra.cc/) for the configuration of all scripts, such as models, hyperparameters, paths and so on. Please refer to the documentation for instructions how to use Hydra.

Currently, the following encoder models are available along with the corresponding configuration files:

- Standard Transformer (`config/ranker/encoder/transformer.yaml`)
- Embedding-based Transformer (`config/ranker/encoder/transformer_embedding.yaml`)
- Selective Transformer (`config/ranker/encoder/selective_transformer.yaml`)

More information about these models can be found in [our paper](https://dl.acm.org/doi/10.1145/3631939).

### Pre-Processing

Currently, datasets must be pre-processed in order to use them for training. Refer to [this guide](https://github.com/mrjleo/ranking-utils#dataset-pre-processing) for a list of supported datasets and pre-processing instructions.

### Training

We use [PyTorch Lightning](https://lightning.ai/docs/pytorch/stable/) for training. Use the training script to train a new model and save checkpoints. At least the following options must be set: `training_data.data_dir` and `training_data.fold_name`.

For example, in order to train a symmetric dual-encoder, run the following:

```
python train.py \
    ranker/encoder@ranker.query_encoder=transformer \
    ranker.doc_encoder.pretrained_model=bert-base-uncased \
    ranker/encoder@ranker.doc_encoder=transformer \
    ranker.doc_encoder.pretrained_model=bert-base-uncased \
    training_data.data_dir=/path/to/preprocessed/files \
    training_data.fold_name=fold_0
```

The default configuration for training can be found in `config/trainer/fine_tune.yaml`. All defaults can be overriden via the command line.

You can further override or add new arguments to other components such as the [`pytorch_lightning.Trainer`](https://lightning.ai/docs/pytorch/stable/common/trainer.html#trainer-flags). Some examples are:

- `+trainer.val_check_interval=5000` to validate every 5000 batches.
- `+trainer.limit_val_batches=1000` to use only 1000 batches of the validation data.

**Important**: Training using the DDP strategy (`trainer.strategy=ddp`) may throw an error due to unused parameters. This can be worked around using `trainer.strategy=ddp_find_unused_parameters_true`.

#### Model Parameters

Model (hyper)parameters can be overidden like any other argument. They are prefixed by `ranker.model` and `ranker.model.hparams`, respectively. Check `config/ranker/dual_encoder.yaml` for available options.

#### Validation

Validation using ranking metrics is enabled by default (see [ranking-utils](https://github.com/mrjleo/ranking-utils?tab=readme-ov-file#validation) for more information). This can be configured using the Trainer options (see `config/trainer/fine_tune.yaml`).

#### Output Files

The default behavior of Hydra is to create a new directory, `outputs`, in the current working directory. In order to use a custom output directory, override the `hydra.run.dir` argument.
