---
layout: post
title: Simple, Quick, and Easy Beam
subtitle: A simple (and hopefully, quick and easy) guide on Apache Beam (Python SDK)
tags: [apahce, beam, guide, introduction, beginner, python]
---

### Introduction
In the previous post, I talked a little bit about Beam. So, what exactly is Beam?

Beam is an open source project as a tool on data pipeline. It is one of the project under Apache Software Foundation and just recently, it got out of incubation (Yeay! 😄).

You can read more about it on their website [here](https://beam.apache.org/)

Basically, Beam is a tool for batch or streaming processing of data. That is where the name comes from **B**atch and str**eam**. Using the SDKs, you can use it to build a program to define the pipeline of the process. Currently, there are 3 SDKs; Java, Python and Go.

I was lucky to be introduced to Beam through my collegue at work but during that time there was not many guide on creating Beam pipeline using Python. (Maybe because most of the developer wrote it in Java, I guess)

So, here I will be going to show a simple tutorial (hopefully 😀) *for you to get your hands dirty* using a Python script. I will also try to explain the process and the Beam concept and terminology together with the process.

### Installing and importing dependencies
A few things to note, Beam is a DAG (directed acyclic graph), or in simpler terms, the processing done in the pipeline is like a *one-way road*. The data cannot be changed back after a process is done.

First and foremost you can make your own virtual environment if you prefer to but I will jump straight to installing the Beam dependencies.
Firstly, install the Beam Python SDK from PyPI.
```bash
pip install apache-beam
```

Beam can also be run with a distributed processing tools such as Spark, Flint, etc. This are called **runners**. When you run the program based on your choice of SDKs, you need to specify which runner you are going to use. If your are going to run it in the cloud (AWS, GCP, Azure, etc.), other dependencies are also required. You can check for more detail in their [website](https://beam.apache.org/get-started/quickstart-py/). For now, we are going to run it locally. So, there is no need to specify any runner. But, if you want to explicitly point them you can import them inside the program.

For the script, make sure to import the beam package first.
```python
import apache_beam as beam
```

### Options in the pipeline
If you choose to explicitly state the runner for running it locally. Import the **standard options** from the pipeline options. (This is were the statement of the type of runner is suppose to be written.
```python
from apache_beam.options.pipeline_options import StandardOptions

options.view_as(StandardOptions).runner = 'DirectRunner'
# for other type of runner, please check them on the website or documentation
```

There are also other options that can be adjusted, but, for easier purposes, I am going to skip them.

### Process in the pipeline
Next, create a function to state the transformation process. It can be any type of transformation; replace string, etc. As note, there are 2 terminology that you should know in Beam; **PCollection** and **Transform**. The simplest form of pipeline can be shown as this:
```bash
Input PCollection -> Transform >> Output PCollection
```

**PCollection** is a class provided by the SDKs to represent the a dataset. In can be data of any size. If the data is large, Beam will cut it into chunks of several dataset and run the **transform** process in parallel. In simple term, we can regard this PCollection as a bucket of data. Another thing to note that usually the data handled is in the form of text file, and Beam will read them line by line. Each line is then point as one element in a dataset.

**Transform** is the process or operation that is stated inside the pipeline. You provide the transformation in the form of function object. Beam also provided some of predefined transformation such as count.

One more thing that I forgot to mention is the **input** and **output** of the pipeline. The input of the pipeline can be files (usually txt or csv) or a database. You can connect is to any source of database. For the output, you can ingest it back into a database or output it as file (usually txt). For the purpose of this tutorial, we are going to create our own dataset (Yes, Beam can create data but a small one of course 😄) as input and just print out the output for visual purposes.
```python
# our input data
beam.Create(['cat dog','monkey dog','snake cat','dog','lion','cat'])

# for output we are going to use the print function
beam.Map(print))
```

As you can see I used Beam built-in function **create** and **map**. There are others but you can check those in the beam documentation.
**Map** is one of the transformation process in Beam. So, basically what we do here is print out the **PCollection** and the printed out of the data will be the **output**.

### First program of Beam
Now, we try to combine all the things that we learned.
```python
import apache_beam as beam
from apache_beam.options.pipeline_options import StandardOptions # optional

options.view_as(StandardOptions).runner = 'DirectRunner' #optional

beam.Create(['cat dog','monkey dog','snake cat','dog','lion','cat'])

beam.Map(print))
```

But wait, we forgot one important part. In Beam, we have to define the pipeline. So the correct way to write is as follows (for a simpler purpose, from here onwards I will disregard the optional parts): 
```python
import apache_beam as beam

with beam.Pipeline() as p:
  output = (
    p | "create data" >> beam.Create(['cat dog','monkey dog','snake cat','dog','lion','cat'])
      | "print out data" >> beam.Map(print)
      )
```

Now, our pipeline is complete. Try to run the above code and you will get these
```bash
cat dog
monkey dog
snake cat
dog
lion
cat
```

In a simple term the ```output``` is the output of the data and to get the output, Beam ```create``` the data and ```map``` the data with the built-in python ```print``` function. The whole process runs inside the pipeline that is defined by ```p```. To show this as a pipeline shown above regarding PCollection, you can understand it as:
```bash
create data (Input PCollection) -> print out data (Transform) -> output (Output PCollection)
```

You may notice that in the above python code that we can each part of the pipeline.

### Transformation in a pipeline
How about that for a start? Maybe it is hard to understand because in the above example we define print as a **transform**. So, to understand more about a real transformation, we will add another layer of transformation inside the pipeline. This time rather than using a built-in function, we will define our own function. Here will be the function that we will use.
```python
def SplitItem(element):
  return element.split(' ')
```

Next, we will add the function inside the pipeline.
```python
p | "create data" >> beam.Create(['cat dog','monkey dog','snake cat','dog','lion','cat'])
  | "split data" >> beam.FlatMap(SplitItem)
  | "print out data" >> beam.Map(print)
```

Notice that, rather than the ```Map``` function, we used the ```FlatMap```. The difference is that ```Map``` applies a simple 1-to-1 mapping function over each element in the collection but ```FlatMap``` applies a simple 1-to-many mapping function over each element in the collection. The many elements are flattened into the resulting collection. You can view the source [here](https://beam.apache.org/documentation/transforms/python/elementwise/map/) and [here](https://beam.apache.org/documentation/transforms/python/elementwise/flatmap/). This will be the full code:
```python
import apache_beam as beam

def SplitItem(element):
  return element.split(' ')
  
with beam.Pipeline() as p:
  output = (
    p | "create data" >> beam.Create(['cat dog','monkey dog','snake cat','dog','lion','cat'])
      | "split data" >> beam.FlatMap(SplitItem)
      | "print out data" >> beam.Map(print)
      ) 
```

The output will return the results as follows
```bash
cat 
dog
monkey 
dog
snake 
cat
dog
lion
cat
```

Just as a side note. If ```Map``` was used rather than ```FlatMap```, the results will be:
```bash
['cat', 'dog']
['monkey', 'dog']
['snake', 'cat']
['dog']
['lion']
['cat']
```

As you can see the data is written nicely with ```FlatMap```. Each element per line. The pipeline now becomes:
```bash
create data (Input PCollection) -> split data (Transform) -> (PCollection) -> print out data (Transform) -> output (Output PCollection)
```

So, as you can see based on the pipeline, now it is hard see the data between ```split data``` and ```print out data``` right? But, because we just use a print function what happen after ```split data``` is exactly the same as ```print out data```.

Now, how about we add another layer of complexity 😁. We are going to count each element and output them but this time we are going to use Beam built-in ```count``` function. The code is shown as follows:
```python
import apache_beam as beam

def SplitItem(element):
  return element.split(' ')
  
with beam.Pipeline() as p:
  output = (
    p | "create data" >> beam.Create(['cat dog','monkey dog','snake cat','dog','lion','cat'])
      | "split data" >> beam.FlatMap(SplitItem)
      | "count data" >> beam.combiners.Count.PerElement()
      | "print out data" >> beam.Map(print)
      ) 
```

If you run the above code, the results will be as follows:
```bash
('cat', 3)
('dog', 3)
('monkey', 1)
('snake', 1)
('lion', 1)
```

And, the pipeline becomes
```bash
create data (Input PCollection) -> split data (Transform) -> (PCollection) -> count data (Transform) -> (PCollection) -> print out data (Transform) -> output (Output PCollection)
```

As you can see, the data is counted and we could get the numbers for each element. So, the provided function is a very good one and there are others that you can check [here](https://beam.apache.org/documentation/transforms/python/aggregation/count/)

### Wrapping up
To sum it all up, we did a simple pipeline using Apache Beam in Python using our own function, Python built-in function and also Beam built-in function. As you can see, Beam is pretty much a versatile tool for data processing. You can expand and modified it to your own liking.
If you found any mistakes in the post, please contact me and If you have any pipeline that you built that you would like to share, please do feel free to share it here.
As a side note, the InteractiveRunner for Beam was recently developed and I could see a lot more potential to use it inside a Jupyter Notebook (well, it is one of the most used tool for data practitioners 😀)
