# Search by `diaObjectId`

!!! info "List of arguments"
    The list of arguments for running a search by `diaObjectId` can be found at [https://api.lsst.fink-portal.org :lucide-external-link:](https://api.lsst.fink-portal.org){target="blank_"}. The schema of the returned payload can be found on the [schema page :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"} and you can also retrieve it [programmatically](definitions.md).

Every time a new alert is emitted, a new `diaSource` is created. LSST makes an association (1'' matching radius) with a catalog of known Solar system objects (SSO) from the MPC prior to sending alerts. If the alert is not matched to a known SSO, it is associated to an existing `diaObject` pointing to this direction on the sky, and a `diaObjectId` is assigned to it. This page describes how to retrieve all `diaSources` information associated to the same `diaObjectId`. For alerts matched to a Solar System objects, see [Search by Solar System object name](solar_system.md).

!!! warning "`/api/v1/sources` vs `/api/v1/objects`"
    The endpoint `/api/v1/sources` gives access to alert data for an object (`diaSources`, incl. object lightcurves), while `/api/v1/objects` gives access to summary information for an object (`diaObject`). If you are coming from the ZTF API, see [ZTF to LSST migration](../../data/ztf_to_lsst.md#api-differences).

## Alert data

Alerts emitted by the same astronomical object share the same `diaObjectId` identifier. This identifier is a long integer (64-bit integer). The main table for static objects in Fink database is indexed against this identifier, and you can efficiently query all alerts from the same `diaObjectId`:

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    # get alert data for 169830579938263176
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/sources",
      json={
        "diaObjectId": "169830579938263176",
        "columns": "r:diaSourceId,r:midpointMjdTai,r:psfFlux,r:psfFluxErr",
        "output-format": "json"
      }
    )

    # Format output in a DataFrame
    if r.status_code == 200:
      pdf = pd.read_json(io.BytesIO(r.content))
    ```

=== "curl"

    ```bash
    # Get alert data for 169830579938263176 and save it in a CSV file
    curl -H "Content-Type: application/json" -X POST \
        -d '{"diaObjectId":"169830579938263176", "output-format":"csv"}' \
        https://api.lsst.fink-portal.org/api/v1/sources -o 169830579938263176.csv
    ```

=== "wget"

    ```bash
    # you can also specify parameters in the URL, e.g. with wget:
    wget "https://api.lsst.fink-portal.org/api/v1/sources?diaObjectId=169830579938263176&output-format=json" -O 169830579938263176.json
    ```

=== "Query URL"

    Paste this query on your browser to see results:
    ```
    https://lsst.fink-portal.org/169830579938263176
    ```

You can also retrieve the data for several objects at once:

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    # ID as string
    mylist = ["169830579938263176", "169342390932865250"]

    # get alert data for many objects
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/sources",
      json={
        "objectId": ",".join(mylist),
        "columns": "r:diaSourceId,r:midpointMjdTai,r:psfFlux,r:psfFluxErr",
        "output-format": "json"
      }
    )

    # Format output in a DataFrame
    pdf = pd.read_json(io.BytesIO(r.content))
    ```

!!! warning "Do not abuse!"
    Although the REST API gives you access to hundreds of millions of alerts without account, it is not designed to massively download data. If you have hundreds of objects to query, you probably want to select only a small subset of columns, or you can use the [Data Transfer service](../data_transfer.md).

You can also choose different output format:

=== "json"

    ```python
    import io
    import requests
    import pandas as pd

    # get alert data for 169830579938263176
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/sources",
      json={
        "diaObjectId": "169830579938263176",
        "columns": "r:diaSourceId,r:midpointMjdTai,r:psfFlux,r:psfFluxErr",
        "output-format": "json"
      }
    )

    # Format output in a DataFrame
    pdf = pd.read_json(io.BytesIO(r.content))
    ```

=== "csv"


    ```python
    import io
    import requests
    import pandas as pd

    # get alert data for 169830579938263176
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/sources",
      json={
        "diaObjectId": "169830579938263176",
        "columns": "r:diaSourceId,r:midpointMjdTai,r:psfFlux,r:psfFluxErr",
        "output-format": "csv"
      }
    )

    # Format output in a DataFrame
    pd.read_csv(io.BytesIO(r.content))
    ```

=== "Parquet"

    ```python
    import io
    import requests
    import pandas as pd

    # get alert data for 169830579938263176
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/sources",
      json={
        "diaObjectId": "169830579938263176",
        "columns": "r:diaSourceId,r:midpointMjdTai,r:psfFlux,r:psfFluxErr",
        "output-format": "parquet"
      }
    )

    # Format output in a DataFrame
    pdf = pd.read_parquet(io.BytesIO(r.content))
    ```

=== "votable"

    ```python
    import io
    import requests
    from astropy.io import votable

    # get alert data for 169830579938263176
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/sources",
      json={
        "diaObjectId": "169830579938263176",
        "columns": "r:diaSourceId,r:midpointMjdTai,r:psfFlux,r:psfFluxErr",
        "output-format": "votable"
      }
    )

    # VO table
    vt = votable.parse(io.BytesIO(r.content))
    ```

## Object data

The endpoint `/api/v1/objects` give access to summary information about an object, such as the number of alerts for the object, the mean flux per band, etc. You would simply use:

=== "Python"

    ```python
    import io
    import requests

    # get summary data for 169830579938263176
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/objects",
      json={
        "diaObjectId": "169830579938263176",
        "output-format": "json"
      }
    )

    if r.status_code == 200:
      # dictionary with object properties
      properties = r.json()[0]
    ```


<!--



!!! tip "Optimisation: selecting a subset of columns"
    By default, we transfer all available data fields (original ZTF fields and Fink science module outputs). But you can also choose to transfer only a subset of the fields (see [https://api.ztf.fink-portal.org/api/v1/schema](https://api.ztf.fink-portal.org/api/v1/schema) for the list of available fields):

    ```python
    # select only jd, magpsf and sigmapsf
    r = requests.post(
        "https://api.ztf.fink-portal.org/api/v1/objects",
        json={
            "objectId": "ZTF21aaxtctv",
            "columns": "i:jd,i:magpsf,i:sigmapsf" # (1)!
        }
    )
    ```

    1. Note that the fields should be comma-separated. Unknown field names are ignored. You cannot select fields starting with `v:` because they are created on-the-fly on the server at runtime and not stored in the database.

    This way, the transfer will be much faster than querying everything!

## Upper limits and bad quality data

You can also retrieve upper limits and bad quality data (as defined by Fink quality cuts)
alongside valid measurements. For this you would use the argument `withupperlim`.

!!! tip "Checking data quality"
    Note that the returned data will contained a new column, `d:tag`, to easily check data type:
    `valid` (valid alert measurements), `upperlim` (upper limits), `badquality` (alert measurements that did not pass quality cuts).

```python
import io
import requests
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

sns.set_context("talk")

# get data for ZTF21aaxtctv
r = requests.post(
    "https://api.ztf.fink-portal.org/api/v1/objects",
    json={"objectId": "ZTF21aaxtctv", "withupperlim": "True"},
)

# Format output in a DataFrame
pdf = pd.read_json(io.BytesIO(r.content))

fig = plt.figure(figsize=(15, 6))

colordic = {1: "C0", 2: "C1"}
namedic = {1: "g", 2: "r"}

for filt in [1, 2]:
    mask_filt = pdf["i:fid"] == filt

    # The column `d:tag` is used to check data type
    mask_valid = pdf["d:tag"] == "valid"
    plt.errorbar(
        pdf[mask_valid & mask_filt]["i:jd"].apply(lambda x: x - 2400000.5),
        pdf[mask_valid & mask_filt]["i:magpsf"],
        pdf[mask_valid & mask_filt]["i:sigmapsf"],
        ls="",
        marker="o",
        color=colordic[filt],
        label="{} band".format(namedic[filt]),
    )

    mask_upper = pdf["d:tag"] == "upperlim"
    plt.plot(
        pdf[mask_upper & mask_filt]["i:jd"].apply(lambda x: x - 2400000.5),
        pdf[mask_upper & mask_filt]["i:diffmaglim"],
        ls="",
        marker="^",
        color=colordic[filt],
        markerfacecolor="none",
        label="{} band (upper limit)".format(namedic[filt]),
    )

    maskBadquality = pdf["d:tag"] == "badquality"
    plt.errorbar(
        pdf[maskBadquality & mask_filt]["i:jd"].apply(lambda x: x - 2400000.5),
        pdf[maskBadquality & mask_filt]["i:magpsf"],
        pdf[maskBadquality & mask_filt]["i:sigmapsf"],
        ls="",
        marker="v",
        color=colordic[filt],
        label="{} band (bad quality)".format(namedic[filt]),
    )

plt.gca().invert_yaxis()
plt.xlabel("Modified Julian Date")
plt.ylabel("Magnitude")
plt.legend()
plt.show()
```

![sn_example](https://user-images.githubusercontent.com/20426972/113519225-2ba29480-958b-11eb-9452-15e84f0e5efc.png) -->
