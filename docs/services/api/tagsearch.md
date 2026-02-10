# Search by tag

!!! info "List of arguments"
    The list of arguments for running a search by tag can be found at [https://api.lsst.fink-portal.org :lucide-external-link:](https://api.lsst.fink-portal.org){target="blank_"}. The schema of the returned payload can be found on the [schema page :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"} and you can also retrieve it [programmatically](definitions.md).

## Default class search

!!! info "What is a class in Fink?"
    We recommend to read how the [classification scheme](../../science/classification.md) is built.

To facilitate the identification of noteworthy events, users can create filters that combine multiple alert fields, allowing them to generate meaningful tags tailored to their research needs. These tags are available to anyone, and you can easily search alerts by tag:

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    # Get latests 5 hostless candidates
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/tags",
      json={
        "tag": "hostless_candidate",
        "columns": "r:diaObjectId",
        "n": "5"  # (1)!
      }
    )

    # Format output in a DataFrame
    pdf = pd.read_json(io.BytesIO(r.content))
    ```

    1. This the maximum number of _alerts_ to retrieve. It can lead to several times the same _object_ though.

=== "curl"

    ```bash
    # Get latests 5 hostless candidates
    curl -H "Content-Type: application/json" -X POST \
        -d '{"tag":"hostless_candidate", "n":"5"}' \
        https://api.lsst.fink-portal.org/api/v1/tags -o latest_five_hostless_candidates.json
    ```
=== "wget"

    ```bash
    # you can also specify parameters in the URL, e.g. with wget:
    wget "https://api.lsst.fink-portal.org/api/v1/tags?tag=hostless_candidate&n=5&output-format=json" \
        -O latest_five_hostless_candidates.json
    ```

=== "Query URL"

    Paste this query on your browser to see results:
    ```
    https://lsst.fink-portal.org/?action=tag&tag=hostless_candidate&last=5
    ```

You can also easily list the available tags by pasting this into your browser:

``` title="Dictionary of tags and their documentation"
https://api.lsst.fink-portal.org/api/v1/tags
```

## Class search restricted in time

You can also specify `startdate` and `stopdate` for your search:

```python title="Restrict time boundaries"
import io
import requests
import pandas as pd

# Get all alerts reported in TNS between December 10th 2025 and December 31st 2025
r = requests.post(
  "https://api.lsst.fink-portal.org/api/v1/tags",
  json={
    "tag": "in_tns",
    "n": "100", # (1)!
    "startdate": "2025-12-10",
    "stopdate": "2025-12-31"
  }
)

# Format output in a DataFrame
pdf = pd.read_json(io.BytesIO(r.content))
```

1. This the maximum number of _alerts_ to retrieve. There could be less than `n` in the specified period.

## Retrieving full object data

Note that only alerts that pass the criterion of a tag are stored in the corresponding tag table. But for a given object, not necessarily all alerts would satisfy the criteria. Therefore, you will not access the full lightcurve for each object through this endpoint.

Hence, if you need to query all the _alert_ data for _objects_ found with a tag search, or additional data that is not available in the tag table, you would do it in two steps:

```python title="Retrieve full lightcurve for objects found in tag search"
r0 = requests.post(
  "https://api.lsst.fink-portal.org/api/v1/tags",
  json={
    "tag": "in_tns",
    "n": "10",
    "columns": "r:diaObjectId" # (1)!
  }
)

mylist = [str(val["r:diaObjectId"]) for val in r0.json()]

# get full lightcurves for all these alerts
r1 = requests.post(
  "https://api.lsst.fink-portal.org/api/v1/sources",
  json={
    "diaObjectId": ",".join(mylist),
    "columns": "r:diaObjectId,r:midpointMjdTai,r:psfFlux,r:psfFluxErr", # (2)!
    "output-format": "json"
  }
)

# Format output in a DataFrame
pdf = pd.read_json(io.BytesIO(r1.content))
```

1. Select only the column you need to get faster results!
2. Select only the column(s) you need to get faster results!