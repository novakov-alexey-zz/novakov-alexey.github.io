+++
title="Spark API Languages"
date=2022-10-07
draft = false

[extra]
category="blog"
toc = true

[taxonomies]
tags = ["spark", "scala", "java", "python"]
categories = ["scala", "big data"]
+++

{{ resize_image(path="spark-api/spark_logo.jpg", width=620, height=620, op="fit_width") }}

# Context

As part of my role of Data Architect at work, I often deal with AWS data services to run Apache Spark jobs such as EMR and Glue ETL. At the very beginning team needed to choose Spark supported programming language and start writing our main jobs for data processing.

Before we further dive into the languages choice, let's quickly remind what is Spark for EMR. Glue ETL is going to be skipped from the blog-post.

[Apache Spark](https://github.com/apache/spark) is one of the main component of [AWS EMR](https://aws.amazon.com/emr/), which makes EMR still meaningful service to be used by Big Data teams. AWS EMR team is building its own Spark distribution to integrate it with other EMR applications seamlessly. Even though Amazon builds own Spark, they keep the same Spark version, which is equal to open source version of Spark. All features of Apache Spark are available in EMR Spark. EMR allows to run a Spark application in EMR cluster via step type called “Spark Application”.

<!-- more -->

# Spark APIs

Apache Spark is mainly written in [Scala programming language](https://www.scala-lang.org/), which is running on top of JVM. Scala is more than the 15 years old programming language with state of the art eco-system of development tools like SBT, its plugin eco-system and many others. Scala was developed to enable programmers to write scale-able code in terms of code base size. That means one can tackle very large problem with small amount of Scala code.
Spark has also Java code in its main code base. Both languages are working in the same application without any problems. With rise of Data Science domain, Spark community brought R and Python languages support and started to support their SDKs officially as part of the Apache Spark repository. One of the main reason of adding Python and R was to enable Data Scientists to do data preparation for their further work using massive parallel execution engine which Spark is. In summary, there are 5 APIs that Spark supports today:

1. Scala
1. Java
1. SQL
1. Python a.k.a PySpark 
1. R

SQL is available from all supported above languages using string expressions.
First 3 APIs are native, so entire application execution is controlled only by JVM instances running on driver and executor side. In case of PySpark, there are Python interpreters on driver and executor sides, which require additional resources on cluster and client side. However, most of the execution is still happening inside the JVMs even the PySpark SDK is submitting the code. Spark DataFrame API was introduced in Spark some time ago to align performance of all supported languages. There is one major downside still exists when using Python. This issue arises when Spark JVM code needs to communicate with Spark-Python user code back and forth. Communication between Python and JVM processes is done via TCP Sockets, so the overhead of data serialization/de-serialization takes place. Such communication is not so intensive to produce performance penalties, unless user is using User-defined-Function (UDFs) in Python code.
Other Python limitations are coming from the language ergonomics itself. These limitations hurt productivity of data engineers especially when it comes to write internal/corporate library with typical data processing jobs. Still, Python can be easily used for data exploration in EMR Notebooks and/or PySpark shell.

# Scala Benefits

1. Statically typed code is verified by Scala compiler. Enables ease of program code refactoring. This is the main feature of language like Scala that eliminates issues in runtime, which may cause a lot of harm if data processing job fails on some program statement after running for 4 hours already.
1. IDE support: code error syntax highlighting, autocomplete, go-to definition to Spark Scala/Java API or own application code. [PyCharm](https://www.jetbrains.com/pycharm/) or similar Python IDE do not easily work with Spark type-stubs. Out of the Spark, IDE features are by an order of magnitude better for statically typed languages than for dynamically typed languages.
1. SBT (scala build tool similar to Maven in Java) allows to version and package Spark program code as a JAR file using [sbt-assembly](https://github.com/sbt/sbt-assembly) plugin and upload it to S3 bucket using few build commands.
1. Huge eco-system of Java and Scala libraries. Any library from Maven/Ivy repository can be called directly from Spark driver or executor side.
1. Scala benefits over Java language as well, because Java has quite verbose language syntax.

# Python Challenges

1. Syntax validation. Mypy or similar Python type-checker can be used to validate optional type annotations. However, that does not prevent a programmer to ship syntactically incorrect code to EMR Step execution, unless it is deployed by CI/CD job. Using Scala, one can't ship incorrect code to Spark without going through the compilation phase.
1. Refactoring is labor-intensive. Usually, Spark programs evolve to a home-grown library of data-processing jobs that requires code sharing to get manageable and scaleable codebase. Often times it is hard to refactor Python code automatically or semi-automatically even with IDE. Editors and IDEs work better for statically typed languages, since they introspect type annotations and make decision on wrong/correct syntax highlighting, automatic renaming and other refactoring tasks. Refactored Python program requires severe testing than it would be needed for a Scala program. Code refactoring here is not necessarily related to Spark API, but to that parts which combine user's and Spark's code all together.
1. White-space sensitivity and back-slash continuation syntax. Due to the fact that Spark API is extensively using “builder” pattern. Such as when program statements are split by dot and line break characters in the same time. This can also be done in Python with back-slash syntax or wrapping the entire statement in parentheses, which clutters the code.

# Scala Learning Curve

1. In order to write Spark code in Scala, one can learn that in one day. Spark DataFrame API is almost identical across all Spark languages. Programmers do not require to know all Scala language aspects, but they can discover them over time.
1. Scala function definition syntactically looks very like Python function definition. Difference becomes even smaller, when using optional type annotation in Python. Python developers usually find Scala's syntax quite familiar.
1. There are many resources and free courses on Scala fundamentals, which were produced over the last 10 years in Scala eco-system.
1. IntelliJ IDEA provides well-supported [JetBrains Scala Plugin](https://plugins.jetbrains.com/plugin/1347-scala) for a long time. There is also light alternative to IntelliJ like VSCode + [Metals LSP (Metals)](https://scalameta.org/metals/),
The last one is an implementation of Language Server Protocol for Scala.
1. Knowledge of Java helps to learn Scala, but they are not mandatory.


# Summary

No matter whether you are going to use data science libraries in Python further down the line of data pipelines, it worth to consider Scala as the main data processing language.
Having Scala as main language for your data platform APIs, data processing pipelines and DevOps makes the work of software engineers as a joy. Python may seems easy language to use and pick up, but its dynamic types and cumbersome syntax leads to a refactoring nightmare and makes the entire code base as one giant technical debt, which can't be easily eliminated.