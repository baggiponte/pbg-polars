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

# 🏗️ Polars fundamentals: contexts and expressions
Two key ingredients

Polars syntax is more expressive than `pandas`. It revolves around two fundamental concepts:

* **Contexts** where expressions are optimised.
* **Expressions**, which are building blocks that describe data transformations.

---

# 🏗️ Polars fundamentals: contexts and expressions
Contexts

Here are the four contexts. Contexts are called on the data, and can be chained:

```python
data.select(...)

data.with_columns(...)

data.groupby(...).agg(...)

data.filter(...)
```

---

# 🏗️ Polars fundamentals: contexts and expressions
Expressions

Can live outside of data but need a context to run an operation.


```python
pl.col("a").sum()
pl.col("a", "b").unique()
pl.all().mean()
pl.all().exclude("b").std()
pl.col(pl.Datetime)
```

The expression syntax is very broad - as much as `pandas`'.


---

## 😴 Lazy and eager mode
What does this mean?

Polars has two modes: *eager* and *lazy*.

Eager mode like pandas: every operation is performed sequentially, with limited optimisations.

Lazy mode is where Polars shines.

---

## 😴 Lazy and eager mode
Enabling lazy mode

Lazy mode can be entered by:

* Reading a dataset with `scan_*` functions instead of `read_*`.
* Calling `DataFrame.lazy()` on an eager DataFrame.

---

## 😴 Lazy and eager mode
Enabling lazy mode

Lazy mode operations are not evaluated by default, so you need to either:

* Call `LazyFrame.collect()` to run the operations.
* Call `LazyFrame.sink_*("path/to/destination)` to write the files to disk.


---

## ⚡Unique features
Stuff we can't dive into but we might use

* Great support for nested data types: operations benefit from the query engine!
* Window functions (we shall use those).
* Streaming engine: can work with data larger than memory.
  * Just call `collect(streaming=True)`. No changes in API.
* Can use [SQL](https://pola-rs.github.io/polars/user-guide/sql/intro/)!
* There is a [CLI](https://github.com/pola-rs/polars-cli?tab=readme-ov-file#polars-cli) too.

---

## 🥲 Weaknesses
Some things that need improving

* SQL support might be better in DuckDB, but Polars is catching up fast.
* Supports reading from remote storages. Still a new features so DuckDB can be faster, but is rapidly improving.
* JSON support is limited (DuckDB might be better).
* Frequent releases. `0.19` series has a few breaking changes.
  * Should be the latest minor before going stable.


---

## ⚠️ Dangerous live coding

Let's get our hands dirty!

---
layout: intro
---

# 🙏 Thank you!

Please share your feedback! My address is lucabaggi [at] duck.com

<div class="absolute right-5 top-5">
<img height="150" width="150"  src="/qr-linkedin.svg">
</div>
