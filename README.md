# CorPipe 25: CRAC 2025 Winning System for Multilingual Coreference Resolution

This repository contains the source code and the pre-trained models of CorPipe 25,
the CRAC 2025 winning system for multilingual coreference resolution, which
is described in the following paper:

---

<img src="figures/crac25_empty_nodes_baseline.svg" alt="CRAC 25 Empty Nodes Baseline" align="right" style="width: 15.8%">
<img src="figures/corpipe25_architecture.svg" alt="CorPipe 25 Architecture" align="right" style="width: 34.2%">

<h3 align="center"><a href="https://arxiv.org/abs/2509.17858">CorPipe at CRAC 2025: Evaluating Multilingual Encoders for Multilingual Coreference Resolution</a></h3>

<p align="center">
  <b>Milan Straka</b><br>
  Charles University<br>
  Faculty of Mathematics and Physics<br>
  Institute of Formal and Applied Lingustics<br>
  Malostranské nám. 25, Prague, Czech Republic
</p>

**Abstract:** We present CorPipe 25, the winning entry to the CRAC 2025 Shared
Task on Multilingual Coreference Resolution. This fourth iteration of the shared
task introduces a new LLM track alongside the original unconstrained track,
features reduced development and test sets to lower computational requirements,
and includes additional datasets. CorPipe 25 represents a complete
reimplementation of our previous systems, migrating from TensorFlow to PyTorch.
Our system significantly outperforms all other submissions in both the LLM and
unconstrained tracks by a substantial margin of 8 percentage points. <br clear="both">

---

## Content of this Repository

- The directory `data` is for the CRAC 2025 data, and the preprocessed
  and tokenized version needed for training.
  - The script `data/get.sh` downloads and extracts the CRAC 2025 training and
    development data, plus the unannotated test data of the CRAC 2025 shared
    task.

- The `corpipe25.py` is the complete CorPipe 25 source file.

- The `corefud-score.sh` is an evaluation script used by `corpipe25.py`, which
  - performs evaluation (using the official evaluation script from the `corefud-scorer` submodule),
  - optionally (when `-v` is passed), it also:
    - runs validation (using the official UD validator from the `validator` submodule) on the output data,
    - performs evaluation with singletons,
    - performs evaluation with exact match.

- The `res.py` is our script for visualizing performance of running and finished
  experiments, and for comparing two experiments. It was developed for our needs
  and we provide it as-is without documentation.

# The Released Models

