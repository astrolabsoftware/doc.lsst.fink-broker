## Search by Solar System object name

!!! info "List of arguments"
    The list of arguments for running a search by Solar System name can be found at [https://api.lsst.fink-portal.org :lucide-external-link:](https://api.lsst.fink-portal.org){target="blank_"}. The schema of the returned payload can be found on the [schema page :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"} and you can also retrieve it [programmatically](definitions.md).

Every time a new alert is emitted, a new `diaSource` is created. LSST makes an association (1'' matching radius) with a catalog of known Solar system objects (SSO) from the MPC prior to sending alerts. If the alert is matched to a known SSO, it is associated to an existing `ssObject`, and a `ssObjectId` is assigned to it. This page describes how to retrieve all `diaSources` information associated to the same `ssObject`. For alerts matched to a static object, see [Search by `diaObjectId`](diaobjectid.md).

## Object data

You can enter any name (e.g. Ukyounodaibu), number (e.g. 734394), or provisonal designation (e.g. 2015 BC557, K15Bt7C) of asteroids. Under the hood, we resolve the name using the [quaero :lucide-external-link:](https://ssp.imcce.fr/webservices/ssodnet/api/quaero/){target="blank_"} service from SsODNet. You can also search for comets (although none has been seen yet by Rubin in the alert stream), but note that we have far less comets than asteroids.

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    # get all data for provisional designation 2015 BC557
    r = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/sso",
        json={"n_or_d": "2015 BC557", "output-format": "json"},
    )

    # Format output in a DataFrame
    pdf = pd.read_json(io.BytesIO(r.content))
    ```

=== "curl"

    ```bash
    # Get data for the asteroid 2015 BC557 and save it in a CSV file
    curl -H "Content-Type: application/json" -X POST -d '{"n_or_d":"2015 BC557", "output-format":"csv"}' https://api.lsst.fink-portal.org/api/v1/sso -o 2003_UT84.csv
    ```

=== "wget"

    Note that you can replace spaces `" "` in object name with underscores in queries:

    ```bash
    # you can also specify parameters in the URL
    wget "https://api.lsst.fink-portal.org/api/v1/sso?n_or_d=2015_BC557&output-format=json" -O 2015_BC557.json
    ```

=== "Query URL"

    SSO pages are indexed by packed provisional designation. Paste this query on your browser to inspect the object:
    ```
    https://lsst.fink-portal.org/K15Bt7C
    ```

!!! warning "Faster queries"
    For faster queries, you can select only alert fields of interest by specifying the argument `columns` in your payload:

    ```python title="Specify only a subset of columns"
    r = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/sso",
        json={
            "n_or_d": "2015 BC557",
            "columns": "r:midpointMjdTai,r:psfFlux,r:psfFluxErr,r:ra,r:dec",
            "output-format": "json",
        },
    )
    ```

    See the list of available columns at the [schema page :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"}

You can also retrieve the data for several objects at once:

```python title="Several objects at once"
import io
import requests
import pandas as pd

# ID as string
mylist = ["734394", "K15Bt7C", "Schwarzschilda", "Ukyounodaibu"]

# get alert data for many objects
r = requests.post(
    "https://api.lsst.fink-portal.org/api/v1/sso",
    json={
        "n_or_d": ",".join(mylist),
        "columns": "r:midpointMjdTai,r:psfFlux,r:psfFluxErr,r:ra,r:dec",
        "output-format": "json",
    },
)

# Format output in a DataFrame
pdf = pd.read_json(io.BytesIO(r.content))
```


!!! warning "Mixing types"
    Note that you can mix asteroid and comet names, unless you specify `withEphem=True` (see below), in which
    case you must give only a list of asteroid names or list of comet names (schemas for ephemerides are not the same).

!!! warning "Do not abuse!"
    Although the REST API gives you access to hundreds of millions of alerts without account, it is not designed to massively download data. If you have hundreds of objects to query, you probably want to select only a small subset of columns, or you can use the [Data Transfer service :lucide-external-link:](../data_transfer.md){target="blank_"}.

Note that you can also choose different output format:

=== "json"

    ```python
    import io
    import requests
    import pandas as pd

    # get data for provisional designation 2015 BC557
    r = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/sso",
        json={"n_or_d": "2015 BC557", "output-format": "json"},
    )

    # Format output in a DataFrame
    pdf = pd.read_json(io.BytesIO(r.content))
    ```

=== "csv"


    ```python
    import io
    import requests
    import pandas as pd

    # get data for provisional designation 2015 BC557
    r = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/sso",
        json={"n_or_d": "2015 BC557", "output-format": "csv"},
    )

    # Format output in a DataFrame
    pd.read_csv(io.BytesIO(r.content))
    ```

=== "Parquet"

    ```python
    import io
    import requests
    import pandas as pd

    # get data for provisional designation 2015 BC557
    r = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/sso",
        json={"n_or_d": "2015 BC557", "output-format": "parquet"},
    )

    # Format output in a DataFrame
    pdf = pd.read_parquet(io.BytesIO(r.content))
    ```

=== "votable"

    ```python
    import io
    import requests
    from astropy.io import votable

    # get data for provisional designation 2015 BC557
    r = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/sso",
        json={"n_or_d": "2015 BC557", "output-format": "votable"},
    )

    # VO table
    vt = votable.parse(io.BytesIO(r.content))
    ```

### Adding ephemerides from Miriade

!!! warning "Slower queries"
    Beware it adds few seconds delay per API call.

You can also attach the ephemerides provided by the [Miriade ephemeride service :lucide-external-link:](https://ssp.imcce.fr/webservices/miriade/api/ephemcc/){target="blank_"}:

```python title="Adding ephemerides"
import io
import requests
import pandas as pd

