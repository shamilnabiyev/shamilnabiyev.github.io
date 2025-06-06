---
author: "Shamil Nabiyev"
title: "Pandas cheat sheets"
date: "2020-11-20"
tags: [
    "python",
    "pandas"
]
draft: false
---

Pandas cheat sheets

- Find duplicates by column value: `df[df.duplicated(['col_name'])]`
- Find duplicates by row: `df[df.duplicated()]`
- Select rows from a DataFrame based on column values `df.loc[df['column_name'] == some_value]`
- Rename df column `df.rename(columns={'gdp':'log(gdp)'}, inplace=True)`
- Datatype of the columns `df.dtypes`

