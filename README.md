# Multi-Hop Knowledge Graph Reasoning with Reward Shaping

This repository contains the source code release of the paper: [Lin et. al. 2018. Multi-Hop Knowledge Graph Reasoning with Reward Shaping](https://arxiv.org/abs/1808.10568).

## Quick Start

### Environment Variables & Dependencies
#### Use Docker
Build the docker image
```
docker build -< Dockerfile -t multi_hop_kg:v1.0
```
*The rest of the readme assumes that one spins up a container from the image built and works inside it interactively. If you prefer to run experiments outside a container, please change the commands accordingly.*

Spin up a container and run experiments inside it.
```
nvidia-docker run -v `pwd`:/workspace/MultiHopKG -it multi_hop_kg:v1.0
```

#### Mannually Set up 
Alternatively, you can install Pytorch (>=0.4.1) manually and use the Makefile to set up the rest of the dependencies. 
```
make setup
```

### Process data
First, unpack the data files 
```
tar xvzf data-release.tgz
```
and run the following command to preprocess the datasets.
```
./experiment.sh configs/<dataset>.sh --process_data <gpu-ID>
```

`<dataset>` is the name of any dataset folder in the `./data` directory. In our experiments, the five datasets used are: `umls`, `kinship`, `fb15k-237`, `wn18rr` and `nell-995`. 
`<gpu-ID>` is a non-negative integer number representing the GPU index.

### Train models
Then the following commands can be used to train the proposed models and baselines in the paper. By default, dev set evaluation results will be printed when training terminates.

1. Train embedding-based models
```
./experiment-emb.sh configs/<dataset>-<emb_model>.sh --train <gpu-ID>
```
The following embedding-based models are implemented: `distmult`, `complex` and `conve`.

2. Train RL models (policy gradient)
```
./experiment.sh configs/<dataset>.sh --train <gpu-ID>
```

3. Train RL models (policy gradient + reward shaping)
```
./experiment-rs.sh configs/<dataset>-rs.sh --train <gpu-ID>
```

* Note: To train the RL models using reward shaping, make sure 1) you have pre-trained the embedding based models and 2) set the file path pointers to the pre-trained embedding based models correctly ([example](configs/umls-rs.sh)).

### Evaluate pretrained models
To generate the evaluation results of a pre-trained model, simply change the `--train` flag in the commands above to `--inference`. 

For example, the following command performs inference with the RL models (policy gradient + reward shaping) and prints the evaluation results (on both dev and test sets).
```
./experiment-rs.sh configs/<dataset>-rs.sh --inference <gpu-ID>
```

* Note for the NELL-995 dataset: 

  On this dataset we split the original training data into `train.triples` and `dev.triples`, and the final model to test has to be trained with these two files combined. 
  1. To obtain the correct test set results, you need to add the `--test` flag to all data pre-processing, training and inference commands.  
    ```
    ./experiment.sh configs/nell-995.sh --process_data <gpu-ID> --test
    ./experiment-emb.sh configs/nell-995-conve.sh --train <gpu-ID> --test
    ./experiment-rs.sh configs/NELL-995-rs.sh --train <gpu-ID> --test
    ./experiment-rs.sh configs/NELL-995-rs.sh --inference <gpu-ID> --test
    ```
  2. Leave out the `--test` flag during development.

### Change the hyperparameters
To change the hyperparameters and other experiment set up, start from the [configuration files](configs).

## Citation
If you find the resource in this repository helpful, please cite
```
@inproceedings{LinRX2018:MultiHopKG, 
  author = {Xi Victoria Lin and Richard Socher and Caiming Xiong}, 
  title = {Multi-Hop Knowledge Graph Reasoning with Reward Shaping}, 
  booktitle = {Proceedings of the 2018 Conference on Empirical Methods in Natural
               Language Processing, {EMNLP} 2018, Brussels, Belgium, October
               31-November 4, 2018},
  year = {2018} 
}
```