# get data for object 2015 BC557
r = requests.post(
    "https://api.lsst.fink-portal.org/api/v1/sso",
    json={"n_or_d": "2015 BC557", "withEphem": True, "output-format": "json"},
)

# Format output in a DataFrame
pdf = pd.read_json(io.BytesIO(r.content))
```

Where columns not prefixed by `r:` or `f:` are fields returned from Miriade.

### Retrieving cutout stamps

For each alert, you can easily retrieve the associated stamps using the `/api/v1/cutouts` endpoint. For this, you need first to retrieve the `diaSourceId`, and then query for the cutouts:

```python
import io
import requests
from astropy.io import fits

# Get all diaSourceId for object 2015 BC557
# Output is sorted from more recent to least recent
r = requests.post(
    "https://api.lsst.fink-portal.org/api/v1/sso",
    json={"n_or_d": "2015 BC557", "columns": "r:diaSourceId", "output-format": "json"},
)

# Get Science cutouts as FITS for the last 10 alerts
for sid in r.json()[0:10]:
    out = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/cutouts",
        json={
            "diaSourceId": str(sid["r:diaSourceId"]),
            "kind": "Science",
            "output-format": "FITS",
        },
    )
    data = fits.open(io.BytesIO(out.content), ignore_missing_simple=True)
    data.writeto(f"{sid['r:diaSourceId']}_cutoutScience.fits")
```

For more options when downloading images, see the [alert image data :lucide-external-link:](imagesearch.md){target="blank_"} page. Be careful, each image is about 30KB.

## Bulk download for all SSO lightcurves

!!! info "List of arguments"
    The list of arguments for retrieving alert data can be found at [https://api.lsst.fink-portal.org :lucide-external-link:](https://api.lsst.fink-portal.org){target="blank_"} (`/api/v1/ssobulk` endpoint), and the schema of the table (json) can be found at [https://api.lsst.fink-portal.org/api/v1/ssobulk?schema=True :lucide-external-link:](https://api.lsst.fink-portal.org/api/v1/ssobulk?schema=True){target="blank_"}


!!! warning "Experimental service"
    Data aggregation starts at 2026.04.01 (SSO schema from the project was not complete prior to this date). Data is updated once a month, on the first day. 

### Full table

This service lets you download all SSO lightcurves in one call (parquet format) to avoid performing an infinite loop on `/api/v1/sso`:

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    # No arguments
    r = requests.post("https://api.lsst.fink-portal.org/api/v1/ssobulk", json={})

    # Format parquet output in a DataFrame
    pdf = pd.read_parquet(io.BytesIO(r.content))
    ```

=== "curl"

    ```bash
    curl -H "Content-Type: application/json" -X POST \ 
        -d '{}' \
        https://api.lsst.fink-portal.org/api/v1/ssobulk \
        -o sso_fink_lsst_lc.parquet
    ```

Note that we only allow `parquet` as output format as JSON or CSV would be too big. You can retrieve the schema of the table using using the schema argument:

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    r = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/ssobulk", json={"schema": True}
    )

    schema = r.json()
    ```

### Single object

You can also retrieve information about a single object, using its name or IAU number:

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    r = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/ssobulk",
        json={"sso_name": "1998 TT26"},
    )

    # Format output in a DataFrame
    pdf = pd.read_parquet(io.BytesIO(r.content))
    ```



## SSoFT: Solar System object Fink Table

!!! info "List of arguments"
    The list of arguments for retrieving alert data can be found at [https://api.lsst.fink-portal.org :lucide-external-link:](https://api.lsst.fink-portal.org){target="blank_"} (`/api/v1/ssoft` endpoint), and the schema of the table (json) can be found at [https://api.lsst.fink-portal.org/api/v1/ssoft?schema=True :lucide-external-link:](https://api.lsst.fink-portal.org/api/v1/ssoft?schema=True){target="blank_"}

### Full table

This service lets you query the table containing aggregated parameters for known Solar System objects in Fink. This table is updated once a month, with all data in Fink.

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    r = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/ssoft", json={"output-format": "parquet"}
    )

    # Format output in a DataFrame
    pdf = pd.read_parquet(io.BytesIO(r.content))
    ```

=== "curl"

    ```bash
    curl -H "Content-Type: application/json" -X POST \
        -d '{"output-format":"parquet"}' \
        https://api.lsst.fink-portal.org/api/v1/ssoft -o ssoft.parquet
    ```

!!! info "Starting date: 2026/04/04"
    The SSOFT uses LSST alert data only from 2026/04/04. Prior to that date, the project was not proving the fields topocentric and heliocentric distances that are used to fit parameters.

This table contains basic statistics (e.g. coverage in time for each object, name, number, ...), fitted parameters (absolute magnitude, phase parameters, spin parameters, ...), quality statuses, and version numbers. It is several megabytes by default (and as the survey will progress, it will become several hundreds of megabytes), so you can also decide to transfer only a subset of fields:

```python
import io
import requests
import pandas as pd

