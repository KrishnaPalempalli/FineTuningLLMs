####Introduction And Purpose Of Task:####
Our original project for GW4 focused on fine-tuning Large Language Models (LLMs) to identify limericks, poems with a strict five-line AABBA rhyme scheme. This task was inspired by my own difficulty, during elementary school, in identifying rhyme schemes and checking whether my poems followed the limerick format. We haven’t seen any limerick classification LLM models and saw an opportunity: by training LLM models on both limericks and non-limericks, we aimed to create fine-tuned models that support poetry instruction, checking whether students’ limericks follow the right structure and provide a reasoning for them to understand/refine their work, showcasing the potential of LLMs in structured literary tasks.

####Setting Of Task:####
Initially, we aimed to fine-tune models to provide both classification and reasoning for their decisions. We designed a dataset with columns for text/poems, criteria (line count and rhyme scheme), labels (Limerick or Non-Limerick), and explanation. However, fine-tuning models capable of generating text like EleutherAI/pythia-410m and gpt2 proved challenging as they generated no answers or random answers for the classification, criteria, and explanation parts, making it hard to objectively judge the results. Because of this, for GW4, we simplified the task to focus purely on classification, having only poems and labels in the dataset. We selected albert-base-v2 and google/electra-base-discriminator for their efficiency in general classification tasks and compatibility with concrete evaluation metrics.

To train these models, we created a dataset of 300 examples (150 limericks and 150 non-limericks). We got limericks and non-limericks from sources referenced below. Limericks covered various themes and styles. Non-limericks included intentionally altered limericks with different rhyme schemes, other forms of poetry with different line counts and rhyme schemes. During training, the models processed this data as text-label pairs. Once fine-tuned, they classified new text into one of the two categories based on the highest-probability label, allowing us to objectively evaluate their ability to handle subtle challenges in rhyme and structure as discussed below.

####Task Execution And Results:####
We fine-tuned the albert and electra models for limerick classification, evaluating performance using accuracy, precision, recall, F1-score, and loss values. Albert outperformed electra on the held-out test set, achieving higher accuracy (.8 vs. .67) and greater training/validation loss reductions (.3/.15 vs. .2/.05). On 16 diverse baseline examples (8 limericks and 8 non-limericks with varied themes and structures), fine-tuned electra’s accuracy improved to .75, while fine-tuned albert achieved .875 accuracy, far surpassing its baseline of 0.44. Albert misclassified only two subtle non-limerick examples with rhyme scheme variations, showing the potential for further improvement with a larger dataset. Overall, fine-tuning was successful, with these fine-tuned showing promise for distinguishing limericks and non-limericks.

####Revision And Editing Task:####
That is where we stopped for GW4. However, I was dissatisfied because we failed to meet our original goal of fine-tuning LLMs to provide classifications with reasoning to help students understand whether their intended limericks follow the format and why. For the revise and edit task, I explored multi-label classification as a substitute for generating reasoning. I expanded the dataset by adding 300 examples, ensuring equal coverage of cases: 200 "5 Lines, AABBA Rhyme Scheme" Limericks, 200 "5 Lines, Not AABBA Rhyme Scheme" Non-Limericks, and 200 "Not 5 Lines, Not AABBA Rhyme Scheme" Non-Limericks. Each example now had these clear labels in a Reasoning column to explain its classification.

Then, I implemented code as seen in the revised code file to use this data to fine-tune albert-base-v2 and google/electra-base-discriminator models to do binary classification for "Limerick" and "Non-Limerick" and multi-label classification for reasoning attributes ("5 Lines," "Not 5 Lines," "AABBA Rhyme Scheme," and "Not AABBA Rhyme Scheme") using a shared architecture and multi-task learning. I encoded these from 0-5, with the first two classes for binary classification and the remaining four labels for multi-label classification for reasoning. The code uses a weighted combination of Cross-Entropy Loss for binary classification and Binary Cross-Entropy for multi-label classification, along with a custom trainer, BinaryClassMultiLabelTrainer, to handle the loss computation and backpropagation. Once the models are fine-tuned, limerick/non-limerick predictions are determined by selecting the class corresponding to the highest logit value among the classification indices, while reasoning predictions are made by identifying the two labels with the highest logit values among the reasoning indices. 

