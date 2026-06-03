# Fink client

_date 03/06/2026_

This manual has been tested for `fink-client` version 12.0. In case of trouble, send us an email (contact@fink-broker.org) or [open an issue :lucide-external-link:](https://github.com/astrolabsoftware/fink-client/issues){target="blank_"}.

## Purpose

The Fink ecosystem has evolved rapidly in recent years. Initially, in 2020, the fink-client was only a wrapper around the Kafka consumer API to simplify the work of astronomers and other users performing follow-up with the Fink/ZTF Livestream service. In 2023 the client was expanded to support the Data Transfer service, and in 2025 support for the ZTF Xmatch service was added. Neither its core nor its interface changed much during that time.

With the start of LSST, the number of client connections and the need to access additional services grew quickly. For that reason we completely redesigned the CLI and added two more services: bots (an extension of the Livestream) and search (a wrapper around the REST API). We hope you enjoy it!

## Installation of fink-client

You would simply install the latest version of the client using pip in your terminal:

```bash
pip install fink-client --upgrade
```

Check the client is correctly installed by running:

```bash
finkctl
```

You should see the help menu, together with the version of the client.

## Authentication

For some services, such as the Data Transfer and the Livestream, you must register before polling data. Please refer to the "Registration" section in the [fink-client :lucide-external-link:](https://github.com/astrolabsoftware/fink-client#registration){target="blank_"} GitHub repository. 

## Migration from version 11 to 12

If you coming from the client version 11, you will need a few changes for the client to work again.

### Authentication

We recommend that you run again the authentication as the configuration files slightly changed. For this, just remove your current configuration, and authenticate again:

```bash
# backup old configuration
cp -r ~/.finkclient ~/.finkclient_$(date +%Y%m%d)

# Remove old configuration
rm -r ~/.finkclient

# Authenticate again
finkctl auth register ...
```

In case you are unsure about the parameters to fill, you can access the documentation by running:

```bash
finkctl auth register
```

and you can also display the current configuration by running:

```bash
finkctl auth show -survey lsst
```

### Commands mapping

In version 12, the old commands have been replaced:

- `fink_consumer` --> `finkctl stream`
- `fink_datatransfer` --> `finkctl transfer`

Options to these commands remain the same!

## How-to

### How to register?

Based on the information sent when you asked for credentials, authenticate using the `auth` command:

```bash
finkctl auth register ...
```

In case you are unsure about the parameters to fill, you can access the documentation by running:

```bash
finkctl auth register
```

and you can also display the current configuration by running:

```bash
finkctl auth show -survey lsst
```

### How to get data from the Data Transfer service?

Documentation for using the Data Transfer and polling data with the fink-client is available in the [Data Transfer section](data_transfer.md).

### How to know available topics for the Livestream service?

For the list of available topics (aka tags), see the schema page [https://lsst.fink-portal.org/schemas :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"}. From version 12, you can also access this list programmatically:

```bash
finkctl topic list -survey lsst
```

### How to connect to an alert topic produced by Fink?

Documentation for polling data from the Livestream with the fink-client is available in the [Livestream section](livestream.md).

### How to create a Fink bot for my alert topic?

link to bots

### How to get information for a particular astronomical object?

example + link to API

### How to get information for a particular Solar System object?

### How to get information for a particular direction on the sky?
