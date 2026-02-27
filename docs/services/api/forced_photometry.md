# Forced photometry

!!! info "List of arguments"
    The list of arguments for accessing forced photometry can be found at [https://api.lsst.fink-portal.org :lucide-external-link:](https://api.lsst.fink-portal.org){target="blank_"}. The schema of the returned payload can be found on the [schema page :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"} and you can also retrieve it [programmatically](definitions.md).

"Forced" photometry means a measurement made at a fixed coordinate in an image, regardless of whether an above-threshold region was detected there in that particular image. For all objects, starting from the second detection, forced photometry measurements for previous detections are included.

## Data access

Alerts emitted by the same astronomical object share the same `diaObjectId` identifier. This identifier is a long integer (64-bit integer). The main table for forced photometry measurements is indexed against this identifier, and you can efficiently query all measurement from the same `diaObjectId`:

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    # get forced data for 313761043604045880
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/fp",
      json={
        "diaObjectId": "313761043604045880",
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
    # Get forced data for 313761043604045880 and save it in a CSV file
    curl -H "Content-Type: application/json" -X POST \
        -d '{"diaObjectId":"313761043604045880", "output-format":"csv"}' \
        https://api.lsst.fink-portal.org/api/v1/fp -o fp_169830579938263176.csv
    ```

=== "wget"

    ```bash
    # you can also specify parameters in the URL, e.g. with wget:
    wget "https://api.lsst.fink-portal.org/api/v1/fp?diaObjectId=313761043604045880&output-format=json" -O fp_313761043604045880.json
    ```

You can also retrieve the data for several objects at once:

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    # ID as string
    mylist = ["313761043604045880", "313699514821640259"]

    # get forced data for many objects
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/fp",
      json={
        "diaObjectId": ",".join(mylist),
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

    # get forced data for 313761043604045880
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/fp",
      json={
        "diaObjectId": "313761043604045880",
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

    # get forced data for 313761043604045880
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/fp",
      json={
        "diaObjectId": "313761043604045880",
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

    # get forced data for 313761043604045880
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/fp",
      json={
        "diaObjectId": "313761043604045880",
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

    # get forced data for 313761043604045880
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/fp",
      json={
        "diaObjectId": "313761043604045880",
        "columns": "r:diaSourceId,r:midpointMjdTai,r:psfFlux,r:psfFluxErr",
        "output-format": "votable"
      }
    )

    # VO table
    vt = votable.parse(io.BytesIO(r.content))
    ```