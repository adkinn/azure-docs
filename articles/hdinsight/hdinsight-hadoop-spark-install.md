---
title: Use Script Action to install Spark on Hadoop cluster | Microsoft Docs
description: Learn how to customize an HDInsight cluster with Spark using Script Action.
services: hdinsight
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun

ms.assetid: 7ebf4e2b-0742-4a2f-b429-60dc30d3f905
ms.service: hdinsight
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/05/2016
ms.author: nitinme

---
# Install and use Spark on HDInsight Hadoop clusters using Script Action
> [!IMPORTANT]
> This article is now deprecated. HDInsight now provides Spark as a first-class cluster type for Windows-based clusters, which means you can now directly create a Spark cluster without modifying a Hadoop cluster using Script action. Using the Spark cluster type, you get an HDInsight version 3.2 cluster with Spark version 1.3.1.  To install different versions of Spark, you can use Script action. HDInsight provides a sample Script Action script.
>
>

Learn how to install Spark on Windows based HDInsight using Script Action, and how to run Spark queries on HDInsight clusters.

**Related articles**

* [Create Hadoop clusters in HDInsight](hdinsight-provision-clusters.md): general information on creating HDInsight clusters.
* [Get Started with Apache Spark on HDInsight](hdinsight-apache-spark-jupyter-spark-sql.md): create an HDInsight Spark cluster.
* [Customize HDInsight cluster using Script Action][hdinsight-cluster-customize]: general information on customizing HDInsight clusters using Script Action.
* [Develop Script Action scripts for HDInsight](hdinsight-hadoop-script-actions.md).

## What is Spark?
<a href="http://spark.apache.org/docs/latest/index.html" target="_blank">Apache Spark</a> is an open-source parallel processing framework that supports in-memory processing to boost the performance of big-data analytic applications. Spark's in-memory computation capabilities make it a good choice for iterative algorithms in machine learning and graph computations.

Spark can also be used to perform conventional disk-based data processing. Spark improves the traditional MapReduce framework by avoiding writes to disk in the intermediate stages. Also, Spark is compatible with the Hadoop Distributed File System (HDFS) and Azure Blob storage so the existing data can easily be processed via Spark.

This topic provides instructions on how to customize an HDInsight cluster to install Spark.

