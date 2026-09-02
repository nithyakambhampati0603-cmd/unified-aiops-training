# Day 8 Research Homework -- Answers & Explanation

> **Note:** This document answers the three homework items from the
> uploaded sheet. The Contract Copilot scenario and its query/retrieved
> context are taken directly from the homework image.
> fileciteturn0file0L2-L8

------------------------------------------------------------------------

# 1. Word2Vec: How does it generate a vector for a word?

## Simple idea

Word2Vec converts a word into a list of numbers called a **vector** or
**embedding**.

Example:

`vendor → [0.12, -0.44, 0.87, 0.03, ...]`

The important idea is:

> **Words that appear in similar contexts learn similar vectors.**

For example, if **vendor**, **supplier**, and **partner** often appear
in similar sentences, their vectors may be close to each other in vector
space.

## How Word2Vec learns

### Step 1 -- Start with text

Example corpus:

-   The vendor uses confidential data.
-   The supplier processes confidential data.
-   The vendor signed the agreement.

### Step 2 -- Create a vocabulary

The model identifies unique words:

`[the, vendor, uses, confidential, data, supplier, processes, signed, agreement]`

Each word initially has an approximate/random numeric representation.

### Step 3 -- Use a context window

Suppose the sentence is:

`The vendor uses confidential data`

For the target word **vendor**, a context window might contain:

`the, uses, confidential`

Word2Vec learns relationships such as:

`vendor → uses`

`vendor → confidential`

### Step 4 -- Train the neural model

Two common Word2Vec architectures are:

-   **CBOW (Continuous Bag of Words):** predict the target word from
    surrounding words.
-   **Skip-gram:** predict surrounding/context words from the target
    word.

Example using Skip-gram:

**Input:** `vendor`

**Expected context:** `the, uses, confidential`

During training, the model changes its internal weights whenever its
prediction is wrong.

### Step 5 -- The learned weights become the embedding

After many training examples, the hidden/embedding layer contains a
learned vector for each word.

Example:

`vendor → [0.12, -0.44, 0.87, ...]`

`vendor` and `supplier` may have vectors that are closer than `vendor`
and `banana`.

------------------------------------------------------------------------

## Word2Vec flow

``` mermaid
flowchart LR
    A[Training Documents] --> B[Tokenization]
    B --> C[Build Vocabulary]
    C --> D[Create Target + Context Pairs]
    D --> E{Architecture}
    E -->|CBOW| F[Context predicts Target]
    E -->|Skip-gram| G[Target predicts Context]
    F --> H[Neural Network Training]
    G --> H
    H --> I[Update Weights]
    I --> J[Learned Word Vectors]
    J --> K[Similarity / Nearest-Neighbor Search]
```

## Important clarification about "ANN"

The term **ANN** can mean two different things:

1.  **Artificial Neural Network** -- Word2Vec uses a shallow
    neural-network-based training approach to learn embeddings.
2.  **Approximate Nearest Neighbor** -- ANN indexes are commonly used
    **after embeddings are generated** to quickly find similar vectors.

So, technically:

> **Word2Vec generates embeddings through neural-network training. ANN
> search is usually used later to retrieve nearest vectors
> efficiently.**

------------------------------------------------------------------------

# 2. BM25 vs Sentence-Transformer Embeddings

The homework asks for "embedding" using BM25 and sentence-transformers.
These two approaches are fundamentally different.

## Example documents

### Query

`Can the vendor use our data to train AI models?`

### Document A

`The recipient shall use Confidential Information solely to evaluate the proposed partnership and for no other purpose, including training, fine-tuning, or improving any machine learning or AI model.`

### Document B

`This Agreement shall remain in effect for two years from the Effective Date.`

------------------------------------------------------------------------

## A. BM25

BM25 does **not** create a dense semantic embedding like Sentence
Transformers.

Instead, BM25 represents relevance through **term frequencies, document
frequency, and length normalization**.

The main scoring formula is:

\[ BM25(Q,D)=`\sum`{=tex}\_{t `\in `{=tex}Q} IDF(t)`\times`{=tex}
`\frac{f(t,D)(k_1+1)}`{=tex}
{f(t,D)+k_1(1-b+b`\times `{=tex}\|D\|/avgdl)} \]

Where:

