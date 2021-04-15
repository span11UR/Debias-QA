# LUKE-QA-bias-analysis


To produce the result as in `output/result.json`, you should follow the instructions step by step here:

1. Download all required data from link [google drive](https://drive.google.com/drive/folders/1peLPm0rGUmKuE2MeYWVN-3SDDBGUjxbL?usp=sharing) using the CMU account and store them under the directory `data`.

2. Download the LUKE pretrained model and entity data from [google drive](https://drive.google.com/drive/folders/1Gu9BI9w6twOT70Ha2uULuhobaBO21nKy?usp=sharing) using the CMU account and store them under the main directory `LUKE-QA-bias-analysis`.

3. Download the model checkpoints at [google drive](https://drive.google.com/drive/folders/1KTxIjnaLpD5m_23QCsaWxiUSAwxoDZ_U?usp=sharing) using the CMU account and store them under the main directory `LUKE-QA-bias-analysis`.

    ** The file `python_model_reproduce.bin` should also be downloaded if you only want to train the model by yourself instead of loading the weight (Since the code will check the existence of the files even not using them)

4. Install all required packages using `pip3 install -r requirements.txt`, and PyTorch version should be 1.2.0 and CUDA version should be 10

## Fintunning Model
Run the following command for finetunning on SQuAD 1.1 dataset (we use the same command as the original paper for fine tunning):
```
    python3 -m examples.cli \
    --num-gpus=1 \
    --model-file=luke_large_500k.tar.gz \
    --output-dir=output \
    reading-comprehension run \
    --data-dir=data \
    --no-negative \
    --wiki-link-db-file=enwiki_20160305.pkl \
    --model-redirects-file=enwiki_20181220_redirects.pkl \
    --link-redirects-file=enwiki_20160305_redirects.pkl \
    --train-batch-size=2 \
    --gradient-accumulation-steps=3 \
    --learning-rate=15e-6 \
    --num-train-epochs=2
```
    
The model's weight will be generated under the `output` directory named `python_model.bin`, you could change the output directory by modifying the `--output-dir` command. 

For training, we used the AWS p3.2xlarge instance (contains a single V100 GPU) and it takes about 8 hours to train with the exact provided setting.

## Evaluate and Reproduce The Paper's Result
To reproducing the paper results using the weight we have trained, run the following command:
```
    python3 -m examples.cli \
    --model-file=luke_large_500k.tar.gz \
    --output-dir=output reading-comprehension run \
    --checkpoint-file=pytorch_model_reproduce.bin \
    --no-train --no-negative
```
The beam search results, output predictions, and scores will be stored in the `output` directory with the name of `nbest_predictions_.json`, `predictions_.json`, `results.json`

If you want to use the weights trained by the paper itself, then you could download the file [here](https://drive.google.com/file/d/1097QicHAVnroVVw54niPXoY-iylGNi0K/view?usp=sharing)(compressed) and replace the `--checkpoint-file` with the new file path.

## Generate output
Run the following command to generate the prediction and beam search result for four groups of undespecific questions:
```
    python3 -m examples.cli \
    --model-file=luke_large_500k.tar.gz \
    --output-dir=output reading-comprehension run \
    --checkpoint-file=pytorch_model_reproduce.bin \
    --no-train --no-negative --do-unqover --unqover-file=R
```
To change the different groups of undespecific questions you want to predict, using the `--unqover-file` command.

`--unqover-file=R`: use religions dataset

`--unqover-file=E`: use ethnicity dataset

`--unqover-file=G`: use gender dataset

`--unqover-file=N`: use nationality dataset

The final output will be stored in the `output` directory with the name: `nbest_predictions_$groupname_.json` and `predictions_$groupname_.json`

** It takes about 10-20 minutes for the model to generate the output file on the AWS g4dn 2xlarge machine. 

## Get the Data Directly for Bias Analysis
You can also download the file we have generated by our model from [google drive](https://drive.google.com/drive/folders/1vyMeDl5TURGPG9UFUG67GIsi0-EhBe61?usp=sharing).


## reproduced results
By finetuning the model on SQuAD results, we are able to generate the following F1 and exact match scores: 

   LUKE (reported) Exact Match 89.8 F1 95.0
   LUKE (reproduced) Exact Match 89.7 F1 94.9
   
The reproduced result is of minor differences compared to the reported exact match and F1 scores reported in the paper. 