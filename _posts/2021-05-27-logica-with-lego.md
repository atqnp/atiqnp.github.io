---
layout: post
title: Logica with LEGO
subtitle: Exploring Logica using LEGO dataset
tags: [logica, lego, sql, bigquery]
---

It has been a few weeks since my last post. I am still taking some time to get the right mood and consistency for writing 😁. Here, I will explain a little bit about Logica, a language that I had been learning for the past weeks. Since the post is too long, I had to break it down into parts. So, here goes part 1. I will update this space when the next part has been published. 

## Introduction

Recently, Google introduce a new logical-based language in order to make SQL more accessible. It is called Logica.
If you knew SQL then you might be familiar with all the syntax but Logica just made easier. You can read more about it [here](https://opensource.google/projects/logica) and [here](https://opensource.googleblog.com/2021/04/logica-organizing-your-data-queries.html)

At the time of this writing, it is only available for BigQuery processing engine but there are also others in the work.
In this post, I will be going to share a little bit about what I did using Logica while learning them so you could also consider this as a kind of tutorial 😜.
Because I love LEGO, and it became a new hobby of mine (actually I loved them since my childhood but they were very expensive for me 😆), so I combined Logica with LEGO and try to explore the data with it.

Before we go further, a few points to note. The dataset was taken from rebrickable. If you want, you can get them [here](https://rebrickable.com/downloads/). The dataset there is always renewed daily. The one that I used was on/until 26th of May 2021. So it may differs based on the date that you download the dataset. Another point to be taken; the work was done using [Google Colaboratory](https://colab.research.google.com/). You can try and run it yourself. I will try to attach a link to my notebook at the end of this post. Logica can also be run locally and its scripts can be written in its own file ending with the `.l` extension. However, I will not get into that here. If you want to learn more about it you can check it [here](https://logica.dev/)
Another thing to note, before I forgot, a Google Cloud Platform (GCP) account is required to run the program. You can try and make a free account [here](https://cloud.google.com/)

UPDATE: If you don't want to make a GCP account, Logica can use the SQLite engine.

First thing first, package including Logica is required to be installed. You can run the using the pip command.

```python
!pip install logica
``` 

Next, import all of the required package. If you have made your project in GCP you can put it here.

```python
from google.colab import auth
from logica import colab_logica
auth.authenticate_user()
colab_logica.SetProject(YOURPROJECTNAME)
```

UPDATE: If you didn't create a GCP account and wanted to use the SQLite engine, you can add this line. There are also a few others engine available but I have yet to explore them.

```python
@Engine("sqlite");
```

In order to run Logica in the notebook the magic command `%%` is required. So in the notebook we will write it as follows.

```python
%%logica
```

Basically, in Logica, a program is run as a predicate. We need to define the predicate that we wanted to output. For easier understanding, for those who are familiar with Pandas, the predicate is almost the same as a Pandas DataFrame. I will show you an example. Lets say we wanted to name a predicate as `Intro`. In a way, it is similar with Pandas `Intro = pd.DataFrame`. We will try to write the full code below. In Logica, the predicate that wanted to be output need to be defined earlier with the magic command. For the above example:

```python
%%logica Intro

Intro(greeting: "Hello world!")
```

So, this will create a predicate named `Intro` with a column named `greeting` and `Hello world!` as one of its record. To recreate it in quite the same pattern in Pandas:

```python
import pandas as pd

data = {'greeting': ['Hello world!']}
Intro = pd.DataFrame(data)
```

Similar with Pandas, with can recall the predicate later and even use it as a DataFrame in Python. Without further ado, let us dive in. I will try to follow the same concept as an SQL tutorial using Logica.

## Logica - SQL syntax

### Querying, filtering, and sorting data

#### `SELECT` statement
Firstly, we need to connect the database and define the predicate. I already uploaded the data to my BigQuery.

```python
%%logica Sets

Sets(set_num:) :- `lego_data.sets`(set_num:);
```

So, what this does is the database in my BigQuery is extracted to the `Sets` predicate. In this case only the columns `set_num` is extracted. In order to extract any/certain column, `<column name>:` can be used. Multiple column extraction can be divided using a comma, such as:

```python
%%logica Sets

Sets(set_num:, name:, year:) :- 
  `lego_data.sets`(set_num:, name:, year:);
```
in SQL this is similar with

```sql
SELECT
    set_num, 
    name, 
    year 
FROM `lego_data.sets`
```

This will extract the column `set_num`, `name` and `year` from the table. Another way at it is by:

```python
%%logica Sets

Sets(..t) :- `lego_data.sets`(..t);
```
Using this one will extract the whole table similar to:

```sql
SELECT * FROM `lego_data.sets`
```
This will prove useful when there is a large number of columns in a table. In Logica, the column names' are changed with `..<variables>`
#### `AS` clause
A column name can be renamed however we wanted. In Logica, we can do it as follows.

```python
%%logica Sets

Sets(set_num:, name:, year:, pcs:) :- 
  `lego_data.sets`(set_num:, name:, year:, num_parts:pcs);
```
This will change the column name `num_parts` as `pcs`. This change will make us more familiar with the LEGO data since they used the number of parts as pieces.
Similarly, in SQL:

```sql
SELECT 
    set_num,
    name,
    year,
    num_parts AS pcs 
FROM `lego_data.sets`
```

In Logica, the renaming of a column is easier, more explicit and clearer thus making it easier to do the `JOIN` operation. We will cover more on these later.

#### `WHERE` clause

For extracting only a certain record, in SQL usually a `WHERE` clause is used. For example:

```sql
SELECT
    set_num, 
    name, 
    year 
FROM `lego_data.sets` 
WHERE year = 2019
```

In Logica, we can define the filtering during the predicate naming and extraction. There are two ways to do this (that I know of currently :D). The first way is:

```python
%%logica Sets

Sets(set_num:, name:) :- 
  `lego_data.sets`(set_num:, name:, year: 2019);
```

However, as you can see, using this way, the columns that are used for filtering is not extracted into the final predicate. For me, this is a bit of a setback because I wanted to check if the year is correct in the final extraction. But, it may seems like a redundant on some and they prefer it this way. Anyway, another way to go at it is as follows:

```python
%%logica Sets

Sets(set_num:, name:, year:) :- 
  `lego_data.sets`(set_num:, name:, year:),
    year == 2019;
```

Using this method could get you all the columns back. So, I think this might be more preferable to most.

#### `LIKE` operator and wildcard

In Logica, some of the SQL functions and operator that is available in BigQuery can be use with a function-like syntax. The function is written in camel-case. For example, this is the usage of `LIKE` operator and a wildcard.

```python
%%logica Sets

Sets(set_num:, name:, year:) :- 
  `lego_data.sets`(set_num:, name:, year:),
    Like(name, "%Star Wars%");
```
In the example above, I tried to search for all the sets that contain the word Star Wars.

#### `BETWEEN`, `AND`, and `OR` operator

As of the time during this writing, I could not find the `BETWEEN` operator so I had to turn to another method that could act as one. `BETWEEN` operator is similar to the `AND` operator when you put the value in a range. For example `WHERE x BETWEEN 1 AND 6` is similar with `WHERE x >= 1 AND x <= 6`. In Logica, the operator `AND` and `OR` can be written as `&&` and `||` respectively. For example:

```python
%%logica Sets

Sets(set_num:, name:, year:) :- 
  `lego_data.sets`(set_num:, name:, year:), 
    year > 2000 && year < 2010;
```
This will get all the LEGO sets that were made between the year 2001 and 2009. And for an example of `OR` operator:

```python
%%logica Sets

Sets(set_num:, name:, year:) :-
  `lego_data.sets`(set_num:, name:, year:), 
    Like(name, "%Star Wars%") || Like(name, "%Harry Potter%");
```
This one will get us the sets that contains the word Star Wars or Harry Potter.

#### `ORDER BY` and `LIMIT` clause
The usage of `ORDER BY` and `LIMIT` require us to explicitly state it before the extraction of a predicate. For example:

```python
%%logica Sets

@OrderBy(Sets,"pcs desc");
@Limit(Sets, 10);
Sets(set_num:, name:, year:, pcs:) :- 
  `lego_data.sets`(set_num:, name:, year:, num_parts: pcs), 
    year >= 2001 && year <= 2010;
```

Similarly, in SQL:

```sql
SELECT 
    set_num,
    name,
    year,
    num_parts AS pcs 
FROM `lego_data.sets`
WHERE year >= 2001 AND year <= 2010
ORDER BY pcs DESC
LIMIT 10
```

So, basically this will extract the top 10 set with the highest number of pieces made between the year 2001 and 2010. As you can see, in order to use the `ORDER BY` and `LIMIT`, we can use it in a similar pattern as a Python function. For `ORDER BY`, we can set as `@OrderBy(<predicate>,"<column> <blank:default(asc)/desc>")`. In the case of `LIMIT`; `@Limit(<predicate>, <no. of limit:int>`. There are a few other functions that are written this way but I have yet to learn them all.

### Aggregation function
Here, I will show how to use some of the most common aggregation functions: `MAX`,`MIN`,`AVG`,`COUNT`, and `SUM`

#### `MAX()` and `MIN()` function
For example, if you want to get the least number of parts or pieces of a LEGO set each year, you can do as follows:

```python
%%logica Sets

Sets(year:, min_pcs? Min=num_parts) distinct:- 
  `lego_data.sets`(year:, num_parts:);
```

And, if you want to get the most number of parts or pieces of a LEGO set each year:

```python
%%logica Sets

Sets(year:, max_pcs? Max=num_parts) distinct:- 
  `lego_data.sets`(year:, num_parts:);
```

As you can see, the way to write an aggregate function is: `<field name>? <aggregate function>=<aggregate input>` and before the extraction from the table, we have to put `distinct`. Here, `distinct` is actually equivalent with `GROUP BY` in an SQL syntax. So in SQL, it will be something similar as this:

```sql
SELECT 
    year,
    MIN(num_parts) AS min_pcs,
    MAX(num_parts) AS max_pcs
FROM `lego_data.sets`
GROUP BY year
```

#### `AVG()`, `COUNT()`, and `SUM()` function
Similar with the above function, in Logica, these function can be written as follows:

```python
%%logica AvgSets, CountSets, SumSets

AvgSets(year:, avg_pcs? Avg=num_parts) distinct:- 
  `lego_data.sets`(year:, num_parts:);
 
CountSets(year:, num? Count=name) distinct:- 
  `lego_data.sets`(year:, name:);
  
SumSets(year:, pcs? Sum=num_parts) distinct:- 
  `lego_data.sets`(year:, num_parts:);
```

So, `AvgSets` will execute the `AVG()` function and get us the average number of parts or pieces for each year and `CountSets` will call the `COUNT()` function which will then count the number of sets that LEGO made for each year. Lastly, `SumSets` will execute the `SUM()` function and count all the number of pieces that LEGO made each year.

### `JOIN` and `UNION` - Joining multiple tables in Logica

#### `INNER JOIN` 
One of the most executed work or function in SQL is probably the joining of multiple tables. It can be at least as two tables or it can be more than that. In Logica, the joining of tables can be done quite easily especially for an `INNER JOIN`. You just need to share the same column name when querying, extracting or filtering the data.

Here, I will show you on how to do an inner join. In this case, we will join the `Sets` table with the `Theme` table. The `Sets` table contains information on a certain LEGO set but it did not explicitly show the theme information but exchanged it with a theme ID number. So, in order to get more information on the theme of a certain set, we need to join those table together.

```python
%%logica Sets

LegoThemes(..i) :- `lego_data.themes`(..i);
LegoSets(..r) :- `lego_data.sets`(..r);

Sets(set_num:, name:, year:, theme_id:, theme_name:, num_parts: ) :- 
  LegoSets(set_num:, name:, year:, theme_id:, num_parts:), 
  LegoThemes(id:theme_id, name:theme_name, parent_id:);
```

This will join both table while changing the `name` from the `Theme` table to `theme_name`. If this is written in SQL, it will be:

```sql
SELECT 
    LegoSets.set_num,
    LegoSets.name,
    LegoSets.theme_id,
    LegoTheme.name AS theme_name,
    LegoSets.num_parts,
FROM `lego_data.sets` LegoSets
JOIN `lego_data.themes` LegoThemes
ON LegoSets.theme_id = LegoThemes.id
```

This is where I think Logica really shines because when it comes to multiple tables, it will be more complicated that this. In Logica, it is easier to keep track of the column name changes from each table and the column that we joined them on.

#### `OUTER JOIN`, and `LEFT JOIN`
Here, Logica turns from nice and easy to a little bit complicated :laughing:. To do an `OUTER JOIN` or `LEFT JOIN`, we have to define the key that we want to join them on. Then we assigned the value using a function. Because of its complexity, I will use a dummy data (It is not actually dummy but just an excerpt taken from the whole data :laughing:).

```python
%%logica OuterJoin, LeftJoin

Sets(set_num:"76382-1", name:"Hogwarts Moment: Transfiguration Class", 
year: 2021, theme_id: 246, num_parts: 241);
Sets(set_num:"9676-1", name:"TIE Interceptor & Death Star", 
year: 2012, theme_id: 158, num_parts: 65);
Theme(id:158, name:"Star Wars", parent_id:null);
Theme(id:246, name:"Harry Potter", parent_id:null);
Theme(id:435, name:"Ninjago", parent_id:null);

# Determining the keys in the full outer join table.
GetId(theme_id:) distinct :-
  Theme(id:theme_id) | Sets(theme_id:);

# Writing functions for retrieving an outer joined
# record if one exists.
GetSetNum(theme_id) = set_num :-
  set_num AnyValue= (x :- Sets(theme_id:, set_num: x));
GetName(theme_id) = name :-
  name AnyValue= (x :- Sets(theme_id:, name: x));
GetYear(theme_id) = year :-
  year AnyValue= (x :- Sets(theme_id:, year: x));
GetPcs(theme_id) = num_parts :-
  num_parts AnyValue= (x :- Sets(theme_id:, num_parts: x));
GetTheme(theme_id) = name :-
  name AnyValue= (x :- Theme(id:theme_id, name: x));
GetPID(theme_id) = parent_id :-
  parent_id AnyValue= (x :- Theme(id:theme_id, parent_id: x));

# Running full outer join.
OuterJoin(theme_id:, 
  theme_name:, 
  parent_id:, 
  set_num:, 
  name:, 
  year:, 
  num_parts:) :-
      GetId(theme_id:),
      theme_name == GetTheme(theme_id),
      parent_id == GetPID(theme_id),
      set_num == GetSetNum(theme_id),
      name == GetName(theme_id),
      year == GetYear(theme_id),
      num_parts == GetPcs(theme_id);

# Running left join.
LeftJoin(theme_id:, 
  theme_name:  
  set_num: GetSetNum(theme_id),
  name: GetName(theme_id),
  year: GetYear(theme_id),
  num_parts: GetPcs(theme_id)) :-
    Theme(id:theme_id, name:theme_name);

```

#### `UNION ALL`
In order to do a `UNION`, we can use this `|` symbol. Be careful to not be mistaken with the `OR` clause where it is written as `||`. It is quite similar but either it is a single or double symbol. For example;

```python
%%logica UnionTable

UnionTable(set_num:, name:, year:) :-
  `lego_data.sets`(set_num:, name:, year:1999) |
  `lego_data.sets`(set_num:, name:, year:2019);
```

This will perform a `UNION` on the sets that was made in 1999 and 2019.

I wasn't expecting the explanation to be this long 😆. I was going to perform some simple data exploration but since it became so long, I decide to break it into parts. So, I will continue with them on my next post. Stay tuned!

p/s: Actually, I had starting to write this post since last month but could not finish it. I guess, the right way maybe to write it in parts.