-   **Q** = query
-   **D** = document
-   **f(t,D)** = frequency of term `t` in document
-   **IDF** = importance/rarity of the term
-   **k1** = term-frequency saturation parameter
-   **b** = document-length normalization parameter

### Intuitive BM25 representation

After preprocessing, the query may become approximately:

`[vendor, use, data, train, AI, models]`

Document A contains related lexical terms such as:

`use, data, training, machine, learning, AI, model`

Document B contains mostly unrelated terms:

`agreement, effect, two, years, effective, date`

Therefore:

-   **Document A → higher BM25 relevance**
-   **Document B → lower BM25 relevance**

### Key limitation

BM25 mainly depends on lexical overlap. If the query says:

`Can the supplier train models using information?`

and the document says:

`The recipient shall not use confidential information for machine learning`

BM25 may miss some semantic similarity because the exact words differ.

------------------------------------------------------------------------

## B. Sentence-Transformer Embedding

A Sentence Transformer converts the whole sentence or chunk into one
dense vector.

Conceptually:

`"Can the vendor use our data to train AI models?"`

↓

`[0.21, -0.08, 0.67, 0.11, ..., 0.34]`

The real vector usually contains hundreds of dimensions, depending on
the model.

Similarly:

`"The recipient shall not use confidential information for training..."`

↓

`[0.19, -0.06, 0.71, 0.09, ..., 0.31]`

Because both texts express a similar meaning, their vectors should be
close in embedding space.

Similarity is often calculated using **cosine similarity**:

\[ CosineSimilarity(A,B)=`\frac{A\cdot B}{||A||\,||B||}`{=tex} \]

A simplified illustrative result could look like:

  Comparison              Conceptual similarity
  --------------------- -----------------------
  Query vs Document A           High, e.g. 0.90
  Query vs Document B            Low, e.g. 0.10

> These numbers are **illustrative**, not actual output from a specific
> embedding model.

------------------------------------------------------------------------

## BM25 vs Sentence Transformer

  -----------------------------------------------------------------------
  Feature                 BM25                    Sentence Transformer
  ----------------------- ----------------------- -----------------------
  Representation          Sparse lexical features Dense vector

  Understands meaning     Limited                 Stronger

  Uses exact terms        Strongly                Not required

  Synonyms/paraphrases    Weak                    Better

  Example                 `vendor` vs `vendor`    `vendor` can be similar
                                                  to `supplier`

  Retrieval style         Keyword-based           Semantic/vector-based

  Best use                Exact terms, legal      Meaning and paraphrased
                          keywords                questions
  -----------------------------------------------------------------------

## Practical recommendation for Contract Copilot

A **hybrid retrieval** approach is often better:

`BM25 + Dense Embedding Retrieval → Combine / Rerank → Best Chunks`

Why?

-   BM25 helps with exact legal terms such as **Section 4.2**,
    **Confidential Information**, and **Effective Date**.
-   Dense embeddings help when the user asks the same meaning using
    different wording.

------------------------------------------------------------------------

# 3. Contract Copilot -- Diagnose the Wrong RAGAS Metrics

## Given use case

### User query

> "Does this NDA let the vendor use our data to train their AI models?"

### Retrieved Context 1

The relevant clause says that confidential information can be used
solely for evaluating the partnership and **not for other purposes,
including training, fine-tuning, or improving a machine learning or AI
model**.

### Retrieved Context 2

The agreement remains in effect for two years from the Effective Date.

### Generated answer

The generated response gives a generic explanation about confidentiality
clauses and says that NDAs commonly protect sensitive information, but
it does **not directly answer the user's question**.

------------------------------------------------------------------------

## What should the correct answer be?

A better grounded answer would be:

> **No. Based on the retrieved clause, the recipient is not allowed to
> use the confidential information to train, fine-tune, or improve a
> machine-learning or AI model. The information may only be used to
> evaluate the proposed partnership.**

This answer is:

-   Direct
-   Grounded in the retrieved clause
-   Relevant to the question
-   Specific to the contract

------------------------------------------------------------------------

# Why some RAGAS metrics can be misleading or wrong here

## 1. Context Precision may be high

**Context Precision asks:**

> Did the retriever place relevant context near the top?

Here, Context 1 is highly relevant and ranked first.

Therefore, **high Context Precision can be valid**.

But high Context Precision does **not** mean the final answer is
correct.

