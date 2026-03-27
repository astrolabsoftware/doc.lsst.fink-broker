# Livestream

_date 24/02/2026_

This manual has been tested for `fink-client` version 10.0. Other versions might work. In case of trouble, send us an email (contact@fink-broker.org) or [open an issue :lucide-external-link:](https://github.com/astrolabsoftware/fink-client/issues){target="blank_"}.

!!! info "From ZTF to LSST"
    ZTF users need to migrate their fink-client to version 10.0, and authenticate again.


## Purpose

The livestream service is based on the Fink filters. After each exposure, Fink processes the alerts sent by LSST and the filters select alerts to be transmitted based on their content. These alerts are sent to the Fink [Apache Kafka :lucide-external-link:](https://kafka.apache.org/){target="blank_"} cluster, and substreams are produced (1 filter = 1 substream), identified by their _topic_ name. Each alert pushed is available for 7 days in the queue, and consumers can replay streams indefinitely.

As Kafka can be somehow cumbersome, we developed a client to facilitate the stream consuming part for Fink users: [fink-client :lucide-external-link:](https://github.com/astrolabsoftware/fink-client){target="blank_"}. Users can connect to one or more topics, and new topics can be created via new Fink filters.

## Installation of fink-client

To ease the consuming step, the users are recommended to use the [fink-client :lucide-external-link:](https://github.com/astrolabsoftware/fink-client){target="blank_"}, which is a wrapper around Apache Kafka. `fink_client` requires a version of Python 3.9+. Documentation to install the client can be found at [https://github.com/astrolabsoftware/fink-client :lucide-external-link:](https://github.com/astrolabsoftware/fink-client){target="blank_"}. Note that you need to be registered in order to poll data.

For the list of available topics (aka tags), see the schema page [https://lsst.fink-portal.org/schemas :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"}.

## First steps: testing the connection

Processed alerts are stored 4 days on our servers, which means if you forget to poll data, you'll be able to retrieve it up to 4 days after emission. This also means on your first connection, you will have 4 days of alert to retrieve. Before you get all of them, let's retrieve the first available alert to check the connection. On a terminal, run the following

```bash
# access help using `fink_consumer -h`
fink_consumer -survey lsst --display -limit 1
```

This will download the first available alert, and print some useful information.  The alert schema is automatically downloaded from the GitHub repo (see the Troubleshooting section if that command does not work). Then the alert is consumed and you'll move to the next alert. Of course, if you want to keep the data, you need to store it. This can be easily done:

```bash
# create a folder to store alerts
mkdir alertDB

# access help using `fink_consumer -h`
fink_consumer -survey lsst --display --save -outdir alertDB -limit 1
```

This will download the next available alert, display some useful information on screen, and save it (Apache Avro format) on disk. Then if all works, then you can remove the limit, and let the consumer run for ever!

```bash
# access help using `fink_consumer -h`
fink_consumer -survey lsst --display --save -outdir alertDB
```

## Inspecting alerts

Once alerts are saved, you can open it and explore the content. We wrote a small utility to quickly visualise it:

```bash
# access help using `fink_alert_viewer -h`
fink_alert_viewer -survey lsst -f alertDB/313998539674877976.avro
```

of course, you can develop your own tools based on this one! Note Apache Avro is not something supported by default in Pandas for example, so we provide a small utilities to load alerts more easily:

```python
from fink_client.avro_utils import AlertReader

# you can also specify one folder with several alerts directly
r = AlertReader("alertDB/313998539674877976.avro")

# convert alert to Pandas DataFrame
r.to_pandas()
```

## Managing offsets

### Checking offsets

You might want to check where you are on the different queues, that is retrieving the offsets for each topic that you are polling:

```bash
fink_consumer -survey lsst --display_statistics

Topic [Partition]                                   Committed        Lag
========================================================================
fink_sn_near_galaxy_candidate_lsst [0]                     19      27821
fink_sn_near_galaxy_candidate_lsst [1]                     25      27931
fink_sn_near_galaxy_candidate_lsst [2]                     24      28180
fink_sn_near_galaxy_candidate_lsst [3]                     15      28362
fink_sn_near_galaxy_candidate_lsst [4]                     25      28219
fink_sn_near_galaxy_candidate_lsst [5]                     20      27958
fink_sn_near_galaxy_candidate_lsst [6]                     14      27833
fink_sn_near_galaxy_candidate_lsst [7]                     23      28158
fink_sn_near_galaxy_candidate_lsst [8]                     16      28166
fink_sn_near_galaxy_candidate_lsst [9]                     14      28221
------------------------------------------------------------------------
Total for fink_sn_near_galaxy_candidate_lsst              195     280849
------------------------------------------------------------------------
```

In this example, I have one topic `fink_sn_near_galaxy_candidate_lsst`. Polled alert are in the `Committed` column, remaining alerts in the queue in the `Lag` column.

### Resetting offsets

Sometimes you might want to poll again alerts, that is restarting to poll from the beginning of a queue. For this, you can use:

```bash
fink_consumer -survey lsst --display -start_at earliest
Resetting offsets to BEGINNING
...
assign TopicPartition{topic=fink_sn_near_galaxy_candidate_lsst,partition=0,offset=0,leader_epoch=None,error=None}
...
# poll restarts at the first offset
```

All your topic partitions will be reset to the starting offset (`0` in this case). Similarly, you can empty all topics, and restarting polling from the last offset:

```bash
fink_consumer --display -start_at latest
...
assign TopicPartition{topic=fink_sn_near_galaxy_candidate_lsst,partition=0,offset=0,leader_epoch=None,error=None}
...
assign TopicPartition{topic=fink_sn_near_galaxy_candidate_lsst,partition=4,offset=2,leader_epoch=None,error=None}
...
No alerts the last 10 seconds
...
```

Empty partitions will have `offset=0`, but others will have their offset to the latest one. The client will then wait for new data to come. Note that the reset will be actually triggered on the next poll. Hence the command `fink_consumer --display_statistics` will not right away display the reset offsets. This is particularly useful after a bug in the topic (malformed alerts pushed), and you want a fresh restart.

## Write your own consumer

You can write your own consumer to manipulate alerts upon receival by taking inspiration from the [fink_consumer.py :lucide-external-link:](https://github.com/astrolabsoftware/fink-client/blob/master/fink_client/scripts/fink_consumer.py){target="blank_"} script.

## Troubleshooting

In case of trouble, send us an email (contact@fink-broker.org) or open an issue [https://github.com/astrolabsoftware/fink-client :lucide-external-link:](https://github.com/astrolabsoftware/fink-client){target="blank_"}.

### Wrong schema

A typical error though would be:

```
Traceback (most recent call last):
  File "fastavro/_read.pyx", line 835, in fastavro._read.schemaless_reader
  File "fastavro/_read.pyx", line 846, in fastavro._read.schemaless_reader
  File "fastavro/_read.pyx", line 561, in fastavro._read._read_data
  File "fastavro/_read.pyx", line 456, in fastavro._read.read_record
  File "fastavro/_read.pyx", line 559, in fastavro._read._read_data
  File "fastavro/_read.pyx", line 431, in fastavro._read.read_union
  File "fastavro/_read.pyx", line 555, in fastavro._read._read_data
  File "fastavro/_read.pyx", line 349, in fastavro._read.read_array
  File "fastavro/_read.pyx", line 561, in fastavro._read._read_data
  File "fastavro/_read.pyx", line 456, in fastavro._read.read_record
  File "fastavro/_read.pyx", line 559, in fastavro._read._read_data
  File "fastavro/_read.pyx", line 405, in fastavro._read.read_union
IndexError: list index out of range
```

This error happens when the schema to decode the alert is not matching the alert content. Usually this should not happen (schema is included in the alert payload). In case it happens though, you can force a schema:

```
fink_consumer [...] -schema [path_to_a_good_schema]
```

In case you do not have replacement schemas, you can save the current (faulty) schema that is contained within an alert packet:

```bash
fink_consumer -survey lsst -limit 1 --dump_schema
```

You will see the traceback above, with the message:

```
Schema saved as schema_2024-06-03T11:12:36.855544+00:00.json
```


Then you can inspect the schema manually, or open an issue on the fink-client repository by attaching this schema to your message.

### Authentication error

If you try to poll the servers and get:

```
%3|1634555965.502|FAIL|rdkafka#consumer-1| [thrd:sasl_plaintext://xx.xx.xx.xx:yy/bootstrap]: sasl_plaintext://xx.xx.xx.xx:yy/bootstrap: SASL SCRAM-SHA-512 mechanism handshake failed: Broker: Request not valid in current SASL state: broker's supported mechanisms:  (after 18ms in state AUTH_HANDSHAKE)
```

You are likely giving a password when instantiating the consumer. Check your `~/.finkclient/lsst_credentials.yml`, it should contain

```yml
password: null
```

or directly in your code:

```python
# myconfig is a dict that should NOT have
# a 'password' key set
consumer = AlertConsumer(mytopics, myconfig)
```

However, if you want the old behaviour, then you need to specify it using `sasl.*` parameters:

```python
myconfig["sasl.username"] = "your_username"
myconfig["sasl.password"] = None
consumer = AlertConsumer(mytopics, myconfig)
```

### Timeout error

If you get frequent timeouts while you know there are alerts to poll, try to increase the timeout (in seconds) in your configuration file:

```bash
# edit ~/.finkclient/lsst_credentials.yml
maxtimeout: 30
```
