# Mindmaps Directory

This directory contains tab-separated plain-text mindmaps generated from reference materials.

## Format

Mindmaps use a simple tab-separated hierarchical format:

```
Article Title	https://original-source-url.com
	Main Topic 1
		Subtopic 1.1
			Detail 1.1.1
			Detail 1.1.2
		Subtopic 1.2
	Main Topic 2
		Subtopic 2.1
```

## Structure Rules

- **First line:** Always contains the title and original source URL/path separated by a tab
- **Indentation:** Each level uses ONE tab character (no spaces)
  - Root (article) = 0 tabs
  - Main topics = 1 tab
  - Subtopics = 2 tabs
  - Details = 3 tabs
- **No formatting:** Plain text only (no bullets, markdown, or special characters)
- **Concise nodes:** Each node should be 5-10 words maximum
- **Hierarchy:** Organized from general concepts to specific details

## Usage

These mindmaps are automatically generated when using:
```bash
/rgn.add-reference <source> --mindmap
```

## Import/Export

The tab-separated format can be:
- Imported into mindmapping tools that support indented text
- Converted to various formats (Markdown, Freemind, etc.)
- Parsed programmatically for knowledge graphs
- Viewed directly as plain text

## Example

```
Understanding RAG Systems	https://arxiv.org/abs/2301.xxxxx
	Core Components
		Retrieval Module
			Vector Database
			Similarity Search
		Generation Module
			Language Model
			Context Integration
	Implementation Strategies
		Chunking Approaches
			Fixed-size Windows
			Semantic Boundaries
		Embedding Models
			Dense Retrievers
			Sparse Retrievers
	Evaluation Metrics
		Retrieval Quality
		Generation Accuracy
```