### Key lesson

`Good retrieval ≠ Good answer`

------------------------------------------------------------------------

## 2. Context Recall may also look good

The key information required to answer the question was retrieved:

-   no use for training
-   no use for fine-tuning
-   no use for improving AI/ML models

So Context Recall may be high because the required evidence is present.

### Key lesson

`Relevant evidence retrieved ≠ Evidence correctly used by the LLM`

------------------------------------------------------------------------

## 3. Answer Relevance should be low

The user asked a very specific yes/no question:

> Can the vendor use our data to train AI models?

But the generated answer talks generally about:

-   confidentiality clauses
-   sensitive business information
-   defined terms
-   mutual obligations

It avoids the core issue: **AI training permission**.

Therefore, Answer Relevance should be **low**.

### Why?

The answer may sound professional, but it does not answer what the user
actually asked.

------------------------------------------------------------------------

## 4. Faithfulness can be tricky

Faithfulness checks whether the generated answer is supported by the
retrieved context.

The generated answer does not introduce a dramatic false fact. However,
it also fails to use the strongest and most important evidence:

> training, fine-tuning, or improving an AI/ML model is prohibited.

Therefore, a reasonable evaluation discussion is:

-   The answer may receive a moderate/high faithfulness score if most
    statements are generally compatible with the context.
-   But this can be misleading because **faithfulness alone does not
    measure whether the answer actually answered the question**.

### Important distinction

`Faithful ≠ Complete`

`Faithful ≠ Relevant`

A response can be factually safe but still be useless.

------------------------------------------------------------------------

## 5. Context Relevance / Context Utilization problem

The second retrieved chunk about the two-year agreement duration is
unrelated to the user's question.

So the system retrieved:

-   **Chunk 1: highly relevant**
-   **Chunk 2: irrelevant**

This shows that retrieval was not perfect, even though the most
important chunk was ranked first.

------------------------------------------------------------------------

# Metric diagnosis summary

``` mermaid
flowchart TD
    A[User asks about AI training permission] --> B[Retriever returns context]
    B --> C[Relevant clause: AI training prohibited]
    B --> D[Irrelevant clause: 2-year duration]
    C --> E[Generator should use relevant evidence]
    D --> E
    E --> F[Generated generic answer]
    F --> G[Does not directly answer YES/NO]
    C --> H[Context Precision can still be high]
    C --> I[Context Recall can still be high]
    F --> J[Answer Relevance should be low]
    F --> K[Faithfulness alone may be misleading]
```

------------------------------------------------------------------------

# 4. End-to-End Chunking and Embedding Flow for this Contract Copilot Use Case

## The complete ingestion flow

``` mermaid
flowchart TD
    A[Contract / NDA PDF] --> B[Document Parsing]
    B --> C[Extract Text + Preserve Metadata]

    C --> D[Document-Aware Chunking]
    D --> D1[Section 4.1]
    D --> D2[Section 4.2]
    D --> D3[Section 4.3]

    D2 --> E[Create Chunk with Context]
    E --> E1["Chunk text: Recipient shall not use Confidential Information for training, fine-tuning, or improving AI/ML models"]

    E1 --> F[Sentence-Transformer Embedding Model]
    F --> G[Dense Vector]
    G --> H[Vector Database / ANN Index]

    A --> I[BM25 Index Creation]
    C --> I

    J[User Query] --> K[Query Processing]
    K --> L1[BM25 Query]
    K --> L2[Embedding Model]
    L2 --> M[Query Vector]

    L1 --> N[Keyword Retrieval]
    M --> O[Vector Similarity / ANN Retrieval]

    N --> P[Hybrid Retrieval]
    O --> P

    P --> Q[Reranker]
    Q --> R[Top Relevant Chunks]
    R --> S[LLM / Answer Generation]
    S --> T[Grounded Answer]
```

------------------------------------------------------------------------

# 5. How Chunking Happens in This Exact Use Case

Suppose the NDA contains:

## Section 4.1

Definition of Confidential Information.

## Section 4.2

> Recipient shall use Confidential Information solely to evaluate the
> proposed partnership and for no other purpose, including training,
> fine-tuning, or improving any machine learning or AI model.

## Section 4.3

Other confidentiality obligations.

A good contract-aware chunker should ideally preserve the section
boundary.

