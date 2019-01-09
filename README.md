# pnp-databricks-monitoring

Azure Databricks is based on Apache Spark, and both use log4j as the standard library for logging. In addition to the default logging provided by Apache Spark, this reference architecture sends logs and metrics to Azure Log Analytics.  While the Apache Spark logger messages are strings, Azure Log Analytics requires log messages to be formatted as JSON. The com.microsoft.pnp.log4j.LogAnalyticsAppender class transforms these messages to JSON.

Referenced architecture: https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/data/stream-processing-databricks

<h2>Configuration:</h2>

You require the Log Analytics workspace ID and primary key. The workspace ID is the workspaceId value from the logAnalytics output section in step 4 of the deploy the Azure resources section. The primary key is the secret from the output section.

To configure log4j logging, open log4j.properties. Edit the following two values:

```
log4j.appender.A1.workspaceId=<Log Analytics workspace ID>
log4j.appender.A1.secret=<Log Analytics primary key>
```

To configure custom logging, open metrics.properties. Edit the following two values:

```
*.sink.loganalytics.workspaceId=<Log Analytics workspace ID>
*.sink.loganalytics.secret=<Log Analytics primary key>
```

<h2>Build the .jar files for the Databricks job and Databricks monitoring</h2>
Use your Java IDE to import the Maven project file named pom.xml located in the root directory. Perform a clean build. The output of this build is files named azure-databricks-monitoring-0.9.jar.

<h2>Configure custom logging for the Databricks job</h2>
Copy the azure-databricks-monitoring-0.9.jar file to the Databricks file system by entering the following command in the Databricks CLI:
```
databricks fs cp --overwrite azure-databricks-monitoring-0.9.jar dbfs:/azure-databricks-job/azure-databricks-monitoring-0.9.jar
```

Copy the custom logging properties from \azure\azure-databricks-monitoring\scripts\metrics.properties to the Databricks file system by entering the following command:
```
databricks fs cp --overwrite metrics.properties dbfs:/azure-databricks-job/metrics.properties
```

While you haven't yet decided on a name for your Databricks cluster, select one now. You'll enter the name below in the Databricks file system path for your cluster. Copy the initialization script from \azure\azure-databricks-monitoring\scripts\spark.metrics to the Databricks file system by entering the following command:
```
databricks fs cp --overwrite spark-metrics.sh dbfs:/databricks/init/<cluster-name>/spark-metrics.sh
```

<h2>Create a Databricks cluster</h2>
Below the Auto Termination dialog box, click on Init Scripts. Enter dbfs:/databricks/init//spark-metrics.sh, substituting the cluster name created