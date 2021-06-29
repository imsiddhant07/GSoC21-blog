---
title: Syntax-Aware Augmentation (07/06/2021-27/06/2021)
theme: jekyll-theme-cayman
filename: Syntax-AwareAugmentation.md


--- 




### Project overview

My project **"Neural QA Model for DBpedia"** has an end term goal of helping normal users (non-dev) explore the huge knowledge graphs at DBpedia by putting in questions in thier native laguage (English as of now). Ahh, but is answering a question such a big deal for a project? Kindoff yes.

Accessing these huge knowledge graph is only possible using querys - specificlly SPARQL queries, now for a normal person writing a SPARQL query is too much to expect. Thus we come bridging the gap - Translating your Native language question to SPARQL query. Yes, a slightly inlined towards machine translation project.

Also this is the 4th year for the project to be selected in Google Summer of Code, the project has evolved very well over the period of time. Taking it further we intend to bring it few steps closer to perfect ;) That being said, I'll be using two major datasets DBNQA and Lc-QuAD both as training and testing respectively.

### Timeline 

For the first 3 weeks (07/06/2021 - 27/06/2021) I shall be implementing "Syntax Aware Augmentation" as a resort of making the model robust to noise and extending the current state of dataset.


## Syntax-Aware Augmentation | 07/06/2021 - 27/06/2021

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

### Dependency parsing
is the task of analyzing the syntactic dependency structure of a given input sentence S. The output of a depen-dency parser is a dependency tree where the words of the input sentence are connected by typed dependency relations.

Transition based approach to dependency parsing is motivated by a stack-based approach called shift-reduce parsing originally developed for analyzing programming languages. This classic approach is simple and elegant, employing a context-free grammar,  a stack,  and a list of tokens to be parsed. Whereas,
Graph-based approaches to dependency parsing search through the space of possibletrees for a given sentence for a tree (or trees) that maximize some score.   Thesemethods  encode  the  search  space  as  directed  graphs  and  employ  methods  drawnfrom graph theory to search the space for optimal solutions

Sentences are majorly classified under projective and non-projective, in which the dependency lines do not intersect at any given section for the before one. Experimented for both transition-based and graph-based dependency trees with an endterm moto for probabilities for word selection. Came to conclusion for using Neural transition based parsing tree as it has obvious advantage over the parsing accuracy and execution time.

The parsing gives us an output tree, which we use to find the depths and relative probabilites of words present in the sentence. A glance at the output tree for the sentence 
> "What is the nationality of the guy gavriel kay which is also the  sovereign state of the françois langelier?"

![Dependency parsing tree](/assets/Tree.png)

By getting the depth from the tree :
>['What', 'is', 'the', 'nationality', 'of', 'danyon', 'loader', 'which', 'also', 'the', 'sovereign', 'state', 'waitomo', 'caves', 'hotel']
>
>[2, 1, 3, 2, 3, 5, 4, 2, 2, 3, 3, 2, 4, 0, 1]

Using depths to calculate final probabilites :
>['What', 'is', 'the', 'nationality', 'of', 'danyon', 'loader', 'which', 'also', 'the', 'sovereign', 'state', 'waitomo', 'caves', 'hotel']
>
>[0.87144306, 0.7, 0.95085018, 0.87144306, 0.95085018, 1.00603215, 0.98809904, 0.87144306, 0.87144306, 0.95085018,0.95085018, 0.87144306, 0.98809904, 0.7]

Based on the length of the sentence we decide on the number of augmentations to be performed, accordingly selecting words with maximum probability "dayon" -> 1.00603215 & "loader" -> 0.98809904 (here, 2) and performing operations.

### Replacement

* Trained a word embedding (Word2Vec) on the Text8 corpus for obtaining the word vector for the selected words.
* Made a search in the embeddings' dimensions (100-d for ours) around a fixed space with relative similar vectors.
* Found the most similar words by comparing the cosine similarity for the vectors for words, with 1 being high and 0.5 realtively low. Selected the word with highest cosine sim. value.

for "loader"
> embeddings : [ 2.87076589e-02  6.55337214e-01  1.83130503e-01  6.36373907e-02  1.72789529e-01 ... 2.46303618e-01  8.24105918e-01]
>

top 5 words :
> ('loader', 0.9999998807907104), ('operating', 0.6293714642524719), ('booting', 0.6258991360664368), ('firmware', 0.6177793145179749), ('device', 0.6041673421859741)

Following the process, we create new pair of data.

Original Questions | Augmented Questions |
----------|-------------------- |
Is william b. ochiltree a member ofof binge v. smith? | Is william b. ochiltree the membership of binge v. smith? |


### Dropout 
Pretty straight forward, using the same probabilities we drop words from the sentence. Considering the length of the sentence we drop words.

Original Questions | Augmented Questions |
----------|-------------------- |
Is william b. ochiltree a member ofof binge v. smith? | Is william b. ochiltree   member ofof binge v. smith? |
What are the sports played by the universities who also plays interscholastic league of honolulu? | What are the sports played by the universities who  plays interscholastic  of honolulu ? |

