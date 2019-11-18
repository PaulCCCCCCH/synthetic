# Deconfounding Key-word Statistics

## Dependencies:
- `tensorflow==1.15`
- `nltk`
- `keras`

## Important
- May need pass larger `--attention_size` and `--lstm_size` for nli tasks
- Please make sure that all json files (both snli and mnli) are in one directory. Use version 1.0 for snli
and 0.9 for mnli. Embedding file must be specified for nli tasks.
Dataset file name settings can be found in `data_utils.py`:
```
SNLI_FILE_MAP = {
    "train": "snli_1.0_train.jsonl",
    "dev": "snli_1.0_dev.jsonl",
    "test": "snli_1.0_test.jsonl"
}

MNLI_FILE_MAP = {
    "train": "multinli_0.9_train.jsonl",
    "dev_matched": "multinli_0.9_dev_matched.jsonl",
    "dev_mismatched": "multinli_0.9_dev_mismatched.jsonl"
}
```

## Running models on synthetic dataset
run `runner_synthetic.sh` or run `generate.py` and then `main.py`.
See parameters using `python generate.py -h` and `python main.py -h`

## Running models on MNLI dataset:
run `runner_nli.sh`. Supported models:
```
all_models = {
    'reg_attention':    'models.RegAttention',
    'adv_mlp':          'models.LSTMPredModelWithMLPKeyWordModelAdvTrain',
    'hex_attention':    'models.LSTMPredModelWithRegAttentionKeyWordModelHEX',
    'baseline_lstm':    'models.LSTMPredModel',
    'baseline_mlp':     'models.MLPPredModel',
    'baseline_bilstm':  'models.BiLSTMPredModel',           # NLI task only
    'bilstm_attention': 'models.BiLSTMAttentionPredModel',  # NLI task only
    'baseline_esim':    'models.ESIMPredModel'              # NLI task only
}
```

Training logs and visualisations can be found in `./models/${modelname}/vis.html`

## Data Generation
Use `generate.py`. Parameters are defined as follows:
```
parser.add_argument("numtogen", help="Input number of sample sentences to generate", type=int, default=10)
parser.add_argument("grams", help="Specify the number n in n-gram", type=int, default=3)

parser.add_argument("--rebuild", action="store_true", default=False, help="Rebuild vocab and word effect")
parser.add_argument("--filter_n", type=int, help="Consider only the subset of vocabulary that appeared greater than or equal to n times", default=3)
parser.add_argument("--outdir", default="./data", help="Define output path")
```

## Training models
Use `main.py`. Parameters are defined as follows:
```
parser.add_argument("modeltype", help="the type of models to choose from, choose from " + str(model_types))
    parser.add_argument("modelname", help="specify the name of the model", type=str)

    parser.add_argument("--test", action="store_true", default=False, help="Only test and produce visualisation")
    parser.add_argument("--task", type=str, default="synthetic", help="Choose from ['synthetic', 'snli', 'mnli'] ")
    parser.add_argument("--debug", action="store_true", default=False, help="Use debug dataset for quick debug runs")
    parser.add_argument("--lam", type=float, default=0.01, help="Coefficient of regularization term")
    parser.add_argument("--reg_method", type=str, default="none",
                        help="Specify regularization method for key-model weights. Default is 'none'. Choose from " + str(
                            reg_methods))
    parser.add_argument("--epochs", type=int, default=21, help="Specify epochs to train")
    parser.add_argument("--kwm_path", type=str, default="",
                        help="Specify a path to the pre-trained keyword model. Will only train a key-word model if left empty.")
    parser.add_argument("--max_len", type=int, default=30, help="Maximum sentence length (excessive words are dropped)")
    parser.add_argument("--lstm_size", type=int, default=20, help="Size of lstm unit in current model.")
    parser.add_argument("--embedding_dim", type=int, default=20, help="Dimension of embedding to use.")
    parser.add_argument("--data_path", type=str, default="./data",
                        help="Specify data directory (where inputs, effect list, vocabulary, etc. are )")
    parser.add_argument("--batch_size", type=int, default=10)
    parser.add_argument("--keep_probs", type=float, default=0.8,
                        help="Keep probability of dropout layers. Set it to 1.0 to disable dropout.")
    parser.add_argument("--learning_rate", type=float, default=0.1)
    parser.add_argument("--embedding_file", type=str, default="",
                        help="Specify path to the pre-trained embedding file, if had one.")
```


## Key variables
- `word_dict`: mapping from word(str) to index(int)
- `effect_list`: mapping from word index(int) to word effect(float)
- `ngrams`: mapping from an ngram window(a tuple of strings) to a probability
- `samples`: a list of samples, each having the following fields:
    * `sentence`: the sentence as a list of strings
    * `sentence_ind`: the sentence as a list of word indices
    * `effect`: a list of float as effect for each word in the sentence
    * `bow_repr`: bag-of-word representation of the sentence (sum of each word as one-hot vectors)
    * `label`: 0 for negative, 1 for positive


## Vocabulary sizes:
```
>>> len([0 for x in list(counts) if x[1] >= 2])
2488
>>> len([0 for x in list(counts) if x[1] >= 3])
1891
>>> len([0 for x in list(counts) if x[1] >= 4])
1486
>>> len([0 for x in list(counts) if x[1] >= 5])
1227
>>> len([0 for x in list(counts) if x[1] >= 6])
1060
>>> len([0 for x in list(counts) if x[1] >= 7])
926
>>> len([0 for x in list(counts) if x[1] >= 8])
828
>>> len([0 for x in list(counts) if x[1] >= 9])
758
>>> len([0 for x in list(counts) if x[1] >= 10])
692
```

## TODO:



