---
theme: default
layout: cover
highlighter: shiki
colorSchema: light
favicon: favicon/url
title: "Polars: the definitive DataFrame library"
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

<v-clicks>


## 🐻‍❄️ What is Polars?

## 🚀 What makes Polars so fast?

## 🏗️ Polars fundamentals: contexts and expressions

## 😴 Lazy and eager mode

## ⚡Unique features

## 🥲 Weaknesses


</v-clicks>


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

<v-clicks>

* Efficient data representation following Apache Arrow specification
  * Efficient I/O
* Work stealing, AKA efficient multithreading
* Lazy evaluation enables query optimisations

</v-clicks>


---

# 🚀 What makes Polars so fast?
Apache Arrow

A specification of a memory format, i.e. how to represent the data in memory, with implementations in a lot of programming languages (e.g. `pyarrow` in Python). Think of it as "`parquet` but in memory".

<v-clicks>

* Column based.
* Represents different data types efficiently.
* Can represent missing values (unlike `numpy`).
* Nested data types as well.

</v-clicks>


---

# 🚀 What makes Polars so fast?
Apache Arrow

<v-clicks>

The number of copies of the data between different programs that use Arrow is almost zero.

Polars, however, is based on a slightly different Rust implementation of Apache Arrow. Sometimes this means that conversion is not always zero copy.

Moving to or from 1D numerical `numpy` array is zero-copy.

</v-clicks>


---

# 🚀 What makes Polars so fast?
Effective multithreading

<v-clicks>

Pandas has vectorized operations thanks to numpy. However, it is not truly multithreaded.

Polars is, and leverages _work stealing_ so that every thread is always working as much as the others.

Polars achieves this with a work scheduler that assigns tasks to each thread. Simply put, when a thread is idle, Polars takes away work assigned to other threads.

</v-clicks>


---

# 🚀 What makes Polars so fast?
Query optimisations

<v-click>

A list of operations is *compiled* into a *plan* that is optimised. For example:

</v-click>

<v-clicks>

1. Minimize materialisations
2. Predicate pushdown: filter data as early as possible
3. Common subplan elimination: compute duplicate operations only once

</v-clicks>

<v-click>

And many more. For a more thorough overview, watch [Polars' creator latest talk at PyData Amsterdam](https://www.youtube.com/watch?v=NJbBWDzZuWs)

</v-click>

---

# 🚀 What makes Polars so fast?
Query optimisations: an example

```python{all|4|5|6|all}
import pandas as pd

data = (
  pd.read_parquet("path/to/file")
  .query("col_1 > 5")
  .head()
)
```

<v-click>

All data is read into memory, though only the first 5 row where `col_1` is greater than 5 are requested.

</v-click>


---

# 🚀 What makes Polars so fast?
Query optimisations: an example

```python{all|4|5|6|7|all}
import polars as pl

data = (
  pl.scan_parquet("path/to/file")   # more on this later
  .filter(pl.col("col_1") > 5)
  .head()
  .collect()
)
```

<v-click>

This will actually just read from the parquet file the first 5 rows that meet the filter condition.

</v-click>


---

# 🏗️ Polars fundamentals: contexts and expressions
Two key ingredients

Polars syntax is more expressive than `pandas`. It revolves around two fundamental concepts:

<v-clicks>

* **Contexts** where expressions are optimised.
* **Expressions**, which are building blocks that describe data transformations.

</v-clicks>


---

# 🏗️ Polars fundamentals: contexts and expressions
Contexts

Here are the four contexts. Contexts are called on the data, and can be chained:

```python{none|1|3|5|7}
data.select(...)

data.with_columns(...)

data.groupby(...).agg(...)

data.filter(...)
```

<v-click>

But there are also verbs like `join`, `sort`...

</v-click>


---

# 🏗️ Polars fundamentals: contexts and expressions
Expressions

Can live outside of data but need a context to run an operation.


```python{none|1|2|3|4|5}
pl.col("a").sum()
pl.col("a", "b").unique()
pl.col(pl.Datetime)
pl.all().mean()
pl.all().exclude("b").std()
```

<v-clicks>

The [expression](https://pola-rs.github.io/polars/py-polars/html/reference/expressions/index.html) syntax is very broad - as much as `pandas`'.

There is also a broad list of [`selectors`](https://pola-rs.github.io/polars/py-polars/html/reference/selectors.html) which can be used to create algebraic combinations of columns.

</v-clicks>


---

# 😴 Lazy and eager mode
What does this mean?

<v-clicks>

Polars has two modes: *eager* and *lazy*.

Eager mode like pandas: every operation is performed sequentially, with limited optimisations.

Lazy mode is where Polars achieves maximum performance.

We can seamlessly move between those.

</v-clicks>


---

# 😴 Lazy and eager mode
Enabling lazy mode

Lazy mode can be entered by:

<v-clicks>

* Reading a dataset with `scan_*` functions instead of `read_*`.
* Calling `DataFrame.lazy()` on an eager DataFrame.

</v-clicks>


---

# 😴 Lazy and eager mode
Enabling lazy mode

Lazy mode operations are not evaluated by default, so you need to either:

<v-clicks>

* Call `LazyFrame.collect()` to run the operations.
* Call `LazyFrame.sink_*("path/to/destination)` to write the files to disk.

</v-clicks>


---

# ⚡Unique features
Stuff we will go over

<v-clicks>

* Great support for nested data types: operations benefit from the query engine!
* Window functions (we shall use those).
* Streaming engine: can work with data larger than memory.
  * Just call `collect(streaming=True)`. No changes in API.

</v-clicks>


---

# ⚡Unique features
Stuff we can't go over right now

<v-clicks>

* Can use [SQL](https://pola-rs.github.io/polars/user-guide/sql/intro/)!
* There is a [CLI](https://github.com/pola-rs/polars-cli?tab=readme-ov-file#polars-cli) too.
* Polars plugins! Can write your own Rust extensions and hook them inside Polars query engine for incredible speed.
  * More on this next week. No Rust, I promise.

</v-clicks>


---

# 🥲 Weaknesses
Some things that need improving

<v-clicks>

* SQL support might be better in DuckDB, but Polars is catching up fast.
* Supports reading from remote storages. Still a new features so DuckDB can be faster, but is rapidly improving.
* JSON support is limited (DuckDB might be better).
* Frequent releases. `0.19` series has a few breaking changes.
  * Should be the latest minor before going stable.

</v-clicks>


---
layout: intro
---

# 🚧 Dangerous live coding

Let's get our hands dirty!


---
layout: intro
---

# 🙏 Thank you!

Please share your feedback! My address is lucabaggi [at] duck.com

<div class="absolute right-5 top-5">
<img height="150" width="150"  src="/qr-linkedin.svg">
</div>
