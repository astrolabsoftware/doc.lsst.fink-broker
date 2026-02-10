## Search by Solar System object name

!!! info "List of arguments"
    The list of arguments for running a search by Solar System name can be found at [https://api.lsst.fink-portal.org :lucide-external-link:](https://api.lsst.fink-portal.org){target="blank_"}. The schema of the returned payload can be found on the [schema page :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"} and you can also retrieve it [programmatically](definitions.md).

Every time a new alert is emitted, a new `diaSource` is created. LSST makes an association (1'' matching radius) with a catalog of known Solar system objects (SSO) from the MPC prior to sending alerts. If the alert is matched to a known SSO, it is associated to an existing `ssObject`, and a `ssObjectId` is assigned to it. This page describes how to retrieve all `diaSources` information associated to the same `ssObject`. For alerts matched to a static object, see [Search by `diaObjectId`](diaobjectid.md).

## Alert data

You can enter any name (e.g. Schwarzschilda, JulienPeloton), number (e.g. 8467), or provisonal designation (e.g. 2003 UT84, K03U84T) of asteroids. Under the hood, we resolve the name using the [quaero](https://ssp.imcce.fr/webservices/ssodnet/api/quaero/) service from SsODNet. You can also search for comets (although none has been seen yet by Rubin in the alert stream), but note that we have far less comets than asteroids.

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    # get data for provisional designation 2003 UT84
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/sso",
      json={
        "n_or_d": "2003 UT84",
        "output-format": "json"
      }
    )

    # Format output in a DataFrame
    pdf = pd.read_json(io.BytesIO(r.content))
    ```

=== "curl"

    ```bash
    # Get data for the asteroid 2003 UT84 and save it in a CSV file
    curl -H "Content-Type: application/json" -X POST -d '{"n_or_d":"2003 UT84", "output-format":"csv"}' https://api.lsst.fink-portal.org/api/v1/sso -o 2003_UT84.csv
    ```

=== "wget"

    Note that you can replace spaces `" "` in object name with underscores in queries:

    ```bash
    # you can also specify parameters in the URL
    wget "https://api.lsst.fink-portal.org/api/v1/sso?n_or_d=2003_UT84&output-format=json" -O 2003_UT84.json
    ```

=== "Query URL"

    SSO pages are indexed by packed provisional designation. Paste this query on your browser to inspect the object:
    ```
    https://lsst.fink-portal.org/K03U84T
    ```

You can also retrieve the data for several objects at once:

```python title="Several objects at once"
import io
import requests
import pandas as pd

# ID as string
mylist = ["2003 UT84", "464064", "Schwarzschilda"]

# get alert data for many objects
r = requests.post(
  "https://api.lsst.fink-portal.org/api/v1/sources",
  json={
    "n_or_d": ",".join(mylist),
    "columns": "r:midpointMjdTai,r:psfFlux,r:psfFluxErr,r:ra,r:dec",
    "output-format": "json"
  }
)

# Format output in a DataFrame
pdf = pd.read_json(io.BytesIO(r.content))
```


!!! warning "Mixing types"
    Note that you can mix asteroid and comet names, unless you specify `withEphem=True` (see below), in which
    case you must give only a list of asteroid names or list of comet names (schemas for ephemerides are not the same).

!!! warning "Do not abuse!"
    Although the REST API gives you access to hundreds of millions of alerts without account, it is not designed to massively download data. If you have hundreds of objects to query, you probably want to select only a small subset of columns, or you can use the [Data Transfer service](../data_transfer.md).

Note that you can also choose different output format:

=== "json"

    ```python
    import io
    import requests
    import pandas as pd

    # get data for provisional designation 2003 UT84
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/sso",
      json={
        "n_or_d": "2003 UT84",
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

    # get data for provisional designation 2003 UT84
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/sso",
      json={
        "n_or_d": "2003 UT84",
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

    # get data for provisional designation 2003 UT84
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/sso",
      json={
        "n_or_d": "2003 UT84",
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

    # get data for provisional designation 2003 UT84
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/sso",
      json={
        "n_or_d": "2003 UT84",
        "output-format": "votable"
      }
    )

    # VO table
    vt = votable.parse(io.BytesIO(r.content))
    ```

!!! tip "Optimisation: selecting only a subset of fields"
    If `columns` is not specified, we transfer all available data fields (original LSST fields and Fink science module outputs). But you can also choose to transfer only a subset of the fields:


    ```python title="Select fields"
    # select only time, flux, position
    r = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/sso",
        json={
            "n_or_d": "8467",
            "columns": "r:midpointMjdTai,r:psfFlux,r:psfFluxErr,r:ra,r:dec",
        }
    )
    ```

    Note that the fields should be comma-separated. Unknown field names are ignored.

## Object data

The endpoint `TBD` give access to summary information about an object, such as the orbital parameters, the number of alerts for the object, number of oppositions, etc. You would simply use:


```python title="Object summary information"
import requests

# get summary data for 2003 UT84
r = requests.post(
  "https://api.lsst.fink-portal.org/TBD",
  json={
    "n_or_d": "2003 UT84",
    "output-format": "json"
  }
)

if r.status_code == 200:
  # dictionary with object properties
  properties = r.json()[0]
```

## Adding ephemerides from Miriade

You can also attach the ephemerides provided by the [Miriade ephemeride service](https://ssp.imcce.fr/webservices/miriade/api/ephemcc/):

```python title="Adding ephemerides"
import io
import requests
import pandas as pd

# get data for object 2003 UT84
r = requests.post(
  "https://api.lsst.fink-portal.org/api/v1/sso",
  json={
    "n_or_d": "2003 UT84",
    "withEphem": True,
    "output-format": "json"
  }
)

# Format output in a DataFrame
pdf = pd.read_json(io.BytesIO(r.content))
```

Where columns not prefixed by `r:` or `f:` are fields returned from Miriade.

!!! warning "Limitations"
    Beware it adds few seconds delay per API call and color ephemerides are returned only for asteroids