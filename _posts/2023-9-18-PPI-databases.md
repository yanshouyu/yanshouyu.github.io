---
layout: post
title: "Protein-Protein Interaction Databases"
---

To evaluate protein-protein interaction (PPI) experiment results, I've searched 4 PPI database as a cross-reference. Here's a brief summary on the 4 databases.

## [STRING](https://string-db.org/)

Of course STRING is more than just PPI, or we can say the interaction defined in STRING is much broader. It contains many different layers, including literature mining, experiments, co-expression, etc. If we want to narrow down the scope into experimentally validated network, we can search the "physical subnetwork" instead of the "full STRING network".

STRING data is downloadable in flat-file tables. One record represents an edge (interaction) between 2 nodes (i.e. proteins), which has several scores (0 to 1) on different aspects (i.e. channels), e.g. *experimentally_determined_interaction*, *database_annotated*, etc. There is also a *combined_score* computed by *"combining the probabilities from the different evidence channels and corrected for the probability of randomly observing an interaction."*

## [BioGRID](https://thebiogrid.org/)

BioGRID is also more than PPI because it is a *"Database of Protein, Genetic and Chemical Interactions"*. Its data is from curated literatures. It contains comprehensive ID mapping and synonyms. The curation on experiments is of very high quality, with details in literature, experiment system, throughput, score, and notes on qualification.

BioGRID data is freely downloadable under the MIT license. If we are only interested into one protein, we can first search it and download its interaction data.

Based on the high quality searching results I got for my task at hand, I'll mark BioGRID as a good source. 

## [IntAct](https://www.ebi.ac.uk/intact/home)

IntAct is hosted by EBI. Its data come from literature curation and user submission. It has a batch search by which we can input a group of proteins at once. Its website is very interactive. You can view the interaction network on the results and download both the table and the graph in various formats.

IntAct data is also freely available. 

## [Interactome3D](https://interactome3d.irbbarcelona.org/index.php)

Interactome3D is a webservice for the *"structural annotation of protein-protein interaction networks"*. Its current version (by 230918) is 2020_05. A detailed content version can be found in its subpage [Statistics](https://interactome3d.irbbarcelona.org/statistics.php).