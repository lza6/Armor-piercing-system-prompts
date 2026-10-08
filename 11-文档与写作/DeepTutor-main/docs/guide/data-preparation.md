# Data Preparation

DeepTutor provides demo knowledge bases and sample questions to help you get started quickly.

## Demo Knowledge Bases

We provide two pre-built knowledge bases on [Google Drive](https://drive.google.com/drive/folders/1iWwfZXiTuQKQqUYb5fGDZjLCeTUP6DA6?usp=sharing):

### 1. Research Papers Collection

<div class="info-card">
  <div class="info-header">
    <span class="info-icon">📄</span>
    <span class="info-title">5 Research Papers (20-50 pages each)</span>
  </div>
  <div class="info-content">
    <p>A curated collection of cutting-edge research papers from our lab, covering RAG and Agent fields.</p>
    <p><strong>Included Papers:</strong></p>
    <ul>
      <li><a href="https://github.com/HKUDS/AI-Researcher">AI-Researcher</a> - Automated research paper generation</li>
      <li><a href="https://github.com/HKUDS/AutoAgent">AutoAgent</a> - Autonomous agent framework</li>
      <li><a href="https://github.com/HKUDS/AutoAgent">AutoAgent</a> - Autonomous agent framework</li>
      <li><a href="https://github.com/HKUDS/DeepCode">DeepCode</a> - AI code assistant</li>
      <li><a href="https://github.com/HKUDS/VideoRAG">VideoRAG</a> - Video understanding with RAG</li>
    </ul>
    <p><strong>Best for:</strong> Research scenarios, broad knowledge coverage</p>
  </div>
</div>

### 2. Data Science Textbook

<div class="info-card">
  <div class="info-header">
    <span class="info-icon">📚</span>
    <span class="info-title">8 Chapters, 296 Pages</span>
  </div>
  <div class="info-content">
    <p>A comprehensive deep learning textbook from UC Berkeley.</p>
    <p><strong>Source:</strong> <a href="https://ma-lab-berkeley.github.io/deep-representation-learning-book/">Deep Representation Learning Book</a></p>
    <p><strong>Topics Covered:</strong></p>
    <ul>
      <li>Neural Network Fundamentals</li>
      <li>Representation Learning</li>
      <li>Deep Learning Architectures</li>
      <li>Advanced Topics</li>
    </ul>
    <p><strong>Best for:</strong> Learning scenarios, deep knowledge depth</p>
  </div>
</div>

## Download & Setup

### Step 1: Download

Visit our [Google Drive folder](https://drive.google.com/drive/folders/1iWwfZXiTuQKQqUYb5fGDZjLCeTUP6DA6?usp=sharing) and download:

- `knowledge_bases.zip` - Pre-built knowledge bases with embeddings
- `questions.zip` - Sample questions and usage examples (optional)

### Step 2: Extract

Extract the downloaded files into the `data/` directory:

```
DeepTutor/
├── data/
│   └── knowledge_bases/
│       ├── research_papers/      # Research papers KB
│       ├── data_science_book/    # Textbook KB
│       └── kb_config.json        # Knowledge base config
└── user/                         # User data (auto-created)
```

### Step 3: Verify

After extracting, your knowledge bases will be automatically available when you start DeepTutor.

::: warning Embedding Compatibility
Our demo knowledge bases use `text-embedding-3-large` with `dimensions = 3072`.

If your embedding model has different dimensions, you'll need to create your own knowledge base instead.
:::

## Creating Custom Knowledge Bases

### Supported File Formats

| Format | Extension | Notes |
|:-------|:----------|:------|
| PDF | `.pdf` | Supports text extraction and layout analysis |
| Text | `.txt` | Plain text files |
| Markdown | `.md` | Markdown with formatting support |

### Via Web Interface

1. Navigate to `http://localhost:3782/knowledge`
2. Click **"New Knowledge Base"**
3. Enter a unique name for your knowledge base
4. Upload your documents (single or batch upload)
5. Wait for processing to complete

::: tip Processing Time
- Small documents (< 10 pages): ~1 minute
- Medium documents (10-100 pages): ~5-10 minutes
- Large documents (100+ pages): May take longer
:::

### Via Command Line

```bash
# Initialize a new knowledge base with documents
deeptutor kb create <kb_name> --doc <pdf_path>

# Add documents to existing knowledge base
deeptutor kb add <kb_name> --doc <new_document.pdf>
```

## Data Storage Structure

All user data is stored in the `data/` directory:

```
data/
├── knowledge_bases/              # Knowledge base storage
│   ├── <kb_name>/
│   │   ├── documents/            # Original documents
│   │   ├── chunks/               # Chunked content
│   │   ├── embeddings/           # Vector embeddings
│   │   └── graph/                # Knowledge graph data
└── user/                         # User activity data
    ├── solve/                    # Problem solving results
    ├── question/                 # Generated questions
    ├── research/                 # Research reports
    ├── notebook/                 # Notebook records
    └── logs/                     # System logs
```

---

**Next Step:** [Local Installation →](/guide/local-start)

<style>
.info-card {
  background: var(--vp-c-bg-soft);
  border: 1px solid var(--vp-c-border);
  border-radius: 12px;
  padding: 20px;
  margin: 16px 0;
}

.info-header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 16px;
}

.info-icon {
  font-size: 1.5rem;
}

.info-title {
  font-size: 1.1rem;
  font-weight: 600;
  color: var(--vp-c-text-1);
}

.info-content {
  color: var(--vp-c-text-2);
  line-height: 1.7;
}

.info-content ul {
  margin: 12px 0;
  padding-left: 20px;
}

.info-content li {
  margin: 6px 0;
}

.info-content a {
  color: var(--vp-c-brand-1);
}
</style>
