---
title: "Full-Text Search and the Inverted Index"
tags: ["search", "database", "indexing"]
status: "draft"
prerequisites: []
---

# Full-Text Search and the Inverted Index

Full-Text Search (FTS) is the technology that enables finding and ranking search terms within a large body of text, known as a "corpus." Unlike simple string matching (like SQL's `LIKE` operator), FTS is designed to be fast, scalable, and intelligent, providing relevance-ranked results.

The core data structure that powers modern FTS is the **Inverted Index**.

## The Problem with Traditional Methods

In a traditional relational database, searching for a substring within a text field typically uses a `LIKE '%search_term%'` query. This approach is inefficient for two main reasons:

1.  **Full Scan Required:** It forces the database to perform a full scan of the text in every single row. This does not scale as the amount of data grows.
2.  **No Concept of Relevance:** It returns a simple match/no-match. It cannot determine which document is *more relevant* to the user's query.

## The Solution: The Inverted Index

An inverted index is a data structure that maps content, such as words or numbers, to its locations in a database file, or in a document or a set of documents. It is the most popular data structure used in document retrieval systems, used on a large scale by search engines.

Instead of mapping documents to the words they contain (a forward index), an inverted index maps words to the documents they appear in.

**Analogy:** Think of the index at the back of a book. Instead of reading the entire book to find where "database" is mentioned, you look up "database" in the index, and it gives you the exact page numbers (e.g., 42, 113, 256).

### How it Works: A Simple Example

Imagine we have three documents:

-   **Doc 1:** "The quick brown fox"
-   **Doc 2:** "A brown dog and a brown cat"
-   **Doc 3:** "The lazy dog"

The inverted index would look like this:

| Term    | Document List |
|---------|---------------|
| a       | {Doc 2}       |
| and     | {Doc 2}       |
| brown   | {Doc 1, Doc 2}|
| cat     | {Doc 2}       |
| dog     | {Doc 2, Doc 3}|
| fox     | {Doc 1}       |
| lazy    | {Doc 3}       |
| quick   | {Doc 1}       |
| the     | {Doc 1, Doc 3}|

When a user searches for "brown dog", the system:
1.  Looks up "brown" -> `{Doc 1, Doc 2}`
2.  Looks up "dog" -> `{Doc 2, Doc 3}`
3.  Finds the intersection -> `{Doc 2}`

This lookup process is incredibly fast, even with billions of documents.

## The Analysis Process

Before indexing, the raw text is processed in a pipeline called **Analysis**. This ensures that searches are effective. Key steps include:

1.  **Tokenization:** Breaking the text down into individual words or terms (tokens). "The quick brown fox" -> `The`, `quick`, `brown`, `fox`.
2.  **Normalization:** Converting tokens to a standard form. Most commonly, this is lowercasing. `The` -> `the`.
3.  **Stop Word Removal:** Removing common words that add little semantic value, like "a", "and", "the", "is".
4.  **Stemming/Lemmatization:** Reducing words to their root form. "running" -> "run" (stemming), or "ran" -> "run" (lemmatization). This allows a search for "run" to match documents containing "running" or "ran".

## Relevance Scoring

Modern search engines also calculate a **relevance score** to rank the results. A common algorithm is **TF-IDF (Term Frequency-Inverse Document Frequency)**.

-   **Term Frequency (TF):** How often does the term appear in *this* document? (More is better).
-   **Inverse Document Frequency (IDF):** How rare is the term across *all* documents? (Rarer is better).

A term like "database" is more significant than a term like "the" because it appears far less frequently across the entire corpus. Documents are ranked based on this combined score.

## Key Technologies

-   **Apache Lucene:** A high-performance, full-featured text search engine library written in Java. It is the underlying technology for the two systems below.
-   **Elasticsearch:** A distributed, RESTful search and analytics engine built on top of Lucene.
-   **OpenSearch / Apache Solr:** Other popular open-source search platforms also built on Lucene.
