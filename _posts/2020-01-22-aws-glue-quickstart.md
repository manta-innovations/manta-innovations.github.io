---
layout: post
title:  AWS Glue Quickstart
date:   2020-02-01
image:  glue_logo.png
author: jhole89
tags:   serverless, apache spark, aws glue, sbt
---

### AWS Glue Architecture
AWS Glue is a fully managed serverless ETL service that uses Apache Spark as an execution engine to run distributed
big data jobs through task parallelism. It differs to traditional on-premise or cloud based hadoop clusters by charging
on a usage base rather than provisioning and uptime cost. This is very efficient for apache spark and big data workloads,
which can often see short periods of intense compute followed by long periods of clusters idleness. On top of reducing 
the monetary cost of running spark clusters, this also avoids the DevOps and infrastructure burden of keeping these 
clusters healthy, which easily eats into time and energy that should be spent on adding value to the data.

When setting up a new AWS Glue job there are a number of settings to chose from, one of which is the Glue version 
(at the time of writing AWS Glue supports Spark versions 2.4.3 via Glue 1.0.0, and 2.2.1 via Glue 0.9.0). This is 
because Glue needs to know what version of Spark Cluster to run on and which version of spark to compile against - yes,
AWS Glue compiles your Spark code. When executing an AWS Glue job, the first stage is a compilation of the provided
script against the declared Glue/Spark version, and any other dependencies in your code. Due to this you need to provide
your raw files to AWS Glue, which it can compile, via the `script location` setting, however this is limited to a single
script file - all your logic must fit in a single script, which Glue will compile prior to executing.

### One file to rule them all!
Having all of our code in a single file is not ideal. At a cognitive level it means we can't create logical separations 
where one module of our code is associated and responsible for a single concept (e.g. logging). It means instead of 
having a natural subsection to start with when looking for something, a developer must search through potentially tens 
of thousands of lines of code that may have nothing to do with what they are looking for. At a technical level it means
diffs and merges become horrendous as people start stepping on each others toes refactoring the same file; and what 
about functionality we want used across multiple scripts? Say we have some business logic or standard lines of code 
(e.g. setting up our logger) that we ideally would abstract into a module and then invoke via a single call for each 
script. In fact, every AWS Glue script has a prerequisite code block that we could easily abstract seen here:
```scala
import com.amazonaws.services.glue.GlueContext
import com.amazonaws.services.glue.util.{GlueArgParser, Job}
import org.apache.spark.sql.SparkSession

import scala.collection.JavaConverters._

object GlueJob {
  val sparkSession: SparkSession = SparkSession.builder().getOrCreate()
  val glueCtx: GlueContext = new GlueContext(sparkSession.sparkContext)
    
  def main(sysArgs: Array[String]): Unit = {
    val args: Map[String, String] = GlueArgParser.getResolvedOptions(sysArgs, Seq("JOB_NAME").toArray)
    
    Job.init(args("JOB_NAME"), glueCtx, args.asJava)
    
    /* Your code goes here */
    
    Job.commit()
  }
}
```
Because we're limited to one file we'd end up copying this large block of code and others into each script, creating huge 
amounts of code repetition and breaking the principle of _DRY_ (Don't Repeat Yourself). Separation is good, abstraction
is good, re-usability is good...DON'T REPEAT YOURSELF.

### Monorepo's to the rescue
So how can we avoid repeating ourselves when we are limited to having all of our code in a single file? Well, AWS Glue
does allow us to provide other compiled packages to the job via an `--extra-jars` parameter. This means we can move
any common, shared, or abstracted logic from across our scripts into a shared jar which we can provide to the Glue job,
and be seen an as external dependency as any other third party library would. We can also bundle any other third party 
libraries within our shared jar as an uber/fat-jar and just provide Glue a single jar dependency (excluding the AWS Glue 
and Apache Spark libraries which are provided by AWS). To accomplish this we use monorepo's to create a project repo
separated into two (or more) logical modules, scripts (for individual scripts) and shared (for shared logic, base 
classes, loggers, spark contexts, etc), and we link shared as a dependency to scripts. This can be easily set up by 
creating two top level directory paths `scipts/src/main/scala/scripts` and `shared/src/main/scala/shared` at our 
project root and configuring our `sbt.build` file (Scala's build tool) as such:
```scala
lazy val shared = project.settings(settings, libraryDependencies ++= commonDependencies)
lazy val scripts = project
  .settings(settings, libraryDependencies ++= commonDependencies)
  .dependsOn(shared % "compile->compile;test->test")
```

With this structure any sbt tasks we execute are run against both modules independently. This means we can now run a 
`sbt package` and produce two jars - one composed of our scripts only, which as Glue requires uncompiled files we 
don't actually need and thus can ignore (we can actually configure this module to not even produce a jar with a couple
of extra settings), and another compose of our shared library which we can set the Glue jobs to
use. Now that we have a way to abstract away and use common functionality, a typical development pattern would be
1. Start writing your Glue scripts & tests inside the `scripts` module
2. Abstract any shared functionality into the `shared` module
3. Run tests, style checks, and package our code via `sbt test`, `sbt scalastyle`, and `sbt package` respectfully
4. Upload your Glue scripts (located within `scripts/src/main/scala/scripts`) and shared library jar (located within 
`shared/target`) to AWS S3 buckets
5. Create a AWS Glue Job which points to the script, set the main class using the parameter 
`--class scripts.<your_script_name>`, set the shared dependency using the parameter `--extra-jars <your_jar_name>.jar`

### Can't this be automated?
With a pattern established, its important that we make this easy to reuse again and again, rather than keep recreating
the same build files. For that purpose we've created a [quickstart repo](https://www.github.com/jhole89/aws-glue-sbt-quickstart) 
available free for use that contains everything to get started, including customisable style and format checking, 
a basic logger, spark and glue contexts, and a base glue script to extend from. This means we can spend less time on 
setting up boilerplate and jump straight into coding Glue scripts and running them in the cloud. Simply clone or fork
the repo and follow the [README](https://github.com/jhole89/aws-glue-sbt-quickstart/blob/master/README.md) to get started, or 
alternatively check out the [build.sbt](https://github.com/jhole89/aws-glue-sbt-quickstart/blob/master/build.sbt) to see how to
adapt this pattern to an already existing repo.

*Note:* While AWS Glue and Apache Spark support writing jobs in Scala and Python (via PySpark), we have only talked
about Glue and Spark in the context of Scala here. This is because Apache Spark is written in Scala and we encourage 
anyone using Spark to interface with it using Scala. Ignoring the numerous typesafe benefits of a language like Scala, 
if we are using a framework written in one language, we should use that language to integrate with it. Using PySpark is 
the equivalent of trying to write a Sprint Boot application via a Python wrapper - there's a reason this doesn't exist, 
it's a bad idea. Despite this all the above still stands for PySpark, however instead of jars we have to provide our 
shared library as a zipfile, but all the rest of the ideas and patterns still hold.