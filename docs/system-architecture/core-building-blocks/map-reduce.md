# MapReduce

## Table of Contents
1. [What is MapReduce](#what-is-mapreduce)
2. [Problems It Solves](#problems-it-solves)
3. [Relationship to Horizontal Scaling and Data Processing](#relationship-to-horizontal-scaling-and-data-processing)
4. [Key Considerations Before Implementation](#key-considerations-before-implementation)
   - [Distributed File System](#1-distributed-file-system)
   - [No Data Movement](#2-no-data-movement)
   - [Key-Value Structure](#3-key-value-structure)
   - [Machine Failures](#4-machine-failures)
   - [Idempotency](#5-idempotency)
5. [Example: Count Unique Words Across Multiple Files](#example-count-unique-words-across-multiple-files)
   - [Problem Statement](#problem-statement)
   - [MapReduce Approach](#mapreduce-approach)
   - [Python Implementation](#python-implementation)
6. [Additional Problem Statements](#additional-problem-statements)
7. [Diagram: MapReduce Architecture](#diagram-mapreduce-architecture)

---

## What is MapReduce

- **Definition**: A distributed computing framework that processes large datasets by breaking them into smaller chunks, processing them in parallel across multiple machines, and then combining the results
- **Core Concept**: Inspired by functional programming's `map()` and `reduce()` functions
- **Two Main Phases**:
  - **Map Phase**: Transforms input data into intermediate key-value pairs
  - **Reduce Phase**: Aggregates intermediate values with the same key into final results
- **Execution Model**: Automatically handles parallelization, fault tolerance, and data distribution across a cluster
- **Use Case**: Batch processing of massive datasets where latency is not critical

<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 9.17.16 PM.png' width=700 />

---

## Problems It Solves

- **Data Volume Overload**: Processes terabytes/petabytes of data that cannot fit on a single machine
- **Processing Time**: Distributes computation across multiple machines to reduce total processing time
- **Fault Tolerance**: Automatically recovers from machine failures without losing progress
- **Complexity of Distributed Programming**: Abstracts away low-level distributed system details (networking, synchronization, fault handling)
- **Resource Utilization**: Leverages idle CPU and disk resources across a cluster
- **Scalability**: Linearly scales performance by adding more machines to the cluster

---

## Relationship to Horizontal Scaling and Data Processing

### Horizontal Scaling
- **MapReduce enables horizontal scaling** by distributing computation across multiple commodity machines
- Instead of upgrading a single powerful server (vertical scaling), MapReduce adds more machines to the cluster
- **Cost-effective**: Uses cheaper commodity hardware instead of expensive high-end servers
- **Fault tolerance**: Losing one machine doesn't stop the entire job; work is redistributed

### Data Processing
- **Batch Processing Model**: Optimized for processing entire datasets rather than individual requests
- **Data Locality**: Moves computation to where data resides, minimizing network traffic
- **Parallel Processing**: Processes multiple data chunks simultaneously across the cluster
- **Aggregation**: Combines results from distributed processing into meaningful insights

---

## Key Considerations Before Implementation
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 9.19.21 PM.png' width=600 />

### 1. **Distributed File System**
- **Requirement**: Data must be stored in a distributed file system (e.g., HDFS, S3)
- **Why**: Enables data locality—computation runs on nodes where data is stored
- **Benefit**: Reduces network bandwidth by avoiding data movement across the cluster
- **Consideration**: Replication factor ensures fault tolerance (typically 3 copies per block)

### 2. **No Data Movement**
- **Principle**: "Move computation to data, not data to computation"
- **Implementation**: Mappers run on nodes containing input data blocks
- **Benefit**: Minimizes network I/O, which is the bottleneck in distributed systems
- **Trade-off**: Requires careful data partitioning and placement strategy

### 3. **Key-Value Structure**
- **Input Format**: Data must be decomposable into key-value pairs
- **Map Output**: Produces intermediate key-value pairs
- **Reduce Input**: Groups all values for the same key
- **Consideration**: Choosing the right key is critical for load balancing and result correctness
- **Example**: For word count, key = word, value = count

### 4. **Machine Failures**
- **Assumption**: Failures are expected in large clusters
- **Handling**: MapReduce framework automatically re-executes failed tasks on other machines
- **Checkpointing**: Intermediate results are written to disk for recovery
- **Consideration**: Design jobs to be resilient; avoid dependencies between tasks
- **Implication**: Speculative execution may re-run slow tasks on multiple machines

### 5. **Idempotency**
- **Requirement**: Tasks must produce the same result when executed multiple times
- **Why**: Failed tasks are retried; speculative execution may run tasks in parallel
- **Implementation**: Avoid side effects; ensure deterministic computation
- **Consideration**: Reducers must be idempotent to handle duplicate intermediate results
- **Example**: Aggregation functions (sum, count, max) are idempotent; incrementing a counter is not

---

## Example: Count Unique Words Across Multiple Files
<img src='../../../Resources/19-core-building-blocks/Screenshot 2026-02-16 at 10.47.49 PM.png' width=750 />

### Problem Statement
Count the frequency of each unique word across 2 large text files using MapReduce.

**Input Files**:
- `file1.txt`: Contains text data
- `file2.txt`: Contains text data

**Expected Output**: A list of unique words with their total frequency across both files

### MapReduce Approach

```
Map Phase:
  Input: (filename, text_chunk)
  Process: Split text into words, emit (word, 1) for each word
  Output: [(word1, 1), (word1, 1), (word2, 1), ...]

Shuffle & Sort Phase (automatic):
  Group all values by key: {word1: [1, 1, 1], word2: [1, 1], ...}

Reduce Phase:
  Input: (word, [1, 1, 1, ...])
  Process: Sum all counts for the word
  Output: (word, total_count)
```

### Python Implementation

```python
from collections import defaultdict
from typing import List, Tuple, Dict

# Simulating MapReduce framework
class MapReduceWordCount:
    """
    Simple MapReduce implementation for word counting across multiple files.
    """
    
    def __init__(self, files: List[str]):
        """
        Initialize with list of file paths.
        
        Args:
            files: List of file paths to process
        """
        self.files = files
        self.intermediate_results = defaultdict(list)
    
    def map_function(self, filename: str, text: str) -> List[Tuple[str, int]]:
        """
        Map phase: Convert text into (word, 1) pairs.
        
        Args:
            filename: Name of the file being processed
            text: Text content from the file
            
        Returns:
            List of (word, count) tuples
        """
        # Split text into words and normalize
        words = text.lower().split()
        
        # Emit (word, 1) for each word
        mapped_pairs = []
        for word in words:
            # Remove punctuation (simplified)
            clean_word = ''.join(c for c in word if c.isalnum())
            if clean_word:
                mapped_pairs.append((clean_word, 1))
        
        return mapped_pairs
    
    def shuffle_and_sort(self, mapped_data: List[Tuple[str, int]]) -> Dict[str, List[int]]:
        """
        Shuffle and sort phase: Group values by key.
        
        Args:
            mapped_data: Output from all mappers
            
        Returns:
            Dictionary with words as keys and lists of counts as values
        """
        shuffled = defaultdict(list)
        for word, count in mapped_data:
            shuffled[word].append(count)
        return shuffled
    
    def reduce_function(self, word: str, counts: List[int]) -> Tuple[str, int]:
        """
        Reduce phase: Aggregate counts for each word.
        
        Args:
            word: The word (key)
            counts: List of counts from all mappers
            
        Returns:
            Tuple of (word, total_count)
        """
        total = sum(counts)
        return (word, total)
    
    def execute(self) -> Dict[str, int]:
        """
        Execute the complete MapReduce job.
        
        Returns:
            Dictionary with words and their total counts
        """
        # Map Phase: Process each file
        all_mapped_pairs = []
        for filename in self.files:
            try:
                with open(filename, 'r', encoding='utf-8') as f:
                    text = f.read()
                    mapped_pairs = self.map_function(filename, text)
                    all_mapped_pairs.extend(mapped_pairs)
            except FileNotFoundError:
                print(f"Warning: File {filename} not found")
                continue
        
        # Shuffle and Sort Phase
        shuffled_data = self.shuffle_and_sort(all_mapped_pairs)
        
        # Reduce Phase
        final_results = {}
        for word, counts in shuffled_data.items():
            word_result, total_count = self.reduce_function(word, counts)
            final_results[word_result] = total_count
        
        return final_results
    
    def get_top_words(self, results: Dict[str, int], top_n: int = 10) -> List[Tuple[str, int]]:
        """
        Get the top N most frequent words.
        
        Args:
            results: Output from execute()
            top_n: Number of top words to return
            
        Returns:
            List of (word, count) tuples sorted by count
        """
        return sorted(results.items(), key=lambda x: x[1], reverse=True)[:top_n]


# Example Usage
if __name__ == "__main__":
    # Create sample files for demonstration
    sample_files = ["file1.txt", "file2.txt"]
    
    # Create sample data
    with open("file1.txt", "w") as f:
        f.write("hello world hello python world data processing")
    
    with open("file2.txt", "w") as f:
        f.write("hello data science world python hello")
    
    # Run MapReduce job
    mr = MapReduceWordCount(sample_files)
    results = mr.execute()
    
    # Display results
    print("Word Frequency Count:")
    print("-" * 40)
    for word, count in sorted(results.items(), key=lambda x: x[1], reverse=True):
        print(f"{word:20} : {count:5}")
    
    # Display top 5 words
    print("\nTop 5 Words:")
    print("-" * 40)
    for word, count in mr.get_top_words(results, 5):
        print(f"{word:20} : {count:5}")
```

**Output**:
```
Word Frequency Count:
----------------------------------------
hello                :     3
world                :     3
python               :     2
data                 :     2
processing           :     1
science              :     1

Top 5 Words:
----------------------------------------
hello                :     3
world                :     3
python               :     2
data                 :     2
processing           :     1
```

---

## Additional Problem Statements

### 1. **Log Analysis**
- **Problem**: Analyze billions of log entries to find error patterns
- **Map**: Extract error type and timestamp from each log line
- **Reduce**: Count occurrences of each error type, calculate error rates
- **Benefit**: Identify critical issues affecting system reliability

### 2. **Inverted Index Creation**
- **Problem**: Build a search index for a massive document corpus
- **Map**: For each document, emit (word, document_id) pairs
- **Reduce**: Collect all document IDs for each word
- **Benefit**: Enable fast full-text search across billions of documents

### 3. **Data Deduplication**
- **Problem**: Remove duplicate records from a massive dataset
- **Map**: Emit (record_hash, record) for each record
- **Reduce**: Keep only one copy of each unique record
- **Benefit**: Reduce storage costs and improve data quality

### 4. **Graph Processing**
- **Problem**: Compute PageRank for billions of web pages
- **Map**: Distribute page rank to outgoing links
- **Reduce**: Aggregate incoming rank contributions for each page
- **Benefit**: Rank pages by importance for search results

### 5. **Time Series Aggregation**
- **Problem**: Aggregate metrics across millions of servers
- **Map**: Emit (metric_name, timestamp_bucket, value) from each server
- **Reduce**: Aggregate values for each metric and time bucket
- **Benefit**: Generate dashboards and alerts from massive metric streams

### 6. **Machine Learning Feature Extraction**
- **Problem**: Extract features from terabytes of raw data
- **Map**: Parse raw data and compute feature vectors
- **Reduce**: Aggregate statistics (mean, variance) for feature normalization
- **Benefit**: Prepare data for training ML models at scale

### 7. **Join Operations**
- **Problem**: Join two massive datasets that don't fit in memory
- **Map**: Emit (join_key, (table_id, record)) for both datasets
- **Reduce**: Combine records with matching keys from both tables
- **Benefit**: Perform complex data transformations on petabyte-scale data

---

## Diagram: MapReduce Architecture

```
┌─────────────────────────────────────────────────────────┐
│            INPUT DATA (Distributed File System)         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │  Block 1     │  │  Block 2     │  │  Block 3     │   │
│  │ (Node A)     │  │ (Node B)     │  │ (Node C)     │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
└─────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────┐
│                        MAP PHASE                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │  Mapper 1    │  │  Mapper 2    │  │  Mapper 3    │   │
│  │ (Node A)     │  │ (Node B)     │  │ (Node C)     │   │
│  │              │  │              │  │              │   │
│  │ Input: Block │  │ Input: Block │  │ Input: Block │   │
│  │ Output:      │  │ Output:      │  │ Output:      │   │
│  │ (k1, v1)     │  │ (k2, v2)     │  │ (k3, v3)     │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
└─────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────┐
│                  SHUFFLE & SORT PHASE                   │
│  ┌─────────────────────────────────────────────────┐    │
│  │ Group by Key:                                   │    │
│  │ k1 → [v1, v1, v1]                               │    │
│  │ k2 → [v2, v2]                                   │    │
│  │ k3 → [v3, v3, v3, v3]                           │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────┐
│                      REDUCE PHASE                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ Reducer 1    │  │ Reducer 2    │  │ Reducer 3    │    │
│  │              │  │              │  │              │    │
│  │ Input:       │  │ Input:       │  │ Input:       │    │
│  │ (k1, [v...]) │  │ (k2, [v...]) │  │ (k3, [v...]) │    │
│  │              │  │              │  │              │    │
│  │ Output:      │  │ Output:      │  │ Output:      │    │
│  │ (k1, result) │  │ (k2, result) │  │ (k3, result) │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└──────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────┐
│                    FINAL OUTPUT                          │
│  ┌───────────────────────────────────────────────────┐   │
│  │ (k1, final_result_1)                              │   │
│  │ (k2, final_result_2)                              │   │
│  │ (k3, final_result_3)                              │   │
│  └───────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

**Key Points**:
- **Data Locality**: Mappers run on nodes containing input blocks
- **Parallelism**: Multiple mappers and reducers execute simultaneously
- **Fault Tolerance**: Failed tasks are automatically re-executed
- **Scalability**: Add more nodes to process larger datasets
