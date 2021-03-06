# UNION

![](https://img.shields.io/github/last-commit/thu-coai/UNION?color=blue) 

Automatic Evaluation Metric described in the paper [UNION: An **UN**referenced Metr**I**c for Evaluating **O**pen-e**N**ded Story Generation](https://arxiv.org/abs/2009.07602) (EMNLP 2020). Please refer to the [Paper List](https://github.com/thu-coai/PaperForONLG) for more information about **O**pen-e**N**ded **L**anguage **G**eneration (ONLG) tasks. Hopefully the paper list will help you know more about this field.



### Contents

* [Prerequisites](#prerequisites)
* [Computing Infrastructure](#computing-infrastructure)
* [Quick Start](#quick-start)
  * [1 Constructing Negative Samples](#1-constructing-negative-samples)
  * [2 Training of UNION](#2-training-of-union) 
  * [3 Prediction of UNION](#3-prediction-with-union)
  * [4 Correlation Calculation](#4-correlation-calculation)
* [Data Instruction](#data-instruction-for-files-under-data)
* [Citation](#citation)



## Prerequisites

The code is written in TensorFlow library. To use the program the following prerequisites need to be installed.

- Python 3.7.0
- tensorflow-gpu 1.14.0
- numpy 1.18.1
- regex 2020.2.20
- nltk 3.4.5



## Computing Infrastructure

We train UNION based on the platform: 

- OS: Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-98-generic x86_64)
- GPU: NVIDIA TITAN Xp



## Quick Start

#### 1. Constructing Negative Samples

Execute the following command: 

```shell
cd ./Data
python3 ./get_vocab.py your_mode
python3 ./gen_train_data.py your_mode
```

- `your_mode` is `roc` for `ROCStories corpus` or  `wp` for `WritingPrompts dataset`. Then the summary of vocabulary and the corresponding frequency and pos-tagging will be found under `ROCStories/ini_data/entitiy_vocab.txt` or `WritingPrompts/ini_data/entity_vocab.txt`. 
- Negative samples and human-written stories will be constructed based on the original training set. The training set will be found under `ROCStories/train_data` or `WritingPrompts/train_data`.
- Note: currently only 10 samples of the full original data and training data are provided. The full data can be downloaded from [THUcloud](https://cloud.tsinghua.edu.cn/d/b3bdeee2c9b647439746/) or [GoogleDrive](https://drive.google.com/drive/folders/1Cfc-YkQo-27ovVug2bfpqBclECimvgwu?usp=sharing).




#### 2. Training of UNION

Execute the following command: 

```shell
python3 ./run_union.py --data_dir your_data_dir \
    --output_dir ./model/union \
    --task_name train \
    --init_checkpoint ./model/uncased_L-12_H-768_A-12/bert_model.ckpt
```

- `your_data_dir` is `./Data/ROCStories` or `./Data/WritingPrompts`.
- The initial checkpoint of BERT can be downloaded from [bert](https://github.com/google-research/bert). We use the uncased base version of BERT (about 110M parameters). We train the model for 40000 steps at most. The training process will task about 1~2 days. 



#### 3. Prediction with UNION

Execute the following command: 

```shell
python3 ./run_union.py --data_dir your_data_dir \
    --output_dir ./model/output \
    --task_name pred \
    --init_checkpoint your_model_name
```

- `your_data_dir` is `./Data/ROCStories` or `./Data/WritingPrompts`. If you want to evaluate your custom texts, you only need tp change your file format into ours. 

-  `your_model_name` is `./model/union_roc/union_roc` or `./model/union_wp/union_wp`. The fine-tuned checkpoint can be downloaded from the following link:

  | Dataset        | Fine-tuned Model                                             |
  | -------------- | ------------------------------------------------------------ |
  | ROCStories     | [THUcloud](https://cloud.tsinghua.edu.cn/d/23722540a7f944019427/); [GoogleDrive](https://drive.google.com/drive/folders/1T1SYM9OxPEsjpk2eGWot0iuyJbfIGmBM?usp=sharing) |
  | WritingPrompts | [THUcloud](https://cloud.tsinghua.edu.cn/d/0154034b7a574d0498c9/); [GoogleDrive](https://drive.google.com/drive/folders/1Z6uYG4WQBR3jzZAykQGfAEHriWc8CA0l?usp=sharing) |

- The union score of the stories under `your_data_dir/ant_data` can be found under the output_dir `./model/output`.



#### 4. Correlation Calculation

Execute the following command: 

```shell
python3 ./correlation.py your_mode
```

Then the correlation between the human judgements under  `your_data_dir/ant_data` and the scores of metrics under `your_data_dir/metric_output` will be output. The figures under "./figure" show the score graph between metric scores and human judgments for `ROCStories corpus`.



## Data Instruction for files under `./Data`

```markdown
├── Data
   └── `negation.txt`             # manually constructed negation word vocabulary.
   └── `conceptnet_antonym.txt`   # triples with antonym relations extracted from ConceptNet.
   └── `conceptnet_entity.csv`    # entities acquired from ConceptNet.
   └── `ROCStories`
       ├── `ant_data`        # sampled stories and corresponding human annotation.
              └── `ant_data.txt`        # include only binary annotation for reasonable(1) or unreasonable(0)
              └── `ant_data_all.txt`    # include the annotation for specific error types: reasonable(0), repeated plots(1), bad coherence(2), conflicting logic(3), chaotic scenes(4), and others(5). 
              └── `reference.txt`       # human-written stories with the same leading context with annotated stories.
              └── `reference_ipt.txt`
              └── `reference_opt.txt`
       ├── `ini_data`        # original dataset for training/validation/testing.
              └── `train.txt`
              └── `dev.txt`
              └── `test.txt`
              └── `entity_vocab.txt`    # generated by `get_vocab.py`, consisting of all the entities and the corresponding tagged POS followed by the mention frequency in the dataset.
       ├── `train_data`      # negative samples and corresponding human-written stories for training, which are constructed by `gen_train_data.py`.
              └── `train_human.txt`
              └── `train_negative.txt`
              └── `dev_human.txt`
              └── `dev_negative.txt`
              └── `test_human.txt`
              └── `test_negative.txt`
       ├── `metric_output`   # the scores of different metrics, which can be used to replicate the correlation in Table 5 of the paper. 
              └── `bleu.txt`
              └── `bleurt.txt`
              └── `ppl.txt`             # the sign of the result of Perplexity needs to be changed to get the result for *minus* Perplexity.
              └── `union.txt`
              └── `union_recon.txt`     # the ablated model without the reconstruction task
              └── ...
   └── `WritingPrompts`
       ├── ...
 
```

- The annotated data file `ant_data.txt` and `ant_data_all.txt` are formatted as `Story ID ||| Story ||| Seven Annotated Scores`.
- `ant_data_all.txt` is only available for `ROCStories corpus`. `ant_data_all.txt` is the same with `ant_data.txt` for `WrintingPrompts dataset`. 



### Citation

Please kindly cite our paper if [this paper](https://arxiv.org/abs/2009.07602) and the [code](https://github.com/thu-coai/UNION) are helpful.

```
@misc{guan2020union,
    title={UNION: An Unreferenced Metric for Evaluating Open-ended Story Generation},
    author={Jian Guan and Minlie Huang},
    year={2020},
    eprint={2009.07602},
    archivePrefix={arXiv},
    primaryClass={cs.CL}
}
```