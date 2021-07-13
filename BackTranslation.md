



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








