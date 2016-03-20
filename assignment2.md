---
layout: page
title: Assignment 2
tagline: Big data assignment 2
description: An introduction to Spark.
---

# Introduction

Spark is a framework to work with big data easily. It does this by handling lots
of backend for you. That means you can import data in a variety of ways and then
work on it with simple code snippets. You can try out commands much like in an
interactive Python session, but Spark gives you a lot more performance and a lot
more options. It also gives you better organization of your products.

For this introduction, I'm going to assume you've already installed Spark and
you know your way around linux. I'm also going to assume you know your way
around whatever programming language you use with Spark.

# Notebooks

Spark can be run locally or remotely and can be accessed through a web
interface. If running locally, it will bind to `localhost:9000` by default. When
you visit this webpage, you'll land in the files tree. This is where all your
notebooks show up. A notebook is a collection of text and commands. The results
of the commands are saved and you can also rerun them. This allows you to gather
thoughts and experiments on data in one place. It allows you to easily compare
the results as it can also show graphs based on the output of commands. Apart
from notebooks, there are also directories in the files tree, which allow you to
organize your notebooks.

You can create new notebooks from the menu in the top right. You can delete or
duplicate them with the buttons on the right. When you open a notebook (by
clicking on the link at the left), a kernel will be started for it. This will
keep all data related to your live session. It allows you to execute the code in
the notebook and it remembers the data for you. Once you shut it down (the
delete button will have been replaced by a shutdown button, be careful not to
click twice), this is all gone. You can still edit the notebook, but you can no
longer run code on it. It is usually not necessary to shutdown notebooks as they
don't consume a lot of resources when not in use.

# Data

In order to work with big data, you need to put it somewhere on the filesystem.
If you've installed Spark directly on your system, this is easy enough, but if
you've installed Spark in a docker container, you'll need to use docker commands
to copy the data from your filesystem to the filesystem of the container. Once
you've done that, you can access it in a notebook. Begin by creating a new
notebook from the `New` menu in the top right of files page.

One of the easiest ways to check your data is there, is to use shell commands.
You can use `:sh`, followed by a shell command to run it on the underlying
system. Use for example `:sh ls /path/to/data` to check that it's there. Note
that `Enter` generates a newline, in order to run a command, press
`Ctrl+Enter`. The command stays in the first box. Below it, some code will
appear, indicating what is happening. Below that, the output will appear, marked
to the left with `Out[n]:` where n is the number of the command. Everytime you
run a command, it will increase to the next number. Run the command again using
`Ctrl+Enter` and watch it increase.

# Programming

The `:sh` syntax is used to run shell commands, but normally, the code is
executed as Scala code. (Or python code or something else if you have Spark set
up that way. We'll assume Scala here.) Let's look at an example.

    val rdd = sc.parallelize(0 to 999, 8)
    rdd.takeSample(false, 5)

`0 to 999` simply generates the range of numbers from 0 to 999. `sc` is the
spark context. This command will cut the collection of numbers into 8 partitions
which can then be analyzed on separate CPUs. All operations executed on this
distributed dataset will now be automatically parallelized. If the number of
partitions is not specified, it will be automatically determined based on the
number of cores available. `takeSample(false, 5)` simply selects 5 items from
the collection at random. If the first argument is set to `true`, it may take an
element more than once.

In order to read a data set from file, you can use:

    val lines = sc.textFile("/path/to/file")

Let's run some basic operations on it.

    val no_empty = lines.filter(_ != "")

This filters the lines and returns only the lines that satisfy the condition, in
this case, not being empty. The condition is specified as a function that takes
a line and produces a boolean. Scala allows a shorthand for this: In `_ != ""`,
the `_` represents the line coming in and the result of the expression is simply
the output of the function. Instead of removing empty lines, we can also make
sure they're not empty:

    val hellos = lines.map(line => "Hello! " + line)

This will prepend `Hello! ` to every line. Again, the argument is a function,
this time with a different notation. It says it takes a `line` as argument and
produces `"Hello! " + line` as result. We could also have written this as
`"Hello! " + _` which uses the syntax we used earlier. The map function used
here is the one referenced by the name MapReduce. The other function referenced
there is reduce:

    val file_contents = lines.reduce(_ + "\n" + _)

Here we're using the `_` syntax again. The first one refers to the first
argument the function should receive, and the second one to the second argument.
Reduce takes a function that takes two arguments and combines them. It then
calls this function on successive elements of the dataset and results from
previous calls until all data points have been combined into one. The example
above simply concatenates all lines together with newlines in the middle, thus
reconstructing the original content of the file which was split by the
`textFile` call.

This will be all for programming, you can find more functions
[here](https://spark.apache.org/docs/latest/programming-guide.html#transformations).

# Spark UI

The Spark UI gives some insight in what resources are being used by Spark. You
can reach it at `localhost:4040`. You'll be dropped in the Jobs tab. By looking
at the Event Timeline, you can see at what point jobs ran, how much time they
took and whether they succeeded. Below it is the same information in a table,
with a little bit more detail. Here you can also see how many tasks a job is
split in. (rightmost column) This will often correspond to the number you pass
as second argument to `parallelize`.

The second tab shows all stages separately. This seems similar to the first tab,
because so far, we've mostly run jobs that consist of only one stage. This tab
also shows how much disk / network IO was used to run a stage. The storage tab
shows what objects are currently being stored in memory. These are generally
your data sets, or analysis results you still want to reuse. The Environment tab
simply shows some environment variables for Spark. This can help if Spark ever
misbehaves. The last tab, Executors, shows the executors currently connected.
When Spark is used commercially, many executors split over many machines can be
connected, allowing large amounts of computation to happen in parallel.

# Execution Model

In order to avoid running computations that don't need to be run, Spark uses
lazy execution. This means subresults will only be computed once it's confirmed
that they're really necessary for the final result.

As mentioned before, the way to parellelize computations in Spark is by
partitioning the dataset. When simply using `sc.parallelize`, the best way to
partition is simply guessed. It's also possible to specify how a dataset should
be partitioned. This can be done with the following command:

    col1 = col0.partitionBy(new HashPartitioner(2))

where the `HashPartitioner` is just an arbitrary example. The 2 here indicates
the dataset will be split into 2 partitions.

# Conclusion

Spark is an awesome platform for playing around with large data sets and it
transitions well into serious applications.
