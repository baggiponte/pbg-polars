---
theme: default
layout: cover
highlighter: shiki
colorSchema: light
favicon: favicon/url
title: Polars and functime
---

# 🐻‍❄️ Polars
The definitive DataFrame library

<div class="absolute bottom-10">

    👤 Luca Baggi
    💼 ML Engineer @xtream
    🐍 Organiser @Python Milano

</div>

<div class="absolute right-5 top-5">
<img height="150" width="150"  src="/qr-github.svg">
</div>


---

# 📍 Keynote Outline

## 🐻‍❄️ What is Polars?

## 🚀 What makes Polars so fast?

## 🏗️ Polars fundamentals: contexts and expressions

## 😴 Lazy and eager mode

## ⚡Unique features


---

# 🐻‍❄️ What is Polars?
In a nutshell

> Dataframes powered by a multithreaded, vectorized query engine, written in Rust

<v-clicks>

* A `DataFrame` frontend, i.e. work with a Python object and not a SQL table.
* Utilises all cores on your machine, efficiently (more on this later).
* Has a fast query engine that does a lot of optimisations under the hood.
* Bonus: in-process, like `sqlite`.
  * As easy to install as `pip install polars`. No other default dependencies.

</v-clicks>


---

# 🐻‍❄️ What is Polars?
In plain terms

<v-clicks>

* `pandas` but much faster, no indexes and a more expressive syntax.
  * Not a "drop-in" replacement like `modin` (for the three people out there using it).
* Like `duckdb` but with less SQL (though you can use SQL too!).

</v-clicks>


---

# 🐻‍❄️ What is Polars?
What it is not

Not a distributed system like `pyspark`: runs on one machine (for now).

Polars, however, can increase your ceiling so much that you will only need `pyspark` for truly big data, i.e. **complex transformations on more than 1TB**.

> Where the pipeline is simple Polars' streaming mode vastly outperforms Spark and is recommended for all dataset sizes. [Palantir Technologies](https://www.palantir.com/docs/foundry/announcements/#introducing-faster-transforms-for-small-to-medium-sized-datasets)


---

# 🚀 What makes Polars so fast?
The key ingredients

* Efficient data representation following Apache Arrow specification
  * Efficient I/O
* Work stealing, AKA efficient multithreading
* Lazy evaluation enables query optimisations


---

# 🚀 What makes Polars so fast?
Apache Arrow

A specification of a memory format, i.e. how to represent the data in memory, with implementations in a lot of programming languages (e.g. `pyarrow` in Python). Think of it as "`parquet` but in memory".

* Column based.
* Represents different data types efficiently.
* Can represent missing values (unlike `numpy`).
* Nested data types as well.


---

# 🚀 What makes Polars so fast?
Apache Arrow

The number of copies of the data between different programs that use Arrow is almost zero.

Polars, however, is based on a slightly different Rust implementation of Apache Arrow. Sometimes this means that conversion is not always zero copy.

Moving to or from 1D numerical `numpy` array is zero-copy.


---

# 🚀 What makes Polars so fast?
Effective multithreading

Pandas has vectorized operations thanks to numpy. However, it is not truly multithreaded. Polars is, and leverages _work stealing_ so that every thread is always working as much as the others.

Polars achieves this with a work scheduler that assigns tasks to each thread. Simply put, when a thread is idle, Polars takes away work assigned to other threads.


---

# 🚀 What makes Polars so fast?
Query optimisations

A list of operations is *compiled* into a *plan* that is optimised. For example:

1. Minimize materialisations
2. Predicate pushdown: filter data as early as possible
3. Common subplan elimination: compute duplicate operations only once

And many more. For a more thorough overview, watch [Polars' creator latest talk at PyData Amsterdam](https://www.youtube.com/watch?v=NJbBWDzZuWs)


---

# 🚀 What makes Polars so fast?
Query optimisations: an example

```python
import pandas as pd

data = (
  pd.read_parquet("path/to/file")
  .query("col_1 > 5")
  .head()
)
```

All data is read into memory, though only the first 5 row where `col_1` is greater than 5 are requested.


---

# 🚀 What makes Polars so fast?
Query optimisations: an example

```python
import polars as pl

data = (
  pl.scan_parquet("path/to/file")   # more on this later
  .filter(pl.col("col_1") > 5)
  .head()
  .collect()
)
```

This will actually just read from the parquet file the first 5 rows that meet the filter condition.


---





---

# Unique features

* `over()`
* streaming engine


---
layout: intro
---

# 🙏 Thank you!

Please share your feedback! My address is lucabaggi [at] duck.com

<div class="absolute right-5 top-5">
<img height="150" width="150"  src="/qr-linkedin.svg">
</div>