Three pretrained CorPipe 25 models have been released under the CC BY-NC-SA 4.0 license:
- [corpipe25-corefud1.3-base-251101](https://huggingface.co/ufal/corpipe25-corefud1.3-base-251101)
  also available on [LINDAT/CLARIAH-CZ](https://hdl.handle.net/11234/1-6078);
- [corpipe25-corefud1.3-large-251101](https://huggingface.co/ufal/corpipe25-corefud1.3-large-251101)
  also available on [LINDAT/CLARIAH-CZ](https://hdl.handle.net/11234/1-6079);
- [corpipe25-corefud1.3-xl-251101](https://huggingface.co/ufal/corpipe25-corefud1.3-xl-251101)
  also available on [LINDAT/CLARIAH-CZ](https://hdl.handle.net/11234/1-6080).

See the respective model repositories for their performances and training
hyperparameters.

## Training a Multilingual `mT5-large`-based CorPipe 25 Model

To train a multilingual model on all the data using `mT5 large`, you should
1. run the `data/get.sh` script to download the CRAC 2025 data,
2. create a Python environments with the packages listed in `requirements.txt`,
3. train the model itself using the `corpipe25.py` script.

   For training a mT5-large variant with square-root mix ratios and without corpus ids, use
   ```sh
   tbs="ca_ancora cs_pcedt cs_pdt cu_proiel de_potsdamcc en_gum en_litbank es_ancora fr_ancor fr_democrat grc_proiel hbo_ptnk hi_hdtb hu_korkor hu_szegedkoref ko_ecmt lt_lcc no_bokmaalnarc no_nynorsknarc pl_pcc ru_rucor tr_itcc"

   python3 corpipe25.py --train --dev --treebanks $(for c in $tbs; do echo data/$c/$c-corefud-train.conllu; done) --batch_size=8 --learning_rate=6e-4 --learning_rate_decay  --adafactor --encoder=google/mt5-large --exp=corpipe25-corefud1.3-large --compile
   ```

## Predicting with a CorPipe 25 Model

To predict with a trained model, use the following arguments:
```sh
corpipe25.py --load ufal/corpipe25-corefud1.3-large-251101 --exp target_directory --epoch 0 --test input1.conllu input2.conllu
```
- instead of a HuggingFace identifier, you can use directory name – if the given path name exists,
  the model is loaded from it;
- the outputs are generated in the target directory, with `.00.conllu` suffix;
- if you want to also evaluate the predicted files, you can use `--dev` option instead of `--test`;
- optionally, you can pass `--segment 2560` to specify longer context size, which very likely produces
  better results, but needs more GPU memory.

## Running the Model on Plain Text

To run the model on plain text, first the plain text needs to be tokenized and
converted to CoNLL-U (and optionally parsed if you also want mention heads),
by using for example UDPipe 2:

```sh
curl -F data="Eve came home and Peter greeted her there. Then Peter and Paul set out to a trip and Eve waved them off." \
  -F model=english -F tokenizer= -F tagger= -F parser=  https://lindat.mff.cuni.cz/services/udpipe/api/process \
  | python -X utf8 -c "import sys,json; sys.stdout.write(json.load(sys.stdin)['result'])" >input.conllu
```

Then the CoNLL-U file can be processed by CorPipe 25, by using for example
```sh
python3 corpipe25.py --load ufal/corpipe25-corefud1.3-large-251101 --exp . --epoch 0 --test input.conllu
```
which would generate the following predictions in `input.00.conllu`:
```
# generator = UDPipe 2, https://lindat.mff.cuni.cz/services/udpipe
# udpipe_model = english-ewt-ud-2.17-251125
# udpipe_model_licence = CC BY-NC-SA
# newdoc
# global.Entity = eid-etype-head-other
# newpar
# sent_id = 1
# text = Eve came home and Peter greeted her there.
1	Eve	Eve	PROPN	NNP	Number=Sing	2	nsubj	_	Entity=(c1--1)
2	came	come	VERB	VBD	Mood=Ind|Number=Sing|Person=3|Tense=Past|VerbForm=Fin	0	root	_	_
3	home	home	ADV	RB	_	2	advmod	_	Entity=(c2--1)
4	and	and	CCONJ	CC	_	6	cc	_	_
5	Peter	Peter	PROPN	NNP	Number=Sing	6	nsubj	_	Entity=(c3--1)
6	greeted	greet	VERB	VBD	Mood=Ind|Number=Sing|Person=3|Tense=Past|VerbForm=Fin	2	conj	_	_
7	her	she	PRON	PRP	Case=Acc|Gender=Fem|Number=Sing|Person=3|PronType=Prs	6	obj	_	Entity=(c1--1)
8	there	there	ADV	RB	PronType=Dem	6	advmod	_	Entity=(c2--1)|SpaceAfter=No
9	.	.	PUNCT	.	_	2	punct	_	_

# sent_id = 2
# text = Then Peter and Paul set out to a trip and Eve waved them off.
1	Then	then	ADV	RB	PronType=Dem	5	advmod	_	_
2	Peter	Peter	PROPN	NNP	Number=Sing	5	nsubj	_	Entity=(c4--1(c3--1)
3	and	and	CCONJ	CC	_	4	cc	_	_
4	Paul	Paul	PROPN	NNP	Number=Sing	2	conj	_	Entity=(c5--1)c4)
5	set	set	VERB	VBD	Mood=Ind|Number=Plur|Person=3|Tense=Past|VerbForm=Fin	0	root	_	_
6	out	out	ADP	RP	_	5	compound:prt	_	_
7	to	to	ADP	IN	_	9	case	_	_
8	a	a	DET	DT	Definite=Ind|PronType=Art	9	det	_	Entity=(c6--2
9	trip	trip	NOUN	NN	Number=Sing	5	obl	_	Entity=c6)
10	and	and	CCONJ	CC	_	12	cc	_	_
11	Eve	Eve	PROPN	NNP	Number=Sing	12	nsubj	_	Entity=(c1--1)
12	waved	wave	VERB	VBD	Mood=Ind|Number=Sing|Person=3|Tense=Past|VerbForm=Fin	5	conj	_	_
13	them	they	PRON	PRP	Case=Acc|Number=Plur|Person=3|PronType=Prs	12	obj	_	Entity=(c4--1)
14	off	off	ADP	RP	_	12	compound:prt	_	SpaceAfter=No
15	.	.	PUNCT	.	_	5	punct	_	SpaceAfter=No

```

## How to Cite

```
@inproceedings{straka-2025-corpipe,
  title = "{C}or{P}ipe at {CRAC} 2025: Evaluating Multilingual Encoders for Multilingual Coreference Resolution",
  author = "Straka, Milan",
  editor = "Ogrodniczuk, Maciej and Novak, Michal and Poesio, Massimo and Pradhan, Sameer and Ng, Vincent",
  booktitle = "Proceedings of the Eighth Workshop on Computational Models of Reference, Anaphora and Coreference",
  month = nov,
  year = "2025",
  address = "Suzhou, China",
  publisher = "Association for Computational Linguistics",
  url = "https://aclanthology.org/2025.crac-1.11/",
  doi = "10.18653/v1/2025.crac-1.11",
  pages = "130--139",
}
```
