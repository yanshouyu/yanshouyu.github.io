---
layout: post
title: "Literature review: BERTology Meets Biology: Interpreting Attention in Protein Language Models, 2020"
---

Great demonstration of model interpretation from Salesforce research team.
[Project in Github](https://github.com/salesforce/provis). From the project name "provis" we can get of hint of the authors' initial design.

## Methodology

Linking information from:
- Protein attributes (AA/token level, token-pair level, Structure/aggregational level)
- Protein sequence (model input)
- Layer - Head activation (output & output_attn_weight)
- Probing tasks (transferability)


#### Model
Pre-trained BERT-base on protein sequences from [TAPE project](https://github.com/songlab-cal/tape), in huggingface the model is "Rostlab/prot_bert". Note that the authors planned to analyze other models (prot_albert, prot_bert_bfd, prot_xlnet) but for now (210519) only bert and xlnet are implemented (`compute_edge_features.py` line 116-123).

Dataset for pre-train: PFAM database, 31M sequences.
Pre-train task: masked language modeling of AA.

BERT-base: L=12, H=768, A=12 (12 layers, 768 hidden dimension, 12 attention heads). Thus each head deals with 64-dimensional-hidden states (768/12=64). In total 12\*12=144 attention mechanism. 

Implementation:
```python
from transformers import BertModel, AutoTokenizer
model = BertModel.from_pretrained("Rostlab/prot_bert", output_attentions=True)
tokenizer = AutoTokenizer.from_pretrained("Rostlab/prot_bert", do_lower_case=False)
```

#### Dataset

To select dataset for interogating the relationship of attention and protein property, we need it to be representative, unbiased, and **property-rich**. Off-the-shelf usage would be a bonus (already wrapped in TAPE). The authors' 2 choices are:
- ProteinNet Dataset (AlQuraishi, M.)
    - Based on CASP, multi-level properties: sequence, structure, MSA, PSSM.
    - Curated train/validation/test splits for machine learning researchers where test sets are taken from the CASP competition and sequence identity filtering is already performed.
    - ProteinNet provides AA coordinates which are then transformed into contact map.
- Secondary structure dataset from PDB

Data loading is done by TAPE ( `pip install tape-proteins`):
```python
from tape.datasets import dataset_factory
from tape.tokenizers import TAPETokenizer
...
...
class BindingSiteDataset(Dataset):
    ...
    # lmdb is a folder containing mdb files
    data_file = f'secondary_structure/secondary_structure_{split}.lmdb'
    self.data = dataset_factory(data_path / data_file, in_memory)
    ...
```

#### Indicator Functions

To bridge the attention output (including attn_weight) with protein features, the authors constructed several indicator functions.

- For token-pair property (contact maps), shape (S, S):
Indicator function is the sum of an **elementwise product** of property existance matrix (0/1) and attention weight matrix over all protein sequences, divided by sum of attention weight matrix.
- For token-level property (secondary structure, protein binding sites), shape (S):
Indicator function is the sum of positive columns divided by sum of full attention weight matrix. Here the positive column means AA positions where the token-level property exist. 

The `F.multi_head_attention_forward()`  can only return attention weights averaged by heads, thus the attention weights of each head must have been inspected individually. It is also an important practice to learn from this work (check `provis/protein_attention/attention_analysis/compute_edge_features.py`)

#### Probing tasks

The authors used probing tasks to assess the transferability of BERT.

With BERT weights freezed, a single dense layer with softmax on top forms a classifier. Later we'll see that the classifier layer were put on top of each layers to compare the performance.

- token level properties
    - input to classifier: BERT's output vector (dimension=768) of each token
    - Output along the protein sequence dimension;
- token-pair properties
    - input to classifier: pair-wise feature vector, shape (S*2, ), "concatenating the elementwise differences and products of the 2 tokens' output vectors". This is the same design as in TAPE.
    - Output: classification of the token-pair.

Metrics:
- Secondary structure prediction: F1 score
- Contact prediction: precision@L/5
- Binding site prediction: precision@L/20

precision@L/5: precision for the most probable 1/5 sites (ignoring other 4/5 sites) along the full length sequence.

The "5"/"20" selection represents the natural proportion. This is also the CASP standard practice.

#### Experimental details

Omitted tokens <CLS> and <SEP> ((`compute_edge_features.py` line 116-123): 

```python
if model_name == 'bert':
    # Remove attention from <CLS> (first) and <SEP> (last) token
    attns = [attn[:, :, 1:-1, 1:-1] for attn in attns]
elif model_name == 'xlnet':
    # Remove attention from <CLS> (last) and <SEP> (second to last) token
    attns = [attn[:, :, :-2, :-2] for attn in attns]
```
Note that each attention is a 4-D Tensor.

Set minimum attention value (default to 0.1) to clarify the pattern. This setting must have been experimented out of trial-and-error. A good visualization of temperary results is a huge enabler for efficient experiments.  
Implementation of masking by minimum attention value: 

```python
# calculate the denominator (分母)
mask = attns >= min_attn
weight_total += mask.long().sum((2, 3))

# calculate the numerator, skip the nested for loops
mask = attns[:, :, from_index, to_index] >= min_attn
feature_to_weighted_sum[feature_name] += mask * value
```

Truncate protein in analysis experiments to 512 to save memory. 6% sequences were truncated.

## Results

#### BERT's attention on each AA

The most natural token-level property is AA identity. For example, in sequence "ARDN", the property vector for "R" is [0, 1, 0, 0]. With this property vector and attention output in shape (4, 4), token-level indicator function can give us a scaler value representing the attention on "R" in the sequence. For multiple sequences, the indicator function calculates weighted average of the attention from each sequence, with their sequence length as weight (check the formula).

Repeating the calculation on all 144 attention \* 20 AA, we can get 2880 AA-level attention indication as figure 2 and appendix B.1, visualized in heatmap with shape (20-AA, 12-layer, 12-head).

Interestingly, 14 out of 20 AA has one attention head focusing on it (25% indicator ouput value). 

What I'm thinking: the remaining 6 out of 20 are more team-players, right?


#### Attention similarity between AAs

It is rational to compare the AA similarity from the perspective of the 144-dimensional attention. If the pre-trained prot_bert did caught some hidden representation, distance between 2 144-dimensional attention vectors should correlate with their chemical distance, which in bioinformatics is represented by substitution matrix, e.g. BLOSUM62 or PAM50.

The authors put the Pearson correlation of attention vectors between AA-pairs into a 20-by-20 symmetrical matrix and found a similarity with BLOSUM62 (Fig3). The 2 matrices shared a Pearson correlation at 0.80.

What I'm thinking: It would also be very interesting to examine the most different part between attention similarity and substitution matrix - could it be where the light of evolution dimmed a little?

#### Secondary structure

Another token-level property. Heatmaps in Appendix B.2. The attention distribution seems quite even, especially for helix. This is also reasonable since secondary structure is a very elemental property.

#### Tertiary structure (contact map)

Contact map is at token-pair level. The 12-by-12 indicator result in fig4 clearly shows there is one head (12-4) specialized for contact prediction.

Explanation from language model:
> "knowing contacts of masked tokens could provide valuable
context for token prediction"

The authors then took a closer look on attention weights from head 12-4: binning AA pairs by attention weights and calculate the real contact percentage (fig 5).

Note that here the "AA pair" is exact pair on sequences, not the aggregated 20\*20 AA pairs. An example data might look like this: 
```
joint-id: prot1 - Arg12 - Leu25, 
attention weight from head 12-4: 0.8
is_contact: True
```
What I'm thinking: 
1. How about at a finer binning size? 
1. Any aggregated AA pairs in the 20\*20=400 pairs worth investigating more? E.g. Are there certain AA-pairs whose contact ratio do not align under attention weight bins?

#### Binding sites

Binding sites is also a token-level property, thus it means if the specific AAs in the protein is an interface to ligands (or other proteins?).

Attention indicators on binding sites are not very concentrated. But deeper layers do have more. Head 7-1 has top indicator value, 34%.

The authors then took the most correlated head (7-1) and plotted similar "ratio on binned attention" bars. The correlation is not as perfect as contact map due to low recall on high indicator value part (fig 7).

Although the binding site label is token-level, the authors examined token-pair weights generated from head 7-1 on some examples, hoping to find long-range interactions affecting binding site behavior. No solid conclusion was drawn in this part. But such long-range attention weights might open new teritory for protein engineering.

What I'm thinking: What if subclassing binding sites to AA - ligand, AA - AA, or even AA - nucleotide?

#### Cross-layer analysis

There's not very much to say along the layer-averaged attention indicator values, only that contact and binding site tend to be focused on deeper layers (fig 8).

The probing tasks were actually performed by putting a classifier on top of each layer. The authors then assessed layer-wise performance increasement, e.g. $performance(layer_2 + classifier_2) - performance(layer_1 + classifier_1)$. Although the classifier structure is the same, their weights were trained individually.

Fig 9 showed consistent information processing of secondary structure in early layers. Classification of other 2 properties (binding sites, contact) benefitted from deeper layers. Probing helix and strand were easier than on trun/bend (Appendix C, fig 16). Note that the probing tasks used simple top layer so the performance can only indicate information process, not for hitting a high value.

The contact map performance drop on final layer was explained as:
> "... the final layer outputs are not used to compute attention, but rather as an input to the final classifier layer that predicts the masked token for the language model. Whereas attention is a property of token pairs, the predictions are specific to an individual token. Therefore, contact relationships, which exist between tokens, may not be as useful to the model in the final layer."

Based on the explanation above the authors sugguested using the seoncd to last layer for embedding for downstream pairwise tasks.

## Summary

Section 5 to 7 ("related works", "conclusion", "broader impact") form a concise review. The works mentioned there worth a thorough read.

The authors also warned the bias in pre-train dataset "may have unknown biases across the tree of life".

## Comments

The project repository is an excellent example on model interpretability. 
- Experiments are documented in bash scripts clearly named.
- The executable python scripts and packages are placed beside the experimental bash scripts.
- Installable project package for easy integration and generalization.
- Clear separation of calculation and analytical report/visualization.
- Example data within repository.
- Additional data downloadable from AWS S3 with commands written in Readme. (Another practice for complex data: In TAPE project there's download script for additional post-downloading processing.)
- Notebooks to assist manuscript writing and also as good starter material.

What I'm thinking: 
- Comparing the "linguistics" of protein language and "linguistics" in real world, labelled data for the former could have much higher dimensionality. And the data size might already be adequate for many projects.
- The power of self-attention comes ultimately from evolution.
- Have anyone explored other transformer settings than the BERT-base (L=12, H=768, A=12) for protein?
- Will fine-tuning the protBert on a certain species enlighten us on the species-specific context? Will translation between species proteome be a way to humanization or horizontal gene transfer?

## Ref readings

- [LilianWeng's blog on attention mechanism](https://lilianweng.github.io/lil-log/2018/06/24/attention-attention.html)
- [The clearest illustration of attention calculation I've found](https://hackernoon.com/essential-guide-to-transformer-models-in-machine-learning-dzz3tk8)
- [A guide on using transformer-based models](https://towardsdatascience.com/how-to-use-transformer-based-nlp-models-a42adbc292e5)

Andrew Ng's comment on this work from "The Batch" newsletter 210602:
> "**Why it matters**: A transformer model trained only to predict missing amino acids in a sequence learned important things about how amino acids form a larger structure. Interpreting self-attention values reveals not only how a model works but also how nature works."