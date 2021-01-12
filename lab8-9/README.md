
1. Створюємо сторедж аккаунт 

![ssh-hd-insight](../images/lab8-9_create_storage_acc.png)

2. Hierarchical namespace – Enabled. 

![ssh-hd-insight](../images/lab8-9_create_storage_acc_enabled.png)

3. Робимо новий контейнер GEN2

![ssh-hd-insight](../images/lab8-9_create_container.png)

4. Створюємо новий діректорі всередені контейнера 
![ssh-hd-insight](../images/lab8-9_create_directory.png)


6. Створюжмо та запускаємо Azure Databricks Service

![ssh-hd-insight](../images/lab8-9_create_databricks.png)

6. Створюємо  кластер в датабрікс

![ssh-hd-insight](../images/lab8-9_create_databricks_cluster.png)

8. В Azure Active Directory, переходимо в App registrations та створюємо новий.

![ssh-hd-insight](../images/lab8-9_add_registration.png)

9. Переходимо Certificates & secrets та створюємо новий client secret

![ssh-hd-insight](../images/lab8-9_create_client_secret.png)

11. В кластері Libraries –> Install New –> Maven. Coordinates: ` com.microsoft.azure:azure-eventhubs-spark_2.12:2.3.18 `

![ssh-hd-insight](../images/lab8-9_install_maven_to_cluster.png)

12. На сторінці Home вибираємо Create Notebook. Створюємо Notebook на Python.

![ssh-hd-insight](../images/lab8-9_create_notebook_py.png)

13. Сворюємо аналогічно ще один на Scala. 


14. В Data Lake потрібно налаштувати доступ до директорії. Завантажуємо Azure Storage Explorer (https://azure.microsoft.com/features/storage-explorer/) -> Вибираємо створений контейнер –> Створена папка –> Manage ACLs –> Вибираємо свою програму і даємо їй максимальні права доступу. Аналогічно даємо максимальні права доступу через контейнер.

![ssh-hd-insight](../images/lab8-9_add_access_to_storage.png)
![ssh-hd-insight](../images/lab8-9_add_access_to_storage2.png)


15. В Python Notebook вставляємо код. Відповідні поля копіюємо з Azure Active Directory -> App registration -> Створений попередньо registartion.
* Копіюємо Application (client) ID та встаялємо в ` fs.azure.account.oauth2.client.id ` 
* В Certificates & secrets копіюємо ID раніше створеного Client secret та вставляємо в ` fs.azure.account.oauth2.client.secret ` 
* В "fs.azure.account.oauth2.client.endpoint" після https://login.microsoftonline.com/ вставляємо Directory (tenant) ID з вкладки Overview.
* iotlab8 після @ заміняємо на відповідне ім'я Storage account
* iotlab8 перед @ заміняємо на відповідне ім'я Container

```
# Databricks notebook source
configs = {"fs.azure.account.auth.type": "OAuth",
         "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
         "fs.azure.account.oauth2.client.id": "db1d4b32-b569-45f2-b1eb-3b6f2bedbb4a",
         "fs.azure.account.oauth2.client.secret": "1xRCVs~.w.~17~LjrVMP1t~_4D3zNf20_4",
         "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/c5b4757e-b68d-48b7-b124-bc4a3111ec8c/oauth2/token",
         "fs.azure.createRemoteFileSystemDuringInitialization": "true"}

dbutils.fs.mount(
        source = "abfss://nosqlcontainer@nosqlstorage.dfs.core.windows.net",
        mount_point = "/mnt/labs",
        extra_configs = configs)
```        

![ssh-hd-insight](../images/lab8-9_python_config.png)

16. В Scale Notebook вставляємо код і агалогічно з попереднім пунктом заміняємо поля на актуальні і запускаємо код.
```
import org.apache.spark.eventhubs.{ ConnectionStringBuilder, EventHubsConf, EventPosition }
import org.apache.spark.sql.types._
import org.apache.spark.sql.functions._

// To connect to an Event Hub, EntityPath is required as part of the connection string.
// Here, we assume that the connection string from the Azure portal does not have the EntityPath part.
val appID = "db1d4b32-b569-45f2-b1eb-3b6f2bedbb4a"
val password = "1xRCVs~.w.~17~LjrVMP1t~_4D3zNf20_4"
val tenantID = "c5b4757e-b68d-48b7-b124-bc4a3111ec8c"
val fileSystemName = "nosqlcontainer";
var storageAccountName = "nosqlstorage";
val connectionString = ConnectionStringBuilder("Endpoint=sb://labbbb5.servicebus.windows.net/;SharedAccessKeyName=lab5-policy;SharedAccessKey=4pyrsnB+xzOwemqF0yXbk83ZSo6P/oUKmhtAnuNHdBs=;EntityPath=lab5")
  .setEventHubName("lab5")
  .build
val eventHubsConf = EventHubsConf(connectionString)
  .setStartingPosition(EventPosition.fromEndOfStream)

var dataset = 
  spark.readStream
    .format("eventhubs")
    .options(eventHubsConf.toMap)
    .load()
      
val filtered = dataset.select(
    from_unixtime(col("enqueuedTime").cast(LongType)).alias("enqueuedTime")
      , get_json_object(col("body").cast(StringType), "$.year").alias("year")
      , get_json_object(col("body").cast(StringType), "$.leading_case").alias("leading_case")
      , get_json_object(col("body").cast(StringType), "$.sex").alias("sex")
        , get_json_object(col("body").cast(StringType), "$.race").alias("race")
        , get_json_object(col("body").cast(StringType), "$.deaths").alias("deaths").cast(DoubleType)
        , get_json_object(col("body").cast(StringType), "$.death_rate").alias("death_rate").cast(DoubleType)
        , get_json_object(col("body").cast(StringType), "$.death_rate_adjusted").alias("death_rate_adjusted").cast(DoubleType)
  )
  
filtered.writeStream
  .format("com.databricks.spark.csv")
  .outputMode("append")
  .option("checkpointLocation", "/mnt/labs/lab8-9")
  .start("/mnt/labs/lab8-9")
```  

  ![ssh-hd-insight](../images/lab8-9_scala_result.png)

17. В відповідній папці мають успішно згенеруватись дані

![ssh-hd-insight](../images/lab8-9_storage_result.png)
