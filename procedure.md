# Efficient Constituency Tree based Encoding for Natural Language to Bash Translation  
[Read the paper](https://aclanthology.org/2022.naacl-main.230.pdf)

## Aim:  
The paper aims to solve a very fundamental issue in modern day programming for beginners and intermediate level programmers. We are attempting to convert natural language (prompts) into **Bash commands used in Unix-like command-line interfaces**. This is a very meaningful project as the learning curve of Bash and command prompt are extremely steep and need a good amount of domain expertise. 

The model we propose in this paper takes in **natural language invocations** from the user and outputs two values. One being the corresponding bash command and the other being the confidence-score. The confidence-score tells us how confident the model is with the bash command it has generated.  
> _Note: This score is derived from the model’s beam search ranking and reflects relative confidence in each generated output._

There have been previous models that have tried to achieve the same goal. Previous models use a typical encoder-decoder model architecture. This treats the natural language given to it as a sequence of tokens and feeds it into the encoder section of the transformer. This often results in the loss of a lot of information. This is because the syntactic relations between words is lost to a great extent when we convert them into a mere sequence of tokens. 

## The Proposed Solution: Segmented Invocation Transformer (SIT)  
Older models did not use the inherent structure of the natural language. Meaning it would try to decode every single bit of the given natural language input. This treats the sentence as a flat sequence and may ignore structural cues that could simplify decoding.

Usually, when we give natural language commands to convert to Bash commands, certain important keywords can simplify the problem drastically. One of the examples I could think about was:

**Natural Language input:** Delete all the files in the hello folder in my desktop.  
**Required Output**: `rm ~/Desktop/hello/*` (+ the confidence score)  
Instead of looking deep into the natural language input to understand what it means, we can look for keywords. Here our keyword is _Delete_. This narrows it down to a command like `rm`. Now building on that, _in my desktop_ gives us the next part `~/Desktop`. _hello folder_ gives us the next part `/hello`. Finally _all the files_, gives us `/*`. Concatenating, we get `rm ~/Desktop/hello/*`.

We could find segmentation in the natural language to make our work way easier and to reduce the loss of context.

## How we approach the problem

### Normalization:  
We need to clean up each input in such a way that complex patterns are replaced with simpler patterns with the same exact meaning. This is done because we do not want the model to get confused over minor variations.

### Constituency Parsing:  
This is a key part of natural language. Parsing is a key practice in natural language where we build on the relationship words have with each other in a sentence. Prof. Christopher D. Manning, of Stanford talks about dependency parsing in his course CS224N. I have written a detailed explanation of how dependency parsing works and how earlier machine learning enthusiasts tried to implement it ([here](https://vrishank.super.site/2199f98a94ef8072bd0fe2d82edff486)). We represent the parsed sentence as a tree. It cuts through the tree at a certain height. Then we end up with subtrees, each subtree is a segment.  
> The **leaf nodes** of each subtree contain the individual tokens in a segment, but not necessarily "keywords". The segmentation helps us extract **meaningful chunks** that align with components of Bash commands.  
Let's say we started off with N words, we end up making K segments. Where K is lesser than N.

### SIT's Key Modification  
Now that we have made K segments out of N words. We take each one of the K segments individually. A typical transformer architecture takes each of these segments and makes embeddings for each token. Now, instead of doing that we take the embeddings for each token in the segment and average them all out over each segment. This is referred to as the **Averaging Layer**.  
> While segment averaging may lose some fine-grained details, it provides the model with dense, semantically focused inputs aligned to command structure.  
This reduces the amount of tokens given to the LLM from an order of N to an order of K. Highly optimizing the process. Not only does it reduce the amount of tokens the transformer has to process, it also makes sure the transformer receives meaningful segments which we have made from slicing the tree.  
> The rest of the model remains a standard Transformer decoder, but now receives a more structured and condensed input representation, improving both performance and interpretability.

## Key advantages:  
- **Faster inference time** because the transformer has lesser tokens to process (1.8 times faster on CPU).  
- It uses way **lesser parameters** as compared to the previous best model (5 times lesser).  
- **More accurate commands**, as we have already parsed the key values into keywords using the constituency parser.  
> _This contributes to improved accuracy for flags and utilities, as confirmed by attribution analysis._
