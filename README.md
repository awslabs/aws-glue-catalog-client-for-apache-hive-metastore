## AWS Glue Data Catalog Client for Apache Hive Metastore
The AWS Glue Data Catalog is a fully managed, Apache Hive Metastore compatible, metadata repository. Customers can use the Data Catalog as a central repository to store structural and operational metadata for their data.

AWS Glue provides out-of-box integration with Amazon EMR that enables customers to use the AWS Glue Data Catalog as an external Hive Metastore. To learn more, visit our [documentation](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-hive-metastore-glue.html).

This is an open-source implementation of the Apache Hive Metastore client on Amazon EMR clusters that uses the AWS Glue Data Catalog as an external Hive Metastore. It serves as a reference implementation for building a Hive Metastore-compatible client that connects to the AWS Glue Data Catalog. It may be ported to other Hive Metastore-compatible platforms such as other Hadoop and Apache Spark distributions.

**Note**: in order for this client implementation to be used with Apache Hive, a patch included in this [JIRA](https://issues.apache.org/jira/browse/HIVE-12679) must be applied to it. All versions of Apache Hive running on Amazon EMR that support the AWS Glue Data Catalog as the metastore already include this patch.

## Patching Apache Hive and Installing It Locally

Obtain a copy of Hive from GitHub at https://github.com/apache/hive.  

	git clone https://github.com/apache/hive.git

To build the Hive client, you need to first apply this [patch](https://issues.apache.org/jira/secure/attachment/12958418/HIVE-12679.branch-2.3.patch).  Download this patch and move it to your local Hive git repository you created above.  Apply the patch and build Hive.

	git checkout tags/rel/release-2.3.4 -b rel-2.3.4
	patch -p0 <HIVE-12679.branch-2.3.patch
	mvn clean install -DskipTests

Note that the `IMetaStore` API changed in version 2.3.6-SNAPSHOT that is not compatible with the client.

## Building the Hive Client

Once you have successfully patched and installed Hive locally, move into the AWS Glue Data Catalog Client repository and update the following property in pom.xml.

	<hive2.version>2.3.4-SNAPSHOT</hive2.version>

You are now ready to build the Hive client.

	cd aws-glue-datacatalog-hive2-client
	mvn clean package -DskipTests

## Building the Spark Client

As Spark uses a fork of Hive based off the 1.2.1 branch, in order to build the Spark client, you need Hive 1.2 built with this [patch](https://issues.apache.org/jira/secure/attachment/12958417/HIVE-12679.branch-1.2.patch).  Unlike Hive 2.x, Hive 1.x must be built with a Maven profile set to either "hadoop-1" or "hadoop-2".

	cd <your local Hive repo>
	git checkout branch-1.2
	patch -p0 <HIVE-12679.branch-1.2.patch
	mvn clean install -DskipTests -Phadoop-2

Go back to the AWS Glue Data Catalog Client repository and update the following property in pom.xml to match the version of Hive you just patched and installed locally.  Presently, the latest version in the 1.2 branch (branch-1.2) is "1.2.3-SNAPSHOT".

	<spark-hive.version>1.2.3-SNAPSHOT</spark-hive.version>

You've built the Hive 1.2.x branch under the `org.apache.hive` folder of your local maven repository. Next create a copy of these jars to the directory where maven expects to find the files.

	mkdir -p ~/.m2/repository/org/spark-project/hive/hive-metastore
	mkdir -p ~/.m2/repository/org/spark-project/hive/hive-exec


	cp -R ~/.m2/repository/org/apache/hive/hive-metastore/1.2.3-SNAPSHOT ~/.m2/repository/org/spark-project/hive/hive-metastore/
	cp -R ~/.m2/repository/org/apache/hive/hive-exec/1.2.3-SNAPSHOT ~/.m2/repository/org/spark-project/hive/hive-exec/

You are now ready to build the Spark client.

	cd aws-glue-datacatalog-spark-client
	mvn clean package -DskipTests

If you have both versions of Hive patched and installed locally, you can build both of these clients from the root directory of the AWS Glue Data Catalog Client repository.

## Configuring Hive to Use the Hive Client

You need to ensure that the AWS Glue Data Catalog Client jar is in Hive's CLASSPATH and also set the "hive.metastore.client.factory.class" HiveConf variable for Hive to pick up and instantiate the AWS Glue Data Catalog Client.  For instance, on Amazon EMR, the client jar is located in /usr/lib/hive/lib/ and the HiveConf is set in /usr/lib/hive/conf/hive-site.xml.

	<property>
 		<name>hive.metastore.client.factory.class</name>
 		<value>com.amazonaws.glue.catalog.metastore.AWSGlueDataCatalogHiveClientFactory</value>
	</property>

## Configuring Spark to Use the Spark Client

Similarly, for Spark, you need to install the client jar in Spark's CLASSPATH and create or update Spark's own hive-site.xml to add the above property.  On Amazon EMR, this is set in /usr/lib/spark/conf/hive-site.xml.  You can also find the location of the Spark client jar in /usr/lib/spark/conf/spark-defaults.conf.

## License

This library is licensed under the Apache 2.0 License. 
