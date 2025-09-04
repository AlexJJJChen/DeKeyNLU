# DeKeyNLU: Natural Language Understanding for NL2SQL

**DeKeyNLU** is a curated dataset designed to strengthen the *User Question Understanding* (UQU) stage in NL2SQL pipelines. 
It focuses on two core capabilities: **task decomposition** and **keyword extraction**. 
The corpus pairs natural-language questions with gold SQL, plus structured keyword annotations.


## Highlights

- **Scale:** 1500 examples.
- **Focus:** Structured annotations for question understanding (decomposition & keywording).
- **Ready-to-Use:** Each item includes the natural-language **question**, a serialized **keywords** field, and **golden_sql**.
- **Plug & Play:** Easily dropped into UQU → Retrieval → Generation/Refinement pipelines.


## Data Format

- The dataset is distributed as a single file: **`DeKeyNLU.json`** (a JSON array).  
- Each entry is a JSON object. **Common fields** present in ≥95% samples in your copy include:
  - `index`
  - `question`
  - `keywords`
  - `golden_sql`
  - `tester (three rounds followed by the name)`
  - `_sheet` 

**Observed top-level fields (sample counts):**
- `index`: 1500
- `question`: 1500
- `keywords`: 1500
- `golden_sql`: 1500
- `tester (three rounds followed by the name)`: 1500
- `_sheet`: 1500

### The `keywords` field
- The `keywords` field stores a **stringified JSON-like structure** (some entries may use single quotes).  
- You can parse it robustly with `ast.literal_eval` (fallback to `json.loads` with quote normalization).
- In your copy, the parsed `keywords` object contains keys such as: question, task, sub task, object, implementation.


## Quickstart

### Python: Load and parse

```python
import json, ast

def parse_keywords(s: str):
    try:
        return ast.literal_eval(s)
    except Exception:
        # Fallback if strict JSON is used or quotes differ
        return json.loads(s.replace("'", '"'))

with open("DeKeyNLU.json", "r", encoding="utf-8") as f:
    data = json.load(f)

record = data[0]
kw = parse_keywords(record.get("keywords", ""))
print("Question:", record.get("question"))
print("Parsed keywords:", kw)
print("Gold SQL:", record.get("golden_sql"))
```

### Create a UQU training corpus
```python
def build_uqu_example(item):
    kw = parse_keywords(item.get("keywords", "")) or {}
    if isinstance(kw, list):
        # in case of list-style keywords
        kw0 = kw[0] if kw else {}
    else:
        kw0 = kw
    return {
        "input": item.get("question", ""),
        "targets": {
            "main_tasks": kw0.get("task", []),
            "sub_tasks": kw0.get("sub task", []),
            "objects": kw0.get("object", []),
            "implementations": kw0.get("implementation", []),
        }
    }

uqu = [build_uqu_example(x) for x in data]
print("UQU examples:", len(uqu))
```

### Build NL2SQL pairs
```python
pairs = [{"question": x.get("question", ""), "sql": x.get("golden_sql", "")} for x in data]
```


## Recommended Splits

A common practice is to adopt **70% / 20% / 10%** for train / validation / test.
Please adapt the split to your experimental setup.


## Evaluation Notes

- **Pipeline view:** DeKeyNLU is most effective when used to train the **UQU** module of an NL2SQL system, 
  which then feeds **entity/table retrieval** and **SQL generation/refinement**.
- **Robustness:** Use execution accuracy on public dev sets (e.g., BIRD/Spider) to evaluate end-to-end gains.
- **Ablations:** Consider comparing with/without UQU fine-tuning and with alternative keyword/task schemas.


## Files in this Repository

- `DeKeyNLU.json` — the main dataset (JSON array of records).
- `README.md` — this document.

> If you also maintain auxiliary exports (e.g., JSONL), keep them under a `data/` subfolder and document their paths here.


## License

This dataset is intended for **research and non‑commercial** use.
Unless otherwise stated by the authors/maintainers, we recommend releasing under **CC BY‑NC 4.0**.
Please check (or provide) a `LICENSE` file in the repository root to make this explicit.



## Contributing

Contributions are welcome! Useful PRs include:
- **Cleaning/Parsing utilities** (e.g., canonicalizing the `keywords` structure).
- **Evaluation scripts** for execution accuracy and robustness tests.
- **Pipeline components** (UQU/Retrieval/Generation/Refinement) and reproducible baselines.


## Acknowledgements

Thanks to the annotation and review team for their careful work, and to the NL2SQL community for tools and benchmarks.
If you encounter issues or have feature requests, please open a GitHub Issue.
