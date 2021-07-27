



##  Back Translation | 28/07/2021 - 

Adding to the data augmented with Syntax-Aware we further implement Back-Translation.

Back-translation is translating target language to source language and mixing both original source sentence and back-translated sentence to train a model. So the number of training data from source language to target language can be increased. This does both increasing the size for our dataset and adding noise to the data which inturn adds to the robustness of our translation model.

How is it done?
Let us understand this with a flow in mind, starting with what will be our requirements.
1. A dataset (in our case DBNQA - data.en and data.sparql)
2. Translation model for English-to-SPARQL (en-sp model)
3. Translation model for SPARQL-to-English (sp-en model)
4. And a little patience :P (good things take time.)

as we work on requirements,
We batch a subset of our data and translate English questions to respective SPARQL queries using the en-sp model.
> English Questions (from data.en) -> SPARQL queries (translated from en-sp)

To obtain more data we have two options:
1. We store these translations, and pass it to the sp-en model to translate them back to English questions
> SPARQL queries (translated from en-sp) -> English Questions (translated from sp-en)

2. We batch a subset of our data and translate SPARQL queries to respective English questions using the sp-en model.
> SPARQL queries (from data.sparql) -> English Questions (translated from sp-en)

Here, 2 clearly has advantage over the quality of translations to that of 1, but lacks to add noise we expect in our data.

This required creating a training pipeline for both setups. Involves following steps:
1. Getting the data
2. Preprocessing data
3. Defining translation model
4. Setting up optimiser, loss function and evaluation metrics 

The steps seemed to be quite quite straightforward but, while the at a certain step I encountered an issue very common with those related to large-scale datasets.
Firstly for the preprocessing part, 
1. Reading the data from respective files
2. Tokenization and building vocabulary
3. Converting sentences to sequences, padding accordingly

Looking at these from a normal language-language translation problem, say English-German (most common) the default vocabulary size is roughly around 50K-60K which is quite scalable with today's compute.
On the other hand our dataset, builds up a vocabulary size of ~130K for the English lang. and ~250K for the SPARQL lang. Now, for traning the NMT the target sequences needed to be one-hot encoded in order to fit for the loss - categorical crossentropy.

This simply was reshaping and creating a sparse vector for each word, from shape (1, N) -> (1, N, V) 
: where N-length of sequence and V-size of language vocabulary. 

Creating such a high indiced tensor consumed a large chunk of the memory each time causing either the system to crash or you sitting watching your screen get frozen for decades :P

One obvious initial solution that came to mind was to batch the sequences in order to lower the memory consumption, by encoding them right before they were given in to train the model. Tried out with various standard batching size (another hyperparameter to look out for) 128, 64, 32, and 16 all seem to make less or no difference by only delaying the inevitable crash by few moments.
Batch size of 8 seemed to make move, but again on a later point during training (right during the 1st epoch) it again caused memory leaks and the story repeated. 

After discussion with Tommaso and Anand we decided to try out subsampling the dataset in order to reduce the overall root of the problem - the huge size of vocabulary. Decided to sample 10% of the datapoints of the original dataset to test the idea. After obeserving and analyzing the dataset, every 300 examples had a overall same structure so decided to sample 10% (i.e 30) from each 300 questions. After sampling I had a scaled down version of the original dataset with ~89K pairs.

Finally, after few tweaks and fixes things were on track and the training was setup.
I decided to drop the one-hot vectors that was root to the main problem, using the sequences with tokens and changed the loss accordingly, "categorical cross entropy" -> "sparse categorical crossentropy".

The training went well, but the overall quality of translations was not acceptable. Possible reasons being using a smaller dataset, unappropriate hyperparameters and overall model architecture.

For instances,

|Actual|Predcited|
|------|---------|
|Which place founded by party city is also the location fo katy grannan ?|Which place is the city is te also the location is the ?|
|What television show are distributed by national geographic society?|What is the show are are by is the show ?|
|What is the debut team of charlie fisher ?|What is the debut of of is the ?|
|What is the style of architecture of macfarland library ?|What is the style of place of whcih is ?|
|Did cris vaccaro manage a club of puerto rico national football team|Did is what place is the is the the team|

Clearly the tranlsation system did not generalize at all to the smaller(10%) subset of dataset we tried to use. A clear indication to the point that the number of occurences of a given named entity were way more less than expected.

After brainstorming on the issue with mentors, we were sure that the subset selection was an inaapropriate approach choosen to sub sample. Instead suggested on trying and using a continous 10% section through the dataset 