## Install Spark using the Azure Portal
A sample script to install Spark on an HDInsight cluster is available from a read-only Azure storage blob at [https://hdiconfigactions.blob.core.windows.net/sparkconfigactionv03/spark-installer-v03.ps1](https://hdiconfigactions.blob.core.windows.net/sparkconfigactionv03/spark-installer-v03.ps1). This script can install Spark 1.2.0 or Spark 1.0.2 depending on the version of the HDInsight cluster you create.

* If you use the script while creating an **HDInsight 3.2** cluster, it installs **Spark 1.2.0**.
* If you use the script while creating an **HDInsight 3.1** cluster, it installs **Spark 1.0.2**.

You can modify this script or create your own script to install other versions of Spark.

> [!NOTE]
> The sample script works only with HDInsight 3.1 and 3.2 clusters. For more information on HDInsight cluster versions, see [HDInsight cluster versions](hdinsight-component-versioning.md).
>
>

1. Start creating a cluster by using the **CUSTOM CREATE** option, as described at [Create Hadoop clusters in HDInsight](hdinsight-provision-clusters.md). Pick the cluster version depending on the following:

   * If you want to install **Spark 1.2.0**, create an HDInsight 3.2 cluster.
   * If you want to install **Spark 1.0.2**, create an HDInsight 3.1 cluster.
2. On the **Script Actions** page of the wizard, click **add script action** to provide details about the script action, as shown below:

    ![Use Script Action to customize a cluster](./media/hdinsight-hadoop-spark-install/HDI.CustomProvision.Page6.png "Use Script Action to customize a cluster")

    <table border='1'>
        <tr><th>Property</th><th>Value</th></tr>
        <tr><td>Name</td>
            <td>Specify a name for the script action. For example, <b>Install Spark</b>.</td></tr>
        <tr><td>Script URI</td>
            <td>Specify the Uniform Resource Identifier (URI) to the script that is invoked to customize the cluster. For example, <i>https://hdiconfigactions.blob.core.windows.net/sparkconfigactionv03/spark-installer-v03.ps1</i></td></tr>
        <tr><td>Node Type</td>
            <td>Specify the nodes on which the customization script is run. You can choose <b>All nodes</b>, <b>Head nodes only</b>, or <b>Worker nodes only</b>.
        <tr><td>Parameters</td>
            <td>Specify the parameters, if required by the script. The script to install Spark does not require any parameters so you can leave this blank.</td></tr>
    </table>

    You can add more than one script action to install multiple components on the cluster. After you have added the scripts, click the checkmark to start creating the cluster.

You can also use the script to install Spark on HDInsight by using Azure PowerShell or the HDInsight .NET SDK. Instructions for these procedures are provided later in this topic.

## Use Spark in HDInsight
Spark provides APIs in Scala, Python, and Java. You can also use the interactive Spark shell to run Spark queries. This section provides instructions on how to use the different approaches to work with Spark:

* [Use the Spark shell to run interactive queries](#sparkshell)
* [Use the Spark shell to run Spark SQL queries](#sparksql)
* [Use a standalone Scala program](#standalone)

### <a name="sparkshell"></a>Use the Spark shell to run interactive queries
Perform the following steps to run Spark queries from an interactive Spark shell. In this section, we run a Spark query on a sample data file (/example/data/gutenberg/davinci.txt) that is available on HDInsight clusters by default.

1. From the Azure portal, enable Remote Desktop for the cluster you created with Spark installed, and then remote into the cluster. For instructions, see [Connect to HDInsight clusters using RDP](hdinsight-administer-use-management-portal.md#connect-to-clusters-using-rdp).
2. In the Remote Desktop Protocol (RDP) session, from the desktop, open the Hadoop command line (from a desktop shortcut), and navigate to the location where Spark is installed; for example, **C:\apps\dist\spark-1.2.0**.
3. Run the following command to start the Spark shell:

         .\bin\spark-shell --master yarn

    After the command finishes running, you should get a Scala prompt:

         scala>
4. On the Scala prompt, enter the Spark query shown below. This query counts the occurrence of each word in the davinci.txt file that is available at the /example/data/gutenberg/ location on the Azure Blob storage associated with the cluster.

        val file = sc.textFile("/example/data/gutenberg/davinci.txt")
        val counts = file.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_ + _)
        counts.toArray().foreach(println)
5. The output should resemble the following:

    ![Output from running Scala interactive shell in an HDInsight cluster](./media/hdinsight-hadoop-spark-install/hdi-scala-interactive.png)
6. Enter :q to exit the Scala prompt.

        :q

### <a name="sparksql"></a>Use the Spark shell to run Spark SQL queries
Spark SQL allows you to use Spark to run relational queries expressed in Structured Query Language (SQL), HiveQL, or Scala. In this section, we look at using Spark to run a Hive query on a sample Hive table. The Hive table used in this section (called **hivesampletable**) is available by default when you create a cluster.

> [!NOTE]
> The sample below was created against **Spark 1.2.0**, which is installed if you run the script action while creating HDInsight 3.2 cluster.
>
>

1. From the Azure portal, enable Remote Desktop for the cluster you created with Spark installed, and then remote into the cluster. For instructions, see [Connect to HDInsight clusters using RDP](hdinsight-administer-use-management-portal.md#connect-to-clusters-using-rdp).
2. In the RDP session, from the desktop, open the Hadoop command line (from a desktop shortcut), and navigate to the location where Spark is installed; for example, **C:\apps\dist\spark-1.2.0**.
3. Run the following command to start the Spark shell:

         .\bin\spark-shell --master yarn

    After the command finishes running, you should get a Scala prompt:

         scala>
4. On the Scala prompt, set the Hive context. This is required to work with Hive queries by using Spark.

        val hiveContext = new org.apache.spark.sql.hive.HiveContext(sc)

    Note that **sc** is default Spark context that is set when you start the Spark shell.
5. Run a Hive query by using the Hive context and print the output to the console. The query retrieves data on devices of a specific make and limits the number of records retrieved to 20.

        hiveContext.sql("""SELECT * FROM hivesampletable WHERE devicemake LIKE "HTC%" LIMIT 20""").collect().foreach(println)
6. You should see an output like the following:

    ![Output from running Spark SQL on an HDInsight cluster](./media/hdinsight-hadoop-spark-install/hdi-spark-sql.png)
7. Enter :q to exit the Scala prompt.

        :q

### <a name="standalone"></a>Use a standalone Scala program
In this section, we write a Scala application that counts the number of lines containing the letters 'a' and 'b' in a sample data file (/example/data/gutenberg/davinci.txt) that is available on HDInsight clusters by default. To write and use a standalone Scala program with a cluster customized with Spark installation, you must perform the following steps:

* Write a Scala program
* Build the Scala program to get the .jar file
* Run the job on the cluster

#### Write a Scala program
In this section, you write a Scala program that counts the number of lines containing 'a' and 'b' in the sample data file.

1. Open a text editor and paste the following code:

        /* SimpleApp.scala */
        import org.apache.spark.SparkContext
        import org.apache.spark.SparkContext._
        import org.apache.spark.SparkConf

        object SimpleApp {
          def main(args: Array[String]) {
            val logFile = "/example/data/gutenberg/davinci.txt"            //Location of the sample data file on Azure Blob storage
            val conf = new SparkConf().setAppName("SimpleApplication")
            val sc = new SparkContext(conf)
            val logData = sc.textFile(logFile, 2).cache()
            val numAs = logData.filter(line => line.contains("a")).count()
            val numBs = logData.filter(line => line.contains("b")).count()
            println("Lines with a: %s, Lines with b: %s".format(numAs, numBs))
          }
        }

1. Save the file with the name **SimpleApp.scala**.

#### Build the Scala program
In this section, you use the <a href="http://www.scala-sbt.org/0.13/docs/index.html" target="_blank">Simple Build Tool</a> (or sbt) to build the Scala program. sbt requires Java 1.6 or later, so make sure you have the right version of Java installed before continuing with this section.

1. Install sbt from http://www.scala-sbt.org/0.13/tutorial/Installing-sbt-on-Windows.html.
2. Create a folder called **SimpleScalaApp**, and within this folder create a file called **simple.sbt**. This is a configuration file that contains information about the Scala version, library dependencies, etc. Paste the following into the simple.sbt file and save it:

        name := "SimpleApp"

        version := "1.0"

        scalaVersion := "2.10.4"

        libraryDependencies += "org.apache.spark" %% "spark-core" % "1.2.0"



    >[AZURE.NOTE] Make sure you retain the empty lines in the file.


1. Under the **SimpleScalaApp** folder, create a directory structure **\src\main\scala** and paste the Scala program (**SimpleApp.scala**) you created earlier under the \src\main\scala folder.
2. Open a command prompt, navigate to the SimpleScalaApp directory, and enter the following command:

        sbt package


    Once the application is compiled, you will see a **simpleapp_2.10-1.0.jar** file created under the **\target\scala-2.10** directory within the root SimpleScalaApp folder.


#### Run the job on the cluster
In this section, you remote into the cluster that has Spark installed and then copy the SimpleScalaApp project's target folder. You then use the **spark-submit** command to submit the job on the cluster.

1. Remote into the cluster that has Spark installed. From the computer where you wrote and built the SimpleApp.scala program, copy the **SimpleScalaApp\target** folder and paste it to a location on the cluster.
2. In the RDP session, from the desktop, open the Hadoop command line, and navigate to the location where you pasted the **target** folder.
3. Enter the following command to run the SimpleApp.scala program:

        C:\apps\dist\spark-1.2.0\bin\spark-submit --class "SimpleApp" --master local target/scala-2.10/simpleapp_2.10-1.0.jar

1. When the program finishes running, the output is displayed on the console.

        Lines with a: 21374, Lines with b: 11430

## Install Spark using Azure PowerShell
In this section, we use the **<a href = "http://msdn.microsoft.com/library/dn858088.aspx" target="_blank">Add-AzureHDInsightScriptAction</a>** cmdlet to invoke scripts by using Script Action to customize a cluster. Before proceeding, make sure you have installed and configured Azure PowerShell. For information on configuring a workstation to run Azure PowerShell cmdlets for HDInsight, see [Install and configure Azure PowerShell](../powershell-install-configure.md).

Perform the following steps:

1. Open an Azure PowerShell window and declare the following variables:

        # Provide values for these variables
        $subscriptionName = "<SubscriptionName>"        # Name of the Azure subscription
        $clusterName = "<HDInsightClusterName>"            # HDInsight cluster name
        $storageAccountName = "<StorageAccountName>"    # Azure Storage account that hosts the default container
        $storageAccountKey = "<StorageAccountKey>"      # Key for the Storage account
        $containerName = $clusterName
        $location = "<MicrosoftDataCenter>"                # Location of the HDInsight cluster. It must be in the same data center as the Storage account.
        $clusterNodes = <ClusterSizeInNumbers>            # Number of nodes in the HDInsight cluster
        $version = "<HDInsightClusterVersion>"          # For example, "3.2"
2. Specify the configuration values such as nodes in the cluster and the default storage to be used.

        # Specify the configuration options
        Select-AzureSubscription $subscriptionName
        $config = New-AzureHDInsightClusterConfig -ClusterSizeInNodes $clusterNodes
        $config.DefaultStorageAccount.StorageAccountName="$storageAccountName.blob.core.windows.net"
        $config.DefaultStorageAccount.StorageAccountKey=$storageAccountKey
        $config.DefaultStorageAccount.StorageContainerName=$containerName
3. Use the **Add-AzureHDInsightScriptAction** cmdlet to add a script action to cluster configuration. Later, when the cluster is being created, the script action gets executed.

        # Add a script action to the cluster configuration
        $config = Add-AzureHDInsightScriptAction -Config $config -Name "Install Spark" -ClusterRoleCollection HeadNode -Uri https://hdiconfigactions.blob.core.windows.net/sparkconfigactionv03/spark-installer-v03.ps1

    **Add-AzureHDInsightScriptAction** cmdlet takes the following parameters:

    <table style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse;">
    <tr>
    <th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:90px; padding-left:5px; padding-right:5px;">Parameter</th>
    <th style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; width:550px; padding-left:5px; padding-right:5px;">Definition</th></tr>
    <tr>
    <td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Config</td>
    <td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px; padding-right:5px;">The configuration object to which script action information is added.</td></tr>
    <tr>
    <td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Name</td>
    <td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Name of the script action.</td></tr>
    <tr>
    <td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">ClusterRoleCollection</td>
    <td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Specifies the nodes on which the customization script is run. The valid values are HeadNode (to install on the head node) or DataNode (to install on all the data nodes). You can use either or both values.</td></tr>
    <tr>
    <td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Uri</td>
    <td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Specifies the URI to the script that is executed.</td></tr>
    <tr>
    <td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Parameters</td>
    <td style="border-color: #c6c6c6; border-width: 2px; border-style: solid; border-collapse: collapse; padding-left:5px;">Parameters required by the script. The sample script used in this topic does not require any parameters, and hence you do not see this parameter in the snippet above.
    </td></tr>
    </table>
4. Finally, start creating a customized cluster with Spark installed.  

        # Start creating a cluster with Spark installed
        New-AzureHDInsightCluster -Config $config -Name $clusterName -Location $location -Version $version

When prompted, enter the credentials for the cluster. It can take several minutes before the cluster is created.

## Install Spark using PowerShell
See [Customize HDInsight clusters using Script Action](hdinsight-hadoop-customize-cluster.md#call-scripts-using-azure-powershell).

## Install Spark using .NET SDK
See [Customize HDInsight clusters using Script Action](hdinsight-hadoop-customize-cluster.md#call-scripts-using-azure-powershell).

## See also
* [Create Hadoop clusters in HDInsight](hdinsight-provision-clusters.md): create HDInsight clusters.
* [Get Started with Apache Spark on HDInsight](hdinsight-apache-spark-jupyter-spark-sql.md): get started with Spark on HDInsight.
* [Customize HDInsight cluster using Script Action][hdinsight-cluster-customize]: customize HDInsight clusters using Script Action.
* [Develop Script Action scripts for HDInsight](hdinsight-hadoop-script-actions.md): develop Script Action scripts.
* [Install R on HDInsight clusters][hdinsight-install-r] provides instructions on how to use cluster customization to install and use R on HDInsight Hadoop clusters. R is an open-source language and environment for statistical computing. It provides hundreds of built-in statistical functions and its own programming language that combines aspects of functional and object-oriented programming. It also provides extensive graphical capabilities.
* [Install Giraph on HDInsight clusters](hdinsight-hadoop-giraph-install.md). Use cluster customization to install Giraph on HDInsight Hadoop clusters. Giraph allows you to perform graph processing by using Hadoop, and can be used with Azure HDInsight.
* [Install Solr on HDInsight clusters](hdinsight-hadoop-solr-install.md). Use cluster customization to install Solr on HDInsight Hadoop clusters. Solr allows you to perform powerful search operations on data stored.

[hdinsight-provision]: hdinsight-provision-clusters.md
[hdinsight-install-r]: hdinsight-hadoop-r-scripts.md
[hdinsight-cluster-customize]: hdinsight-hadoop-customize-cluster.md
[powershell-install-configure]: powershell-install-configure.md
