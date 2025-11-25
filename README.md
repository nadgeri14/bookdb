# 📚 bookdb — End-to-End Indexing & Querying Pipeline

This repository provides a full workflow to:

- prepare and clean datasets  
- build large-scale token indices using the LLaMA tokenizer  
- serve an API for trace-based querying  
- generate enriched trace outputs  

Designed for HPC environments handling large datasets (tens–hundreds of GB).

---

## 🚀 1. Clone the Repository (Use Scratch Space)

Always work inside scratch on HPC clusters:

```bash
cd /gpfs/scratch/$USER
git clone https://github.com/<your-user>/bookdb.git
cd bookdb
```

---

## 🔐 2. Obtain HuggingFace Token (Required for LLaMA)

The indexer uses the LLaMA tokenizer, which requires authentication.

1. Get a token:  
   https://huggingface.co/settings/tokens  
   (Permission: **Read**)

2. Login on the server:

```bash
huggingface-cli login
```

Paste your token when prompted.

---

## 🧱 3. Build the Index

This is the most compute-heavy stage and can take **24–48 hours**.

Run:

```bash
python indexing_phil.py \
    --data_dir /gpfs/scratch/$USER/ATTEMPT_3/data \
    --save_dir /gpfs/scratch/$USER/ATTEMPT_3/index \
    --tokenizer llama \
    --cpus 64 \
    --mem 180 \
    --shards 8 \
    --workers 8 \
    --add_metadata \
    --ulimit 131072 \
    --token_dtype u32
```

### Notes

- **data_dir**  
  Folder containing raw documents.  
  The script expects certain default filenames — adjust the code to remove or update these checks.

- **save_dir**  
  Destination folder for index shards & metadata.  
  Output files are **large (GB-scale)**.

- **cpus / mem / shards / workers**  
  Tune based on your Slurm allocation.

### ⚠️ Important  
Run this inside:

- `tmux`  
- `screen`  
- or a Slurm job (`srun`, `sbatch`)  

so the process is not interrupted.

---

## 🖥️ 4. Start the API Server

After indexing completes:

```bash
python tracing_app.py
```

Run in background:

```bash
tmux new -s tracing
# or
screen -S tracing
```

This launches the HTTP API that `populate_trace.py` will use.

---

## 📝 5. Populate Traces (Generate Enriched Outputs)

With the API server running:

```bash
python populate_trace.py
```

This queries the server and writes final JSON trace outputs using your index.

---

## 📦 Recommended Directory Structure

```
/gpfs/scratch/$USER/ATTEMPT_3/
│
├── data/               # raw documents to be indexed
├── index/              # output index shards & metadata
└── logs/               # optional logs
```

---

## 💡 HPC Tips

- Use `tmux`, `screen`, or Slurm batch jobs for anything long-running  
- Set file descriptor limits high: `ulimit -n 131072`  
- Ensure scratch has **hundreds of GB free**  
- Monitor job memory with `htop`, `sstat`, or `sacct`

---

## ✔ Summary

1. Clone to scratch  
2. Login to HuggingFace  
3. Run `indexing_phil.py` to build the index  
4. Start API server with `tracing_app.py`  
5. Populate indexed traces via `populate_trace.py`
