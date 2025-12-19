# Movie Plot RAG System

A RAG (Retrieval-Augmented Generation) system that answers questions about movies by searching through plot summaries from Wikipedia.

## What This Does

Ask a question about movies (like "Which movie has an AI named HAL 9000?") and the system:
1. Searches through 400 movie plots to find relevant ones
2. Sends the relevant plots to Claude 
3. Gets back an answer with sources cited

## How It Works

```
Question → Convert to Vector → Search Similar Plots → Send to LLM → Get Answer
```

The key parts:
- **Sentence-BERT**: Converts text into numbers (embeddings) that capture meaning
- **FAISS**: Fast similarity search to find relevant movie plots
- **Claude API**: Generates the actual answer based on retrieved plots
- **Structured Output**: Returns JSON with the answer, which movies were used, and why

## Setup

### What You Need
- Python 3.8+
- Anthropic API key (free tier works)
- The Wikipedia Movie Plots dataset from Kaggle

### Running It

**Option 1: Google Colab (Easier)**

1. Open `movie_rag_system.ipynb` in Colab
2. Download the dataset from [Kaggle](https://www.kaggle.com/datasets/jrobischon/wikipedia-movie-plots)
3. Upload the CSV to Colab
4. Run all cells and enter your API key when asked

**Option 2: Locally**

```bash
pip install -r requirements.txt
# Download dataset from Kaggle and place in project folder
jupyter notebook movie_rag_system.ipynb
```

### If You Get a CSV Error

The dataset has some formatting issues. Use the code in `fixed_csv_loading.py` - it handles the problematic rows automatically.

## Example Usage

```python
result = rag_query("Which movie features an AI system called HAL 9000?")
print(result)
```

Output:
```json
{
  "answer": "The movie '2001: A Space Odyssey' features HAL 9000...",
  "contexts": ["2001: A Space Odyssey: ...HAL 9000 computer becomes antagonistic..."],
  "reasoning": "Found '2001: A Space Odyssey' which mentions HAL 9000 in the plot."
}
```

## Implementation Details

### Chunking Strategy
Long movie plots get split into ~300-word chunks with 50-word overlap. This helps with retrieval accuracy - you get the exact relevant part instead of a huge wall of text.

### Why These Tools?
- **Sentence-BERT**: Free, runs locally, good enough for this use case
- **FAISS**: Super fast for searching through vectors (< 10ms per query)
- **Claude**: Good at following instructions and returning structured JSON

### Adjusting Settings

Want to use more movies? Change this line in the notebook:
```python
sample_size = min(400, len(df))  # Try 200 or 1000
```

Want more context per query? Adjust k:
```python
result = rag_query(question, k=5)  # Default is 3
```

## Common Issues

**CSV won't load**  
→ Use the code from `fixed_csv_loading.py` - some plots have weird formatting

**Out of memory**  
→ Reduce sample_size to 200 movies

**API errors**  
→ Check your Anthropic API key is valid and has credits

## Project Structure

```
movie-rag-system/
├── movie_rag_system.ipynb      # Main notebook
├── fixed_csv_loading.py         # Fixes CSV parsing issues
├── requirements.txt             # Dependencies
├── README.md                    
├── KAGGLE_SETUP_GUIDE.md       # Dataset instructions
└── examples/
    └── example_outputs.json     # Sample results
```

## Notes

- The dataset is about 23MB, so I didn't include it in the repo. Download it from Kaggle.
- I sampled 400 movies to keep it manageable, but you can adjust this
- If you don't have Claude API access, you could swap in OpenAI's API or use a local model

## What I Learned

Building this helped me understand:
- How RAG actually works in practice (not just theory)
- Why chunking matters for retrieval quality  
- The trade-off between retrieval speed and accuracy
- How to handle messy real-world data (the CSV parsing was annoying!)

## License

MIT - feel free to use this however you want.
