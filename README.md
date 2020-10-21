# Swedish spaCy models

The National Library of Sweden / KB Lab releases two pretrained multitask models compatible with the NLP python package [spaCy](https://spacy.io/).  
A language specific model for Swedish is not included in the core models as of the latest release (v2.3.2), so we publish our own models trained within the spaCy framework.  
The models include a part-of-speech tagger, dependency parser and named entity recognition. We trained two separate models for Universal POS tags ([UPOS](https://universaldependencies.org/u/pos/)) and Language Specific POS tags ([XPOS](https://universaldependencies.org/sv/index.html)), as spaCy does not allow for joint training of both. 

## Training data and performance

We trained two separate tagger models for UPOS and XPOS, but the parser and NER are the same for both models. The models are initialized with [FastText](https://fasttext.cc/docs/en/crawl-vectors.html) word embeddings.

### UPOS tagger

The UPOS tagger was trained using two Swedish treebanks from Universal Dependencies, [UD_Swedish-Talbanken](https://universaldependencies.org/treebanks/sv_talbanken/index.html) and [UD_Swedish-LinES](https://universaldependencies.org/treebanks/sv_lines/index.html). The treebanks contain mainly news and non-fiction.

Evaluation result on the joint Talbanken and LinES test sets: **96.37**


### XPOS tagger

Since the Talbanken and LinES treebanks are annotated with different XPOS tagsets we could not use them to train the XPOS model. Instead we used Talbanken and the Stockholm-Umeå Corpus v3.0 ([SUC 3.0](https://spraakbanken.gu.se/en/resources/suc3)), which is a balanced corpus containing different types of news text, as well as fictional and non-fictional prose.  

Evaluation result on the Talbanken test set plus a held-out portion of SUC 3.0: **96.84**

### Dependency parser

The parser is trained on the Talbanken and LinES treebanks, and is equivalent in both models.

Evaluation results on the joint Talbanken and LinES test sets:  
Unlabelled Attachment Score (UAS) **87.03**  
Labelled Attachment Score (LAS) **82.20**

### NER

Named Entity Recognition was trained on the Stockholm Umeå Corpus v3.0 (SUC 3.0). Unlike pos tags, entities in SUC 3.0 were not annotated by human annotators, but automatically generated using [Sparv](https://spraakbanken.gu.se/en/tools/sparv/annotations). This means that they cannot be considered a gold standard.

Evaluation results on a held-out portion of SUC 3.0:  
Precision **86.27**  
Recall **84.48**  
F score **85.37**


## Usage

To download the models you can go to https://data.kb.se/datasets/2020/10/swedish_nlp/spacy/ and start the download manually. If you are running Ubuntu you can also use these commands:

```
wget https://data.kb.se/datasets/2020/10/swedish_nlp/spacy/sv_model_upos.zip
wget https://data.kb.se/datasets/2020/10/swedish_nlp/spacy/sv_model_xpos.zip
```

Once the models are downloaded and unzipped, they can be loaded into python with `spacy.load()`. Note that the path should point to the inner folder that contains the tagger, parser and ner folders.

```python
import spacy
nlp = spacy.load("path/to/model/inner/folder")
doc = nlp("Jag gillar London och Berlin.")
width = 15
for token in doc:
    print(f"{token.text: <{width}} {token.tag_: <{width}} {token.dep_: <{width}}")
    
for ent in doc.ents:
    print(ent.text, ent.label_)
```

Expected output:
```
Jag             PRON            nsubj          
gillar          VERB            ROOT           
London          PROPN           obj            
och             CCONJ           cc             
Berlin          PROPN           conj           
.               PUNCT           punct          
London LOC
Berlin LOC
```

IMPORTANT:   
In the UPOS model the part-of-speech tags are accessible via `token.tag_`, while `token.pos_` contains only unknown tags (X).  
In the XPOS model the trained pos tags can be accessed via `token.tag_`, whereas `token.pos_` contains UPOS tags generated through the Swedish tagmap that maps XPOS to UPOS, so they are not trained. This means that in the XPOS model, if the XPOS tag is incorrect, the UPOS tag is probably also incorrect.

## Acknowledgments

* Resources from [Universal Dependencies](https://universaldependencies.org/) were used to train the tagger and parser.
* Resources from Stockholms University, Umeå University and [Språkbanken](https://spraakbanken.gu.se/) at Gothenburg University were used to train the models for NER.
* A script by @EmilStenstrom was used to convert SUC 3.0 data into IOB format (which is compatible with the spaCy converter for NER). A modified version of the script was used to extract XPOS tags from SUC 3.0 in .conllu format in order to train one of the pos taggers.
* Many thanks to Joakim Nivre and Sara Stymne at Uppsala University for input regarding the choice of training data and licensing issues.
