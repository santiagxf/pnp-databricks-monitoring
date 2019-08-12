# pnp-databricks-monitoring

Azure Databricks is based on Apache Spark, and both use log4j as the standard library for logging. In addition to the default logging provided by Apache Spark, this pattern and practice sends logs and metrics to Azure Log Analytics. To achieve that, we need to deploy custom handlers for the logging events. While the Apache Spark logger messages are strings, Azure Log Analytics requires log messages to be formatted as JSON. The com.microsoft.pnp.log4j.LogAnalyticsAppender class transforms these messages to JSON.

Referenced architecture: https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/data/stream-processing-databricks

<h2>Configuration:</h2>

You require the Log Analytics workspace ID and primary key. The workspace ID is the workspaceId value from the Log Analytics resource in Azure. The primary key is the secret the resource specified in order to inteact with the service.

To configure log4j logging, open log4j.properties. Edit the following two values  and save the file. We will use it later.

```
log4j.appender.A1.workspaceId=[Log Analytics workspace ID]
log4j.appender.A1.secret=[Log Analytics primary key]
```

To configure custom logging, open metrics.properties. Edit the following two values and save the file. We will use it later.

```
*.sink.loganalytics.workspaceId=[Log Analytics workspace ID]
*.sink.loganalytics.secret=[Log Analytics workspace ID]
```

<h2>Build the .jar files for the Databricks job and Databricks monitoring</h2>
We need to specified a way to convert the logs from the log4j format to the one Azure is expecting. We use a JAR module to achieve so. Use your Java IDE to import the Maven project file named pom.xml located in the root directory. Perform a clean build. The output of this build is files named azure-databricks-monitoring-0.9.jar. If you want to skip this, a prebuilt jar can be found in the built directory I created for your convenience. Version used in this case was JRE 1.8.0_191 with Maven 3.6.0.

<h2>Configure custom logging for the Databricks job</h2>
Copy the azure-databricks-monitoring-0.9.jar file to the Databricks file system by entering the following command in the Databricks CLI:

```
databricks fs cp --overwrite azure-databricks-monitoring-0.9.jar dbfs:/azure-databricks-job/azure-databricks-monitoring-0.9.jar
```

Copy the custom logging properties from metrics.properties to the Databricks file system by entering the following command:
```
databricks fs cp --overwrite metrics.properties dbfs:/azure-databricks-job/metrics.properties
```

Copy the initialization script from spark.metrics to the Databricks file system by entering the following command:
```
databricks fs cp --overwrite spark-metrics.sh dbfs:/databricks/init/[cluster-name]/spark-metrics.sh
```

<h2>Create a Databricks cluster</h2>
Below the Auto Termination dialog box, click on Init Scripts. Enter dbfs:/databricks/init/[cluster-name]/spark-metrics.sh, substituting the cluster name created