r = requests.post(
    "https://api.lsst.fink-portal.org/api/v1/ssoft",
    json={"columns": "sso_name,H_g,chi2red", "output-format": "parquet"},
)

# Format output in a DataFrame
pdf = pd.read_parquet(io.BytesIO(r.content))
```

To know the parameters of interest, you can retrieve the schema of the table using using the schema argument:

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    r = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/ssoft", json={"schema": True}
    )

    schema = r.json()
    ```

=== "curl"

    ```bash
    curl -H "Content-Type: application/json" -X POST \
        -d '{"schema": "True"}' \
        https://api.lsst.fink-portal.org/api/v1/ssoft -o ssoft_schema_json

    # print on terminal
    cat ssoft_schema_json | jq
    ```

or view it in your browser at [https://api.lsst.fink-portal.org/api/v1/ssoft?schema=True :lucide-external-link:](https://api.lsst.fink-portal.org/api/v1/ssoft?schema=True){target="blank_"}.

### Single object

You can also retrieve information about a single object, using its name or IAU number:

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    r = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/ssoft",
        json={"sso_name": "1998 TT26", "output-format": "parquet"},
    )

    # Format output in a DataFrame
    pdf = pd.read_parquet(io.BytesIO(r.content))
    ```

=== "curl"

    ```bash
    # using name
    curl -H "Content-Type: application/json" -X POST -d '{"output-format":"json", "sso_name": "1998 TT26"}' https://api.lsst.fink-portal.org/api/v1/ssoft
    ```

### Flavors

By default, we expose the parameters from the `HG` model, that is the simplest phase curve model. 

!!! warning "Data quality"
    Note that even with the simple `HG` phase model, fitted parameters are not great as the phase coverage is very small as we have a very low number of templates and observations. We expect the coverage to improve as the observatory deploys more templates on sky. 

We also expose more complex models such as `HG1G2` ([Muinonen et al. 2010 :lucide-external-link:](https://doi.org/10.1016/j.icarus.2010.04.003){target="blank_"}), `sfHG1G2` ([Colazo et al 2025 :lucide-external-link:](https://doi.org/10.1016/j.icarus.2025.116577){target="blank_"}), and `sHG1G2` ([Carry et al 2024 :lucide-external-link:](https://doi.org/10.1051/0004-6361/202449789){target="blank_"}). You need to specify the argument `flavor`:

```python
import io
import requests
import pandas as pd

r = requests.post(
    "https://api.lsst.fink-portal.org/api/v1/ssoft",
    json={"flavor": "HG1G2", "output-format": "parquet"},
)
```

Idem for the schema of the table, e.g. to get the schema for the SSOFT using the `HG1G2` model [https://api.lsst.fink-portal.org/api/v1/ssoft?flavor=HG1G2&schema=True :lucide-external-link:](https://api.lsst.fink-portal.org/api/v1/ssoft?flavor=HG1G2&schema=True){target="blank_"}.

### Version

The table is versioned (`YYYY.MM`), and you can access previous versions (first available version starts at `2026.08`):

```python
import io
import requests
import pandas as pd

r = requests.post(
    "https://api.lsst.fink-portal.org/api/v1/ssoft",
    json={"version": "2026.08", "output-format": "parquet"},
)
```

By default (that is `version` unspecified), the service will return the current month one (the latest).

## Adding more parameters from the BFT

The SSOFT of Fink contains only the parameters from the observations and phase curve modeling. However you can easily join this table with the [ssoBFT :lucide-external-link:](https://ssp.imcce.fr/webservices/ssodnet/api/ssobft/){target="blank_"} (Solar System Objects Broad and Flat Table of all properties) table provided by [LTE :lucide-external-link:](https://ssp.imcce.fr){target="blank_"}. This table contains physical and dynamical parameters for all known objects (about 900MB as of 2026). Here is an example on joining the two tables:

```python
import io
import requests
import pandas as pd
import rocks  # pip install space-rocks

# Get the SSOFT
r0 = requests.post(
    "https://api.lsst.fink-portal.org/api/v1/ssoft", json={"output-format": "parquet"}
)

ssoft = pd.read_parquet(io.BytesIO(r0.content))

# Get the BFT - and cache it for later
bft = rocks.load_bft()

# Join the two
combined = ssoft.merge(bft, left_on="sso_name", right_on="name", how="left")
```
