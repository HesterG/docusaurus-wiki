---
sidebar_position: 3
title: Global Vectors for Word Representation
---

## Paper Summary

### Abstract

The GloVe (Global Vectors for Word Representation) model presents a new method for learning word vector representations. It combines the strengths of global matrix factorization and local context window methods. GloVe focuses on training with the non-zero elements in a word-word co-occurrence matrix, producing a vector space with meaningful semantic substructure. The model achieves high performance in word analogy tasks and outperforms other models in tasks like word similarity and named entity recognition (NER).

### 1. Introduction

The paper discusses how recent word vector representation models can capture fine-grained semantic and syntactic regularities, often evaluated by the word analogy task. Existing models fall into two categories: global matrix factorization methods (like LSA) and local context window methods (like skip-gram). Each approach has limitations. GloVe combines their advantages by leveraging word co-occurrence statistics globally and capturing meaningful linear substructures in the vector space.

### 2. Related Work

The two main approaches for learning word vectors are global matrix factorization methods (like LSA) and shallow window-based methods (like skip-gram). The paper highlights the strengths and weaknesses of these methods:

- **Matrix factorization** methods, such as LSA, decompose large matrices capturing word usage statistics but fail to capture word relationships well, especially for analogy tasks.
- **Window-based methods** like skip-gram and continuous bag-of-words (CBOW) are better at capturing word relations but do not use global co-occurrence statistics well.

### 3. The GloVe Model

The core idea behind GloVe is to utilize word co-occurrence statistics directly, learning from the ratio of word co-occurrence probabilities. By examining these ratios, the model captures relevant aspects of meaning. The GloVe model introduces a weighted least squares objective function to factorize the log of co-occurrence matrices, efficiently leveraging global statistics while down-weighting rare and frequent co-occurrences. This results in word vectors with more meaningful semantic structure.

### 4. Experiments

The authors evaluate GloVe on several tasks:

- **Word analogy tasks**: GloVe performs significantly better than other models, achieving up to 75% accuracy.
- **Word similarity tasks**: GloVe outperforms skip-gram and other baselines on datasets like WordSim-353 and RG.
- **Named Entity Recognition (NER)**: GloVe provides strong results compared to other vector-based models, showing its usefulness in downstream NLP tasks.

### 5. Conclusion

The paper argues that while both count-based and prediction-based word representation models probe co-occurrence statistics, GloVe’s efficient use of global statistics gives it an advantage. The GloVe model outperforms existing models in word analogy, word similarity, and NER tasks, combining the strengths of both global and local approaches. The authors emphasize that GloVe captures meaningful substructures in word vectors, making it an effective tool for various NLP tasks.

## Q&A

### Global vs. Local Word Representation Models

#### Background

Traditional word representation models can be broadly classified into two categories:

1. **Global Matrix Factorization Methods** (e.g., **Latent Semantic Analysis, LSA**):
   - **How it works**: LSA decomposes large matrices, such as term-document matrices, into lower-dimensional spaces, using techniques like Singular Value Decomposition (SVD) to capture statistical relationships between words across documents.
   - **Strengths**: LSA is effective at capturing broad statistical patterns in word co-occurrence, making it useful for tasks like information retrieval.
   - **Weaknesses**: LSA struggles with tasks like **word analogy** because it doesn't capture fine-grained relationships between specific word pairs. It tends to overgeneralize, missing the nuances in word meanings.

2. **Local Context Window Methods** (e.g., **Skip-gram from Word2Vec**):
   - **How it works**: Skip-gram models learn word representations by predicting the surrounding words (context) of a target word within a small window. This process captures local word relationships.
   - **Strengths**: Skip-gram excels at capturing **syntactic and semantic relationships** between words, making it highly effective in tasks like word analogy (e.g., "king is to queen as man is to woman").
   - **Weaknesses**: Skip-gram doesn't leverage the **global co-occurrence statistics** of a corpus, limiting its ability to learn broader patterns of word usage across an entire dataset.

#### The GloVe Model

GloVe (Global Vectors for Word Representation) overcomes the limitations of both approaches by combining their strengths:

1. **Global Co-occurrence Statistics**:
   - GloVe constructs a **word-word co-occurrence matrix**, where each entry represents how often two words appear together in the entire corpus. The model focuses on the **non-zero entries** to process meaningful word pairs and avoid the computational complexity of sparse data.

2. **Bridging Global and Local Models**:
   - **Global**: GloVe uses the full co-occurrence matrix to capture **global statistical relationships**, learning broad word patterns that matrix factorization methods handle well.
   - **Local**: GloVe also captures **fine-grained relationships** between word pairs, similar to how skip-gram performs in analogy tasks.

3. **Efficiency and Meaningful Relationships**:
   - GloVe is computationally efficient by focusing on non-zero co-occurrence entries, reducing the data complexity.
   - It captures **semantic relationships** by analyzing co-occurrence ratios (e.g., distinguishing between "ice" and "steam" by their relations to words like "solid" or "gas").

#### Conclusion

GloVe successfully combines global statistical insights with the ability to capture nuanced word relationships, providing a balanced and efficient method for word vector learning. It outperforms traditional models like LSA and skip-gram in various tasks such as word analogy, word similarity, and named entity recognition (NER), offering an effective and scalable solution for word representation.

## References

[original paper](https://nlp.stanford.edu/pubs/glove.pdf)
