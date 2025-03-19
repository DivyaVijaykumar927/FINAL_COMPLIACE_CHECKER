# Pgvectorscale Documentation

## Regulatory Compliance System Overview

### System Workflow:
1. **File Upload:** Users upload PDF documents.
2. **Text Extraction:** Extract text using PyMuPDF, pdfplumber, or Tika.
3. **Embedding Generation:** Convert extracted text into vector embeddings using OpenAI's embedding model.
4. **Vector Storage:** Store embeddings in pgvectorscale (PostgreSQL with pgvector).
5. **Compliance Matching:** Compare embeddings against predefined compliance policies stored in the database.
6. **Similarity Search:** Use cosine similarity to identify non-compliant sections.
7. **Report Generation:** Create structured reports highlighting:
   - Compliance adherence score
   - Sections requiring review
   - Suggested improvements

### Tech Stack:
- **Backend:** Python (FastAPI/Django)
- **Database:** PostgreSQL with pgvectorscale
- **PDF Processing:** PyMuPDF, pdfplumber
- **Vector Search:** OpenAI embeddings + pgvectorscale
- **Deployment:** Docker

## PostgreSQL as a Vector Database with Timescale

### Advantages of Using PostgreSQL with pgvectorscale
1. **Robust Open-Source Database:** Rich ecosystem of tools, drivers, and community support.
2. **Unified Data Management:** Store relational and vector data within a single database.
3. **Enhanced Performance:** Faster search capabilities, higher recall, and efficient time-based filtering.
4. **Advanced Indexing Techniques:** Inspired by DiskANN, providing efficient Approximate Nearest Neighbor (ANN) searches.

### Prerequisites
- Docker
- Python 3.7+
- OpenAI API key
- PostgreSQL GUI client

### Steps
#### 1. Set up Docker Environment
Create a `docker-compose.yml` file:

```yaml
services:
  timescaledb:
    image: timescale/timescaledb-ha:pg16
    container_name: timescaledb
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - "5432:5432"
    volumes:
      - timescaledb_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  timescaledb_data:
```
Run the container:
```sh
docker compose up -d
```

#### 2. Connect to Database
- Host: `localhost`
- Port: `5432`
- User: `postgres`
- Password: `password`
- Database: `postgres`

#### 3. Insert Document Chunks as Vectors
Use `insert_vectors.py` to generate and store embeddings using OpenAI's `text-embedding-3-small` model.

#### 4. Perform Similarity Search
Use `similarity_search.py` for querying relevant sections.

## Usage
1. Rename `example.env` to `.env` and add OpenAI API key.
2. Run Docker container.
3. Install dependencies:
```sh
pip install -r requirements.txt
```
4. Insert document vectors:
```sh
python insert_vectors.py
```
5. Perform similarity searches:
```sh
python similarity_search.py
```

## ANN Search Indexes for Faster Queries
Timescale Vector supports the following indexing options:
- **timescale_vector_index (default):** DiskANN-inspired graph index.
- **pgvector's HNSW:** Hierarchical Navigable Small World graph index.
- **pgvector's IVFFLAT:** Inverted file index.

For optimal performance, create an index on the embedding column.

```sql
CREATE INDEX ON documents USING timescale_vector_index(embedding);
```

## Cosine Similarity in Vector Search
### What is Cosine Similarity?
- Measures the cosine of the angle between two vectors.
- Used to evaluate similarity between embeddings.

#### Interpretation
- **1.0** → Most similar
- **0.0** → Unrelated
- **-1.0** → Most dissimilar

#### Cosine Distance in pgvector
- `<=>` operator computes **1 - cosine similarity**.
- **0.0** → Identical vectors (most similar).
- **1.0** → Orthogonal vectors (no similarity).
- **2.0** → Opposite vectors (most dissimilar).

#### Interpreting Results
- **Lower values = higher similarity.**
- **Closer to 0 → High similarity.**
- **Around 1 → Minimal similarity.**
- **Approaching 2 → Opposite meanings (rare).**

