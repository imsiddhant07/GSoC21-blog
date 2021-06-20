
---
title: Week 1 (07/06/2021-13/06/2021)
theme: jekyll-theme-cayman
filename: week2.md


--- 


##  Week 2 | 14/06/2021 - 20/06/2021 

### Project overview

My project **"Neural QA Model for DBpedia"** has an end term goal of helping normal users (non-dev) explore the huge knowledge graphs at DBpedia by putting in questions in thier native laguage (English as of now). Ahh, but is answering a question such a big deal for a project? Kindoff yes.

Accessing these huge knowledge graph is only possible using querys - specificlly SPARQL queries, now for a normal person writing a SPARQL query is too much to expect. Thus we come bridging the gap - Translating your Native language question to SPARQL query. Yes, a slightly inlined towards machine translation project.

Also this is the 4th year for the project to be selected in Google Summer of Code, the project has evolved very well over the period of time. Taking it further we intend to bring it few steps closer to perfect ;) That being said, I'll be using two major datasets DBNQA and Lc-QuAD both as training and testing respectively.

### Timeline 

For the first 3 weeks (07/06/2021 - 27/06/2021) I shall be implementing "Syntax Aware Augmentation" as a resort of making the model robust to noise and extending the current state of dataset.


### Syntax-Aware Augmentation

This technique is being incorporated with the main motive of increasing the robustness and accuracy of NSpM, along with extending the existing DBNQA dataset via Augmentation.

Primarily focusing on three sub techniques:

* **Dropout** : Words in sentence are dropped 
* **Blanking** : Words in sentence are randomly replaced with a special placeholder token \<BLANK\>
* **Replacement** : Words in sentence are selected and replaced with one word which has a similar unigram word frequency over the dataset.

Rather than using word frequency or random selection we have used **Dependency tree** depth as a hueristic clue. It is well known that the meaning of a sentenceis actually determined by only a few of importantwords. Thus modifying those syntactically and semantically more important words may alter the sentence more radically.We abstain from choosing too important words.  To meet such requirements, we need to find a heuristic clue to measure how much important a word is, and then determine selecting probability to alter the corresponding word for data augmentation.

Given a sentence s=w<sub>1</sub>,w<sub>2</sub>,...,w<sub>n</sub> of length n, we first calculate a probability q<sub>i</sub> based on d<sub>i</sub>,the depth of word w<sub>i</sub>, by

q<sub>i</sub> = 1 - (1 / 2<sup>d<sub> i</sub> - 1</sup>) 

Q = {q<sub> i</sub>}

P(S) = Softmax(Q(S))

P(S) = {p<sub>1</sub>, p<sub>2</sub>, .. p<sub>n</sub>}

This week Experimented for both transition-based and graph-based dependency trees with an endterm moto for probabilities for word selection.


Also I feel I have been a little sloppy with the work, I'll have to gear up :))