####New Task Execution And Results:####
I optimized hyperparameters like the number of epochs, learning rates, weights, etc. While many values led to overfitting or insignificant loss reductions, the best values, as shown in the code file, led albert and electra models’ training and validation loss to decrease by about .9 and .85 and by 1.05 and .88, respectively, showing promising performance. On the held-out test set, albert achieved a classification accuracy of .73 and reasoning accuracy of .84, while electra got a classification accuracy of .8 and reasoning accuracy of 0.875. These pretty high results indicate both models can classify unseen examples well and provide correct reasoning labels. 

Additionally, to compare results with the baseline, we tested the same 16 unseen examples from above. The base electra model misclassified 9 examples, mostly predicting "Non-Limerick" and assigning mostly incorrect reasoning labels, resulting in a classification accuracy of 0.43 and reasoning accuracy of 0, worse than random guessing. For instance, it labeled a limerick as “Not 5 Lines” and “AABBA Rhyme Scheme” and a non-limerick as “AABBA Rhyme Scheme” and “Not AABBA Rhyme Scheme.” In contrast, the fine-tuned electra model misclassified only 3 examples (2 non-limericks and 1 limerick), getting both accuracies to .81. It incorrectly assigned the label “AABBA Rhyme Scheme” to a non-limerick and “Not AABBA Rhyme Scheme” to a limerick. This demonstrates the fine-tuned model's improved ability to distinguish between limericks and non-limericks while providing mostly accurate labels, with errors occurring only in subtle rhyme scheme cases. 

Similarly, the base albert model misclassified 8 examples, always predicting "Non-Limerick" with mostly incorrect reasoning labels, resulting in classification and reasoning accuracies of 0.5 and 0.19, respectively. It assigned the same labels for every example, “5 Lines” and “Not AABBA Rhyme Scheme,” which were correct for only 3 cases, indicating it failed to grasp the poem's structure. In contrast, the fine-tuned albert model misclassified only 2 limerick examples, achieving both accuracies at 0.81 and incorrectly assigning the label “Not AABBA Rhyme Scheme” in these cases. This highlights its improved ability to classify and label correctly, with errors limited to subtle rhyme scheme variations.

Additionally, I tested model consistency by repeating the same limerick and non-limerick examples 8 times each. All models produced identical results across these repetitions, showing their output consistency.

####Conclusion:####
We can see the fine-tuned models with reasoning perform better on unseen examples than models without reasoning, achieving higher accuracy scores while associating reasoning labels with classifications. This reasoning feature is very useful for not only highlighting areas where the models struggle, but also adds educational value by helping students understand limericks and their format. However, the model is not deployment-ready due to its imperfect accuracy, particularly with rhyme scheme variations, which risks spreading incorrect information and hindering students' understanding. Overall, this revised project demonstrates the potential of fine-tuning LLMs for specialized tasks like limerick identification with pretty high accuracies (around 80%), while achieving our original goal of providing reasoning for the classifications using labels. 
Citations For Our Dataset And The 16 Test Examples (Jupyter Notebook)

####Dataset:####

Up to and including row 107: These limericks were taken from The little book of limericks / comp. and ed., with an introduction by Wallace and Frances Rice

From row 108 to 147 inclusive: Limericks in these rows were taken from Stanton Vaughns work "Limerick Lyrics" https://ia801908.us.archive.org/7/items/700limericklyric00vauguoft/700limericklyric00vauguoft.pdf
Non-limericks in these rows were basically modifications of the limericks in these rows. Some of them were modified by removing lines, or changing some words to create a different rhyme scheme, etc.

From row 148 to 171 inclusive: These non-limericks were taken from https://spot.colorado.edu/~downton/lifegardening/poems%20in%20little%20boxes.htm
Every three poems belong to one of the categories listed, and we go from the top to the bottom categories in order.

From row 172 to 174 inclusive: These non-limericks were taken from this following page and the pages linked on this page https://hyacinthreview.org/villanelles-poetic-form-guide/

