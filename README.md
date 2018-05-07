## Introduction:

CPNMT is a multilingual-multimodal dataset. It contains two language pairs: English-French and English-German. The text data is language-corresponded descriptions of all products crawled from Ikea.com and underamour.com. For each data sample in each language pairs, there is a corresponding product image that is compressed into a feature vector of size 2048. 

## Data Preprocessing:

Besides the raw, unprocessed version of all the data samples, there are two other versions of the data. the `IKEA/data.norm.tok.lc` folder contains normalized, tokenized, converted to lowercase (processed exclusively in such order) data. The `IKEA/data.norm.tok.lc.bpe` folder contains normalized, tokenized, converted to lowercase, byte-pair encoding (processed exclusively in such order) data. 

## Example:
The below statistics is calculated with unprocessed data: 

![sample](./sample.png?raw=true "example")


## Statistics:

| Language pair  | Language | Tokens | Number of data Samples | Minimum sample length | Maximum sample length | Average sample length | Standard derivation sample length | Vocabulary size |
|----------------|----------|--------|------------------------|-----------------------|-----------------------|-----------------------|-----------------------------------|-----------------|
| English-German | English  | 256204 | 3590                   | 6                     | 343                   | 71.3660167            | 46.3361373                        | 6601            |
|                | German   | 216873 | 3590                   | 6                     | 324                   | 60.4103064            | 39.1434638                        | 10468           |
| English-French | English  | 239822 | 3321                   | 6                     | 334                   | 72.213791             | 47.2512623                        | 6442            |
|                | French   | 275223 | 3321                   | 6                     | 469                   | 82.8735321            | 54.7219488                        | 7575            |


## Characteristics:
⋅⋅* Because all data samples are the description of different products from ikea.com or underamour.com, a data sample might contain more than one sentences. 
⋅⋅*	A data sample is not a strict description of the corresponding product image. A description might contain information that cannot be showed in image. for example, a description for an Underamour product can contains the sentence “Don’t wash it with hot water”.
⋅⋅*	A data sample in German/French might be less complete with its corresponding English data sample because some part of the descriptions of certain products are not available in non-English regions.

- `train.*` : 29K sentences
- `val.*` : 1014 sentences
- `test2016.*` : 1000 sentences
- `test2017.*` : 1000 sentences
- `testcoco.*` : 461 sentences

The `*_images.txt` files are text files containing the list of image names
for each split as they are ordered in the image features files.

## Preparation

The script `scripts/preprocess-bpe-pkl.sh` will first use the following scripts
(which should be **available** in your `$PATH`) from
[Moses](https://github.com/moses-smt/mosesdecoder) repository in order to preprocess the corpora:

 - Normalize punctuations (`normalize-punctuation.perl`)
 - Tokenize (`tokenizer.perl`)
 - Lowercase (`lowercase.perl`)

It will then learn a joint BPE model with 10K merge operations
(for `en->de` and `en->fr` separately)
using the tools provided by the [subword-nmt](https://github.com/rsennrich/subword-nmt) repository. You need
to adapt the script to point to the correct **subword-nmt** folder
by modifying the variables `BPE_APPLY` and `BPE_LEARN`.

Once the BPE-ized files are saved under `data.tok.bpe/bpe.en-de` and `data.tok.bpe/bpe.en-fr`, `nmt-build-dict` from `nmtpy` project will be used to create the vocabulary `.pkl` files in the respective folders.

Since the multimodal architectures have their own data iterators, they need a special `.pkl` corpora file for each Flickr30k and MSCOCO split. These files are created by `scripts/raw2pkl` which is automatically called from `scripts/preprocess-bpe-pkl.sh`.

In the end, the files in `data.tok.bpe/bpe.en-{de,fr}` will be the files
that are used by `nmtpy`. The non-BPE versions of validation and test sets
from `data.tok.bpe` will also be used when scoring the hypotheses with
automatic metrics.

**Note**: The script should be launched directly from the `wmt17-mmt` checkout
folder.

### Image Features

Once the above preprocessing step is completed, you will need to download
and extract the image features under `data/images` as described in the
relevant [README](data/images/) file.

## Training

You should now be ready to train monomodal and multimodal architectures
using the prepared data. If everything went well and you have a recent
enough installation of `nmtpy`, you can use the following commands to
start training your baselines:

```
# Monomodal En->De system
$ nmt-train -c config/monomodal-en-de.conf

# Monomodal En->Fr system
$ nmt-train -c config/monomodal-en-fr.conf

# MNMT (trgmul variant) En->De system
$ nmt-train -c config/mnmt-en-de.conf

# MNMT (trgmul variant) En->Fr system
$ nmt-train -c config/mnmt-en-fr.conf

# MNMT (fusion with conv features) En->De system
$ nmt-train -c config/fusion-en-de.conf

# MNMT (fusion with conv features) En->Fr system
$ nmt-train -c config/fusion-en-fr.conf
```

These configurations will save the best `.npz` checkpoints
under `models/` inside your `wmt17-mmt` checkout.

## Decoding & Scoring

```
# Decode test2017 for monomodal en->de
$ nmt-translate -m models/monomodal-en-de/attention-e128-r256-...-s1234.1.BEST.npz \
                -S data.tok.bpe/bpe.en-de/test2017.norm.tok.lc.bpe10000.en \
                -o nmt.test2017.tok.de

# Decode test2017 for mnmt en->de
$ nmt-translate -m models/mnmt-en-de/mnmt_trgmul-e128-i2048-r256-...-s1234.1.BEST.npz \
                -S data.tok.bpe/bpe.en-de/test2017.bpe10000.pkl \
                   data/images/resnet50-imagenet-pool5/flickr30k_ResNet50_pool5_test2017.npy \
                -o mnmt.test2017.tok.de

# Score both systems (Output stripped to fit here)
$ nmt-coco-metrics -l de -r data.tok.bpe/test2017.norm.tok.lc.de -s *test2017.tok.de

|    Bleu_1     ||    Bleu_2     ||    Bleu_3     ||    Bleu_4     ||    METEOR   |
nmt.test2017.tok.de
|    63.321     ||    48.730     ||    38.804     ||    31.255     ||    51.274   |
mnmt.test2017.tok.de
|    64.342     ||    49.977     ||    39.879     ||    32.112     ||    51.529   |
```

## Results

(Note that the results below belong to single runs while the ones reported
in the paper are averages and ensembles of 5 runs.)

| System          | Val METEOR/BLEU | Test2016 METEOR/BLEU | Test2017 METEOR/BLEU |
|-----------------|-----------------|----------------------|----------------------|
| monomodal-en-de | 56.83/39.17     | 57.40/39.00          | 51.27/31.25          |
| mnmt-en-de      | 56.99/39.15     | 57.05/38.97          | 51.52/32.11          |
| fusion-en-de    |                 |                      |                      |
| monomodal-en-fr | 72.87/57.79     | 74.19/59.02          | 68.87/51.93          |
| mnmt-en-fr      | 73.88/58.93     | 74.75/59.82          | 69.48/52.61          |
| fusion-en-fr    |                 |                      |                      |
