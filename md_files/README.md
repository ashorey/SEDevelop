[<img align="right" src="https://raw.githubusercontent.com/maxim8898/redoc/master/btn_subscribe.png">](https://shop.exchange.se.com/en-US/apps/38969/building-energy-modeling)
## Authentication - for AMITSHOREY

<img src="https://raw.githubusercontent.com/SE-Analytics-Team/public-images/master/forecasting_api/MicroApp.png" style="float: right" />

Authentication is the act of proving an assertion, such as the identify of a computer system user.

  | | |
  |-|-|
  | **Security scheme type**   | API Key       |
  | **Header parameter name**  | Authorization |
  
In contrast with identification, the act of indicating a person or thing's identity, authentication is the process of verifying that identify.

## Repository
Contains Jupyter notebook server, Livy driver to spin up spark, and spark as notebook executor

  | | |
  |-|-|
  | **Spark source**   | [Apache Spark 2.4.2](https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-without-hadoop-scala-2.11.tgz) |
  | **Hadoop source**  | [Apache Hadoop 3.2.0](http://www-us.apache.org/dist/hadoop/common/hadoop-3.2.0/) |
  | **Livy source**    | [Apache Livy 6 with customizations](https://github.com/jahstreet/incubator-livy) |
  | **Jupyter source** | [Jupyter notebook 5.7.5](https://github.com/jupyter/notebook) |
  
## Introduction 
All resources is containerized and prepared to use in Kubernetes. Also helm is planned (in future development)
Example of all infrastructure, that should be: https://github.com/jahstreet/spark-on-kubernetes-docker
Jupyter - custom configuration.
## Spark
Spark image is a basic for all others. Built from alpine with openjdk 8. If you could do it more light - do it.
Options to point attention:
1. SPARK_CLASSPATH or SPARK_DIST_CLASSPATH variable should include path to all *.jar libs, that could be used by Spark. Depending on config, submitted to Livy
2. Pay attention to configs and native libs, that should be copied to Hadoop: *-site.xml and from local native folder
```bash
COPY native/libhdfs.a $HADOOP_HOME/lib/native
COPY native/libsnappy.so.1.1.3 $HADOOP_HOME/lib/native
COPY native/libsnappy.so.1 $HADOOP_HOME/lib/native
COPY native/libsnappy.so $HADOOP_HOME/lib/native
COPY conf/core-site.xml $HADOOP_HOME/etc/hadoop
```
3. Put Livy libraries in the 'distr' folder to copy them into Spark. Livy is built from https://github.com/jahstreet/incubator-livy, as the version could work with kubernetes and spark 2.4* 
```
From 'repl_2.11-jars'
livy-core_2.11-0.7.0-incubating-SNAPSHOT.jar
livy-repl_2.11-0.7.0-incubating-SNAPSHOT.jar
From 'rsc-jars'
livy-rsc-0.7.0-incubating-SNAPSHOT.jar
livy-thriftserver-session-0.7.0-incubating-SNAPSHOT.jar
livy-api-0.7.0-incubating-SNAPSHOT.jar
```
Into Livy zipped distributive the folders 'repl_2.11-jars' 'rsc-jars' SHOULD NOT CONTAIN ANY FILES
Hadoop and Spark are taken from Apache sources according the version. Since 2.4.2 spark, it has been tacken without Scala and Hadoop. Apache likes update all dists suddenly, pay attention to it, and save local version.
Added modified Variables for Spark paths, after that spark could find jars when it is submitted by Livy.
Download via wget several libraries, requred to run notebooks. 
Install additional libs with pip.
Entrypoint - tini, that provide incoming command to spark. Tini helps to make core of Spark and Jupyter running more stable, and reccomended by community.
Mini conda is used as Python libs source and running platfrom. The image with full Anaconda is too huge for kubernetes, so it is needed to add aditional libs if they are required.
## Jupyter
In addition of Spark, this image installs Jupyter via conda:
```
RUN conda install --quiet --yes \\
    'notebook=5.7.5' \\
        'jupyterhub=0.9.4'
```
 and sparkmagic - the connector via jupyter and livy. Configure spark magic config properly. It will be processed by entrypoint script, which replace 'localhost' with LIVY_ENDPOINT service address from env var.
```
 RUN pip install sparkmagic
```
 Also several extensions for Jupyter are installed by Jupyter itself (kernels and so on):
```
 RUN jupyter nbextension enable --py --sys-prefix widgetsnbextension && \\
    jupyter-kernelspec install .......
```
 Entrypoint change Livy endpoint into sparkmagic config, and run Jupyter: notebook server, hub or lab.
    