From row 175 to 179 inclusive: These non-limericks were taken from this following page https://classicalpoets.org/2016/10/19/how-to-write-a-villanelle-with-examples/

From row 180 to 189 inclusive: These non-limericks were taken from this following page https://classicalpoets.org/2020/05/15/ten-great-spenserian-or-scottish-sonnets/

From row 190 to 220 inclusive: These non-limericks were taken from this following page https://reedsy.com/discovery/blog/haiku-poem-examples

From row 221 to 250 inclusive: We took the first 30 limericks from the dataset and just changed some worlds to throw off the rhyme scheme.

From row 251 to 252 inclusive: These non-limericks were taken from this following page and the pages linked on this page https://poets.org/glossary/cinquain

Row 253: This non-limerick was taken from the second stanza in the example poem on this page https://writers.com/cinquain-poetry#:~:text=A%20cinquain%20is%20a%20stanza,to%20play%20with%20cinquain%20poetry. 

From row 254 to 277 inclusive: We took the first 24 limericks on this page
https://www.physics.harvard.edu/undergrad/limericks

From row 278 to 287 inclusive: We took the first 10 quotes from this page https://blog.hubspot.com/sales/famous-quotes

From row 288 to 301 inclusive: We modified the first 14 limericks from rows 254 to 277 by changing like one word in each limerick to make it into a non-limerick (throw off the rhyme scheme).

Newly added rows in the dataset:

From row 302 to 351: I took the first 50 limericks on this page https://parade.com/1249429/marynliles/limericks/

From row 352 to 401: Non-limericks in these rows were basically modifications of the limericks in the rows 302 to 351. Some of them were modified by removing some words, or changing some words, etc. to throw off the rhyme scheme.

From row 402 to 421: I took these short poems from this page basically in order
https://briefpoems.wordpress.com/2016/01/07/slates-one-line-poems-monostich/ 

From row 422 to 442: I took these non-limericks from https://poets.org/poem/cry-cicada
https://poets.org/poem/haiku-0
https://poets.org/poem/seven-steps-heaven-haiku
https://poets.org/poem/japanese-hokku where some of these links included multiple non-limericks within them.

From row 443 to 459: I took these non-limericks from https://poets.org/poems?field_occasion_target_id=All&field_poem_themes_target_id=All&field_form_target_id=413&combine= in order from the beginning.

From row 460 to 484: I took all the non-limericks on this page https://poets.org/poems?field_occasion_target_id=All&field_poem_themes_target_id=All&field_form_target_id=424&combine=

From row 485 to 503: I took all the non-limericks on this page https://poets.org/poems?field_occasion_target_id=All&field_poem_themes_target_id=All&field_form_target_id=428&combine=

From row 504 to 509: I took these non-limericks from https://poets.org/poem/lost-bird in order.

For row 510: I took the non-limerick from https://poets.org/poem/saying-il-haboul

From row 511 to 522: I took the non-limericks from https://poets.org/poem/sainte-marguerite in order.

From row 523 to 527: I took the non-limericks from https://poets.org/poem/park-bench in order.

From row 528 to 536: I took the non-limericks from https://poets.org/poem/dream-and-shame in order.

From row 537 to 539: I took the non-limericks from https://poets.org/poem/helen-0 in order.

From row 540 to 543: I took the non-limericks from https://poets.org/poem/world in order.

From row 544 to 601: I took the limericks from rows 33 to 90 and modified them to be non-limericks by removing some words, or changing some words, etc. to throw off the rhyme scheme.
16 Test Examples: 

First four limericks were taken from Stanton Vaughns work "Limerick Lyrics" https://ia801908.us.archive.org/7/items/700limericklyric00vauguoft/700limericklyric00vauguoft.pdf

Next four limericks were taken from https://www.physics.harvard.edu/undergrad/limericks

First non-limerick was generated by one of our team members on their own

The next two non-limericks were taken from Spirit and Bed sections https://spot.colorado.edu/~downton/lifegardening/poems%20in%20little%20boxes.htm

The next two non-limericks were taken from the first four stanzas https://poets.org/anthology/long-poems

The last three non-limericks were just modifications of the earlier limerick test examples, where some words have been changed or removed to change the rhyme scheme.



