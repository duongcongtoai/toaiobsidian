# LLM
Created: 2025-02-05
## Reading recommendation
Here are the highlights:  
  
1. Anthropic’s Prompt Engineering Interactive Tutorial  
The Google Sheets-based interactive exercises make it easy to experiment with different prompts and see immediately what works and what doesn’t. I’m surprised other model providers don’t have similar interactive guides: [https://lnkd.in/gqr5uQqg](https://lnkd.in/gqr5uQqg)  
  
2. OpenAI’s best practices for finetuning  
While this guide focuses on GPT-3, many techniques are applicable to full finetuning in general. It explains how finetuning works, how to prepare training data, how to pick training hyperparameters, and common finetuning mistakes: [https://lnkd.in/g7_kspz4](https://lnkd.in/g7_kspz4)  
  
1. Llama 3 paper  
The section on post-training data is a gold mine as it details different techniques they used to generate 2.7 million examples for supervised finetuning. It also covers a crucial but less talked about topic: data verification, how to evaluate the quality of synthetic data: [https://lnkd.in/g3ZaMAYZ](https://lnkd.in/g3ZaMAYZ)  
  
2. Efficiently Scaling Transformer Inference (Pope et al., 2022)  
An amazing paper co-authored by Jeff Dean about inference optimization for transformers models. It covers not only different optimization techniques and their tradeoffs, but also provides a guideline for what to do if you want to optimize for different aspects, e.g. lowest possible latency, highest possible throughput, or longest context length: [https://lnkd.in/gq2N7AUb](https://lnkd.in/gq2N7AUb)  
  
3. Chameleon: Plug-and-Play Compositional Reasoning with Large Language Models (Lu et al., 2023)  
My favorite study on LLM planners, how they use tools, and their failure modes. An interesting finding is that different LLMs have different tool preferences: [https://lnkd.in/g-F3Ayab](https://lnkd.in/g-F3Ayab)  
  
4. AI Incident Database  
For those interested in seeing how AI can go wrong, this contains over 3000 reports of AI harms: [https://lnkd.in/gvSWwTJ7](https://lnkd.in/gvSWwTJ7)  
  
5. I find case studies from teams that have successfully deployed AI applications extremely educational. Here are some of useful enterprise case studies. I'll add more case studies soon!  
- LinkedIn: [https://lnkd.in/gsRf2Sw6](https://lnkd.in/gsRf2Sw6)  
- Pinterest's Text-to-SQL: [https://lnkd.in/gF3zKHMf](https://lnkd.in/gF3zKHMf)  
- Gmail’s Smart Compose (2019): [https://lnkd.in/gWS9gqcE](https://lnkd.in/gWS9gqcE)  
- Grab: [https://lnkd.in/g7qRV4Fn](https://lnkd.in/g7qRV4Fn)
## Transformer network
Paper: https://arxiv.org/pdf/1706.03762 Attention is all you need
What?

Multi steps:
tokenize the words
embeding the words in to multi-dimensional array
so that words having similar context appear close to each other

Why?

As an improvement from recurrent layers where this model introduce steps that depend on each other, which disallow parallelism because later step relies on the output of previous steps

What is a layer in neural network?


Neural network is a combination of recognition being broken down into multiple steps, a.k.a it can only 1 thing at a time, 1 feature at a time. A layer is considered a learning phase for the network.


It is a collection of neurons that receive input and pass output to next layer, each layer has parameters (weight + bias)
## What is neuron
a tiny decision making unit

## Questions
- What is multi-headed attentions
- Positional embed encoding?
- How text auto-completion work given that the model output billions of suggestion (every possible words in the universe is assigned a probability)
- What is feed-forward network
- PCA (Principal component analysis)
## Matrix transpose

An act similar to exchange the values of row and column of the matrix.
Application:
- Covarriance compute: from `M x N` dimension matrix `C=1/m X^T X`
Application: high covrarriance is the symtomp of multicollinearity, which has negative effect on the model:
	- uncessarily high standard error
	- inefficient to run t-test

## Matrix inversion
Only square matrices can be inverted

Complexity: O(n^3)
Application:
- OLS: find the coefficient of linear model (with some limitation on complexity if the sample is too much, or too large number of feature, or high correlated features)
- backpropagation +  Newton method
- Kalman filters (used in robotics and time-series prediction) to update state estimate



## References
6. 
tags: 