# Fink Data Transfer

_date 24/02/2026_

This manual has been tested for `fink-client` version 10.0. In case of trouble, send us an email (contact@fink-broker.org) or [open an issue :lucide-external-link:](https://github.com/astrolabsoftware/fink-client/issues){target="blank_"}.

## Purpose

The Data Transfer service allows users to explore and transfer historical data at scale: [https://lsst.fink-portal.org/download :lucide-external-link:](https://lsst.fink-portal.org/download){target="blank_"}. This service lets users to select any observing nights from LSST, define the content of the output, and stream data directly to anywhere!

In Fink you used so far two main services to interact with the alert data:

1. Fink Livestream: based on Apache Kafka, to receive alerts in real-time based on user-defined filters.
2. Fink Science Portal: web application (and REST API) to access and display all processed data.

The first service enables data transfer at scale, but you cannot request alerts from the past. The second service lets you query data from the beginning of the project, but the volume of data to transfer for each query is limited. **Hence, we were missing a service that would enable massive data transfer for historical data.**

This third service is mainly made for users who want to access a lot of alerts for:

1. building data sets,
2. training machine/deep learning models,
3. performing massive or exotic analyses on processed alert data provided by Fink.

We decided to _stream_ the output of each job. In practice, this means that the output alerts will be send to the Fink Apache Kafka cluster, and a stream containing the alerts will be produced. You will then act as a consumer of this stream, and you will be able to poll alerts knowing the topic name. This has many advantages compared to traditional techniques:

1. Data is available as soon as there is one alert pushed in the topic.
2. The user can start and stop polling whenever, resuming the poll later (alerts are in a queue).
3. The user can poll the stream many times, hence easily saving data in multiple machines.
4. The user can share the topic name with anyone, hence easily sharing data.
5. The user can decide to not poll all alerts.


## Installation of fink-client

To ease the consuming step, the users are recommended to use the [fink-client :lucide-external-link:](https://github.com/astrolabsoftware/fink-client){target="blank_"}, which is a wrapper around Apache Kafka. `fink_client` requires a version of Python 3.9+. Documentation to install the client can be found at [services/fink_client](../developers/fink_client.md). Note that you need to be registered in order to poll data.

## Defining your query

To start the service, connect to [https://lsst.fink-portal.org/download :lucide-external-link:](https://lsst.fink-portal.org/download){target="blank_"}. The construction of the query is a guided process.
First, if you have a configuration file, you can upload it. If you do not know what it is, no worry, we will see it later. Then jump on the next page to choose the dates for which you would like to get alert data using the calendar. You can choose multiple consecutive dates. Then go on the next page:

![2](../img/download_filters.png)

For LSST, you can further filter the data in three ways:

1. You can select one or more user-defined tag(s) (the Fink Filters). This is useful if you want to replay an analysis on a previous night.
2. You can select one or more user-defined block(s). Blocks are useful set of conditions.
3. Optionally, you can also impose extra conditions on the alerts you want to retrieve based on their content. You will simply specify the name of the parameter with the condition (SQL syntax). If you have several conditions, put one condition per line, ending with semi-colon. Example of valid conditions:

```sql
-- Example block 1
-- Alerts with flux above 13500 nJy (< mag 21) and
-- at least 3 detections
diaSource.psfFlux > 13500;
diaObject.nDiaSources > 3;

-- Example block 2: Filter on magnitude and specific band
diaSource.band = 'g';
31.4 - 2.5 * LOG10(diaSource.scienceFlux) < 21;

-- Example block 3: Using a combination of fields (magnitude difference between science and template)
2.5 * LOG10(diaSource.psfFlux / diaSource.templateFlux) > 0.5;

-- Example block 3: Filtering on ML scores
clf.snnSnVsOthers_score > 0.5;

-- Example block 4: Filtering on catalog output
xm.tns_type IN ('SN Ia', 'SN II');

-- Example block 5: Only classified objects in SIMBAD and Gaia DR3
pred.is_cataloged;
```

Note that when you start typing a nested field in the schema, a dropdown menu will appear with available fields. See the [schema page :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"} for more information about available tags, block, and fields.

Finally you can choose the content of the alerts to be returned. You have four types of content:
1. Light packet: lightweight (a few KB/alerts), this option transfers only necessary fields for working with lightcurves plus all Fink added values. Prefer this option to start.
2. Medium packet: original LSST alerts plus all Fink added values, but without cutouts.
4. Full packet: original LSST alerts plus all Fink added values.
4. Any fields you want: instead of the pre-defined schema from above, you can also choose to download only the fields of interest for you. Prefer this option if you know what you want (and this will reduce greatly the volume of data to transfer).

Alert schema can be again accessed directly from the [schema page :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"}. Note that you can apply filters (e.g. tags, extra conditions, etc.) on any alert fields regardless of the final alert content as the filtering is done prior to the output schema. Once you have filled all parameters, go to the last iteration, and review all parameters before hitting the submission button:

![1](../img/download_final.png)

The number of alert estimation is only the number of alerts between the chosen dates, and it does not take into account the number of alerts per night, the number of alerts filtered per tags and blocks, and the extra conditions (that could further reduce the number of alerts). However we provide an estimation of the size of the data to be transfered based on the content. You can update your parameters if need be, the estimations will be updated in real-time.

Before submission, download your configuration file. Next time you launch a job, you will be able to upload it to retrieve your parameters!

![1](../img/configuration_data_transfer.png)

After submission, your job will be triggered on the Fink Apache Spark cluster, and a topic name will be generated. Keep this topic name with you, you will need it to get the data. Details about the progress will be automatically displayed on the page.

## Consuming the alert data

### Standard

On the submission web page, when you read the message:

```bash
26/02/24 21:35:14 -Livy- Starting to send data to topic ftransfer_lsst_2026-02-24_34995
```

this means you can already start polling the data on your computer. You will then invoke for example (see the command on the right panel):

```bash
# requires fink-client>=10.0
fink_datatransfer \
    -survey lsst \
    -topic ftransfer_lsst_2026-02-24_34995 \
    -outdir ftransfer_lsst_2026-02-24_34995 \
    --verbose
```

Alert data will be consumed and stored on disk as parquet files. Because LSST is not filling all fields, if you try to read using Pandas directly, you will likely have a typing error. Instead use the provided function from the fink-client package:

```python
from fink_client.visualisation import read_parquet

# read alerts in a DataFrame
pdf = read_parquet("ftransfer_lsst_2026-02-24_34995")

          diaObjectId         snr  ...                   timestamp  tns_type_recomputed
0  313761043604045880  277.624939  ...  2026-02-19 01:34:46.279993              Unknown
1  313761043604045880  281.893402  ...  2026-02-19 01:47:20.428791              Unknown

[2 rows x 29 columns]
```

You can stop the poll by hitting `CTRL+C` on your keyboard, and resume later. The poll will restart from the last offset, namely you will not have duplicate. In case you want to start polling data from the beginning of the stream, you can use the `--restart_from_beginning` option:

```bash
# Make sure <output directory> is empty or does not
# exist to avoid duplicates.
fink_datatransfer \
    -topic <topic name> \
    -survey lsst \
    -outdir <output directory> \
    --verbose \
    --restart_from_beginning
```

Finally you can inspect the schema of the alerts using the option `--dump_schema`:


```bash
# Make sure <output directory> is empty or does not
# exist to avoid duplicates.
fink_datatransfer \
    -topic <topic name> \
    -survey lsst \
    -outdir <output directory> \
    --verbose \
    --dump_schema
```

The option will produce a json file on disk whose name is `schema_<topic name>.json`. Schema can be inspected using e.g.:

```bash
cat filename.json | jq
```

### Multiprocessing

From `fink-client` version 7.0, we have introduced the functionality of simultaneous downloading from multiple partitions through the implementation of multi-processing technology, which is an approach that takes advantage of modern hardware resources to run multiple tasks in parallel.
By using this strategy, the service is able to simultaneously access different partitions of the data stored in the Kafka server, enabling faster and more efficient transfer. The benefits of this approach are numerous, ranging from optimizing transfer times to making more efficient use of available hardware resources.

By default, the client will use all available logical CPUs. You can also specify the number of CPUs to use, as well as the batch size from the command line:

```bash
fink_datatransfer \
    -topic <topic name> \
    -outdir <output directory> \
    -survey lsst \
    -nconsumers 5 \
    -batchsize 1000 \
    --verbose
```

More details on the expected performances are given in this [post :lucide-external-link:](https://fink-broker.org/news/2023-01-17-data-transfer/).

## How is this done in practice?

![1](../img/datatransfer_archi.png)

_(1) the user connects to the service and request a transfer by filling fields and hitting the submit button. (2) Dash callbacks build and upload the execution script to HDFS, and submit a job in the Fink [Apache Spark :lucide-external-link:](https://spark.apache.org/) cluster using the [Livy :lucide-external-link:](https://livy.apache.org/) service. (3) Necessary data is loaded from the distributed storage system containing Fink data and processed by the Spark cluster. (4) The resulting alerts are published to the Fink [Apache Kafka :lucide-external-link:](https://kafka.apache.org/) cluster, and they are available up to 7 days by the user. (5) The user can retrieve the data using e.g. the Fink client, or any Kafka-based tool._

## Troubleshooting

In case of trouble, send us an email (contact@fink-broker.org) or [open an issue :lucide-external-link:](https://github.com/astrolabsoftware/fink-client).

### Timeout error

If you get frequent timeouts while you know there are alerts to poll (especially if you are polling from outside of Europe where the servers are), try to increase the timeout (in seconds) in your configuration file:

```bash
# edit ~/.finkclient/credentials.yml
maxtimeout: 30
```

### UNKNOWN_TOPIC_OR_PART

If you launch the download too early, the queue is not yet filled and you will likely get this error:

```bash
cimpl.KafkaException: KafkaError{code=UNKNOWN_TOPIC_OR_PART,val=3,str="Broker: Unknown topic or partition"}
```

Wait a minute, and retry. If the error persists, contact Julien.

### Known fink-client bugs

1. With version 4.0, you wouldn't have the partitioning column when reading in a dataframe. This has been corrected in 4.1.
2. With version 7.0, files were overwritten because they were sharing the same names, hence leading to fewer alerts than expected. This has been corrected in 7.1.
3. With version prior to 9, you could not partition by time.

## Final note

You might experience some service interruptions, or inefficiencies. This service is highly demanding in resources for large jobs (there is TBs of data behind the scene!), so we keep monitoring the load of the clusters and optimizing the current design.
