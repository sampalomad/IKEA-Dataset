# CPNMT:
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Introduction:

CPNMT is a multilingual-multimodal dataset. It contains two language pairs: English-French and English-German. The text data is language-corresponded descriptions of all products crawled from IKEA and UNDERAMOUR. For each data sample in each language pairs, there is a corresponding product image that is compressed into a feature vector of size 2048. 

## Data Preprocessing:

Besides the raw, unprocessed version of all the data samples, there are two other versions of the data. the `IKEA/data.en.*/data.norm.tok.lc` folder contains normalized, tokenized, converted to lowercase (processed exclusively in such order) data. The `IKEA/data.en.*/data.norm.tok.lc.bpe` folder contains normalized, tokenized, converted to lowercase, byte-pair encoding (processed exclusively in such order) data. 

## Example:
![sample](./sample.png?raw=true "example")


## Statistics:
The below statistics is calculated with unprocessed data: 

| Language pair  | Language | Tokens | Minimum sample length | Maximum sample length | Average sample length | Standard derivation sample length | Vocabulary size |
|----------------|----------|--------|-----------------------|-----------------------|-----------------------|-----------------------------------|-----------------|
| English-German | English  | 256204 | 6                     | 343                   | 71.3660167            | 46.3361373                        | 6601            |
|                | German   | 216873 | 6                     | 324                   | 60.4103064            | 39.1434638                        | 10468           |
| English-French | English  | 239822 | 6                     | 334                   | 72.213791             | 47.2512623                        | 6442            |
|                | French   | 275223 | 6                     | 469                   | 82.8735321            | 54.7219488                        | 7575            |

## Characteristics:
- Because all data samples are the description of different products from IKEA or UNDERAMOUR, a data sample usually contain more than one sentences. 
-	A data sample is not a strict description of the corresponding product image. A description might contain information that cannot be showed in image. for example, a description for an Underamour product can contains the sentence “Don’t wash it with hot water”.
-	A data sample in German or French might be less complete with its corresponding English data sample because some part of the descriptions of certain products are not available in non-English regions.


## Data Format:

### Folder:
- `IKEA/`: data crawled and processed from IKEA and UNDERAMOUR.
- `IKEA/data.en.fr`: English-French data.
- `IKEA/data.en.de`: English-German data.
- `IKEA/data.en.*/data.raw`: unprocessed original data compressed in `.gz`. 
- `IKEA/data.en.*/data.norm.tok.lc`: normalized, tokenized and lowercase-converted data. 
- `IKEA/data.en.*/data.norm.tok.lc.bpe`: normalized, tokenized, lowercase-converted, byte-pair-encoded (10000) data.
- `IKEA/data.en.*/data.image.bpe`: image matrix for `train.*`, `test.*`, `val.*`.

### Data Files:
- `train.*`: 3K samples.
- `test.*`: 300 samples.
- `val.*`: 300 samples.
- `vocab.*`: language-corresponded vocabulary file extract from `*.norm.tok.lc.10000bpe.*`.
- `*_file.code`: language files for byte-pair encoding.
- `*.norm.tok.lc.10000bpe_ims.npy`: corresponded image matrix for `train.*`, `test.*`, `val.*`, each image is stored in a vector of size 2048. 


## Usages:
It can be used for text-only neural machine translation project and multimodal machine translation project.
To download the dataset, open the directory where you want to copy the data to on terminal, enter: 

```$ git clone https://github.com/sampalomad/CPNMT.git```

## Remark:
- Raw image data in `.jpg` can be released upon request.