### Bad chunking

``` text
Chunk 17:
"...evaluate the proposed partnership and for no other purpose, including"

Chunk 18:
"training, fine-tuning, or improving any machine learning or AI model..."
```

### Why is this bad?

The meaning of the prohibition is split across two chunks.

If only Chunk 17 is retrieved, the LLM may not know what activity is
prohibited.

------------------------------------------------------------------------

### Better chunking

``` text
Chunk ID: NDA_4.2

Section: 4.2

Text:
Recipient shall use Confidential Information solely to evaluate
the proposed partnership and for no other purpose, including
training, fine-tuning, or improving any machine learning or AI model.

Metadata:
document_type = NDA
section = 4.2
topic = Confidential Information
```

Now the complete business/legal meaning stays together.

------------------------------------------------------------------------

# 6. How Embedding Happens for the Chunk

## Step-by-step

### Contract chunk

``` text
Recipient shall use Confidential Information solely to evaluate
the proposed partnership and for no other purpose, including
training, fine-tuning, or improving any machine learning or AI model.
```

↓

### Tokenization

The embedding model converts text into tokens.

↓

### Transformer processing

The model considers the relationship between words such as:

-   use
-   confidential information
-   training
-   fine-tuning
-   machine learning
-   AI model

↓

### Pooling

Token-level representations are combined into one sentence/chunk
representation.

↓

### Dense embedding

Conceptually:

``` text
[0.14, -0.32, 0.78, 0.05, 0.61, ...]
```

↓

### Store in vector database

``` text
{
  chunk_id: "NDA_4.2",
  text: "Recipient shall use Confidential Information...",
  vector: [0.14, -0.32, 0.78, ...],
  metadata: {
    section: "4.2",
    document: "NDA"
  }
}
```

------------------------------------------------------------------------

# 7. What happens when the user asks the question?

## User question

> Does this NDA let the vendor use our data to train their AI models?

## Query flow

``` mermaid
flowchart LR
    A[User Question] --> B[Create Query Embedding]
    B --> C[Query Vector]

    C --> D[Compare with Stored Chunk Vectors]
    D --> E[Section 4.2 is Most Similar]

    F[BM25 Keyword Search] --> G[AI / train / vendor / data]
    G --> H[Section 4.2]

    E --> I[Combine Results]
    H --> I

    I --> J[Top Context to LLM]
    J --> K[Generate Direct Grounded Answer]
    K --> L["No – the NDA prohibits use of Confidential Information for training, fine-tuning, or improving AI/ML models."]
```

------------------------------------------------------------------------

# 8. The Key Concept to Remember

The complete RAG flow is:

``` text
DOCUMENT
   ↓
PARSE
   ↓
CHUNK
   ↓
ADD METADATA
   ↓
CREATE EMBEDDINGS
   ↓
STORE IN VECTOR DB / ANN INDEX
   ↓
USER QUERY
   ↓
CREATE QUERY EMBEDDING
   ↓
SIMILARITY SEARCH
   ↓
OPTIONAL BM25 SEARCH
   ↓
HYBRID RETRIEVAL
   ↓
RERANK
   ↓
TOP RELEVANT CHUNKS
   ↓
LLM
   ↓
FINAL GROUNDED ANSWER
```

## One-line interview/training explanation

> **In the Contract Copilot use case, the NDA is first split into
> meaningful section-aware chunks. Each chunk is converted into a dense
> embedding and stored with metadata. When the user asks a question, the
> query is also embedded, and semantic similarity plus optional BM25
> retrieval identifies the most relevant contract clauses. Those clauses
> are passed to the LLM so it can generate a grounded answer.**

------------------------------------------------------------------------

# Final takeaway

For this homework scenario:

-   **Chunking quality matters** because splitting a legal clause
    incorrectly can destroy its meaning.
-   **Embeddings capture semantic meaning** and help retrieve
    paraphrased questions.
-   **BM25 captures lexical matches** and is useful for exact legal
    terminology.
-   **Hybrid retrieval** can combine both strengths.
-   **High Context Precision/Recall does not guarantee a good final
    answer.**
-   In this example, the main failure is that the generator ignored or
    underused the most relevant retrieved evidence.
-   Therefore, **Answer Relevance should be low**, while **Faithfulness
    alone may not fully expose the failure**.
