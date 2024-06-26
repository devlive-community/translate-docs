[TOC]

Users can import your test or online clusters through the catalog management function provided by the AMS Dashboard. Before adding a new Catalog,
please read the following guidelines and select the appropriate creation according to your actual needs.

## Create catalog
In Amoro, the catalog is a namespace for a group of libraries and tables. Under the catalog, it is further divided into different databases, and under each database, there are different tables. The name of a table in Amoro is uniquely identified by the format `catalog.database.table`. In practical applications, a catalog generally corresponds to a metadata service, such as the commonly used Hive Metastore in big data.

AMS can also serve as a metadata service. In order to differentiate the storage method of metadata, Amoro classifies the catalog type into `Internal Catalog` and `External Catalog`. Catalogs that use AMS as the metadata service are internal catalogs, while others are external catalogs. When creating an external catalog, you need to select the storage backend for its metadata, such as Hive, Hadoop, or Custom.

In addition, when defining a catalog, you also need to select the table format used under it. Currently, Amoro supports the following table formats:
[Iceberg]($Iceberg) 、[Mixed-Hive]($Mixed-Hive)、[Mixed-Iceberg]($Mixed-Iceberg).

You can create a catalog in the AMS frontend:
![create_catalog](https://amoro.incubator.apache.org/docs/0.6.1/images/admin/create-catalog.png)

### Configure basic information

- name: catalog name, only numbers, letters, _, - , starting with letters are supported (lower case letters are recommended)
- type: Internal Catalog or External Catalog
- metastore: storage type for table metadata. Hive Metastore (for using HMS to store metadata), Hadoop (corresponding to iceberg's Hadoop catalog), Glue (for using AWS Glue to store metadata), Custom (other iceberg catalog implementations).
- table format: Iceberg 、Mixed-Hive、Mixed  Iceberg. Currently, only Hive Metastore supports both Mixed-Hive and Iceberg format table, only internal catalog support both Mixed-Iceberg and Iceberg format table. All other metastore types only support Iceberg, currently a Metastore only supports one kind of TableFormat, in the future will support multiple
- optimizer group: tables under the catalog will automatically perform self-optimizing within this group.

### Configure storage
- Type: Hadoop or S3
- core-site: the core-site.xml of the hadoop cluster
- hdfs-site: the hdfs-site.xml of the hadoop cluster
- hive-site: the hive-site.xml for Hive
- Region: region of the S3 bucket
- Endpoint: endpoint of the S3 bucket

### Configure authentication
- Type: SIMPLE, KERBEROS, AK/SK or CUSTOM
- hadoop username: username of the hadoop cluster
- keytab: keytab file
- principal: principal of keytab
- krb5: Kerberos krb5.conf configuration file
- Access Key: Access Key for S3
- Secret Key: Secret Access Key for S3

### Configure properties
Common properties include:
- warehouse: Warehouse **must be configured** for ams/hadoop/glue catalog, as it determines where our database and table files should be placed
- catalog-impl: when the metastore is **Custom**, an additional catalog-impl must be defined, and the user must put the jar package for the custom catalog implementation into the **{AMORO_HOME}/lib** directory, **and the service must be restarted to take effect**
- table-filter: Configure a regular expression to filter tables in the catalog. The matching will be done in the format of `database.table`. For example, if it is set to `(A\.a)|(B\.b)`, it will ignore all tables except for table `a` in database `A` and table `b` in database `B`

### Configure table properties
If you want to add the same table properties to all tables under a catalog, you can add these table properties here on the catalog level. If you also configure this property on the table level, the property on the table will take effect.

We recommend users to create a Catalog following the guidelines below：

- If you want to use it in conjunction with HMS, choose `External Catalog` for the `Type` and `Hive Metastore` for the `Metastore`, and choose the table format based on your needs, Mixed-Hive or Iceberg.
- If you want to use Mixed-Iceberg provided by Amoro, choose `Internal Catalog` for the `Type` and `Mixed-Iceberg` for the table format.

## Delete catalog
When a user needs to delete a Catalog, they can go to the details page of the Catalog and click the Remove button at the bottom of the page to perform the deletion.

> Before deleting a Catalog, AMS will verify whether there is metadata for tables under that Catalog.
> If there are still tables under that Catalog, AMS will prompt that the deletion failed.
