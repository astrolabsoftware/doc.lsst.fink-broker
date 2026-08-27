# Conesearch

!!! info "List of arguments"
    The list of arguments for running a conesearch can be found at [https://api.lsst.fink-portal.org](https://api.lsst.fink-portal.org). The schema of the returned payload can be found on the [schema page :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"} and you can also retrieve it [programmatically](definitions.md).

## Simple conesearch

This service allows you to search objects in the database matching in position on the sky given by (RA, Dec, radius). The initializer for RA/Dec is very flexible and supports inputs provided in a number of convenient formats. The following ways of initializing a conesearch are all equivalent:

* 8.986275, -42.709834, 5
* 00h35m56.71s, -42d42m35.40s, 5
* 00 35 56.71, -42 42 35.40, 5
* 00:35:56.71, -42:42:35.40, 5

!!! warning "Search radius"
    The search radius is always in arcsecond, and the maximum radius length is 18,000 arcseconds (5 degrees).

Try this on a terminal:

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    # Get all objects falling within (center, radius) = ((ra, dec), radius)
    r = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/conesearch",
        json={
            "ra": "8.986275",
            "dec": "-42.709834",
            "radius": "5",
            "columns": "r:diaObjectId,r:midpointMjdTai,r:psfFlux,r:psfFluxErr",  # (1)!
        },
    )

    # Format output in a DataFrame
    pdf = pd.read_json(io.BytesIO(r.content))
    ```

    1. Select only the column(s) you need to get faster results!
=== "curl"

    ```bash
    # Get all objects falling within (center, radius) = ((ra, dec), radius)
    curl -H "Content-Type: application/json" -X POST \
        -d '{"ra":"8.986275", "dec":"-42.709834", "radius":"5"}' \
        https://api.lsst.fink-portal.org/api/v1/conesearch -o conesearch.json
    ```
=== "wget"

    ```bash
    # you can also specify parameters in the URL, e.g. with wget:
    wget "https://api.lsst.fink-portal.org/api/v1/conesearch?ra=8.986275&dec=-42.709834&radius=5&output-format=json" -O conesearch.json
    ```

=== "Query URL"

    Paste this query on your browser to see results:
    ```
    https://lsst.fink-portal.org/?action=conesearch&ra=8.986275&dec=-42.709834&radius=5
    ```

Note that in case of several objects matching, the results will be sorted according to the column
`v:separation_degree`, which is the angular separation in degree between the input (ra, dec) and the objects found. In addition, you can specify time boundaries:

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    # Get all objects falling within (center, radius) = ((ra, dec), radius)
    # between 2025-12-10 05:59:37.000 (included) and 2025-12-17 05:59:37.000 (included)
    r = requests.post(
        "https://api.lsst.fink-portal.org/api/v1/conesearch",
        json={
            "ra": "7.4550",
            "dec": "-44.635",
            "radius": "150",
            "startdate": "2025-12-10 05:59:37.000",
            "stopdate": "2025-12-17 05:59:37.000",  # equivalently expressed in days "window": 7
            "kind": "within",
            "columns": "r:diaObjectId,r:midpointMjdTai,r:psfFlux,r:psfFluxErr,f:firstDiaSourceMjdTaiFink",
        },
    )

    # Format output in a DataFrame
    pdf = pd.read_json(io.BytesIO(r.content))
    ```

The parameter `kind` defines the time filtering strategy. `kind=within` will return transients that strictly vary within the bounds. `kind=across` means all transients that had varied inside the bounds (but could also vary before/after). All other parameters remain the same. See below for graphical explanations that would hopefully drive the writing of the query.

### Filtering by boundaries

If the user specifies both `startdate` and `stopdate` (or `window`), depending on `kind`, here is what you should expect (green=the object is returned, red=the object is not returned):

<img width="1085" height="677" alt="image" src="https://github.com/user-attachments/assets/2ddfe532-1876-44fc-ba44-4f1dfa81724a" />

### Filtering by startdate

If the user specifies `startdate`, depending on `kind`, here is what you should expect (green=the object is returned, red=the object is not returned):

<img width="1085" height="677" alt="image" src="https://github.com/user-attachments/assets/6d3060b8-0971-4ef0-ba17-a61f7928cb2e" />

### Filtering by stopdate

If the user specifies `stopdate`, depending on `kind`, here is what you should expect (green=the object is returned, red=the object is not returned):

<img width="1085" height="677" alt="image" src="https://github.com/user-attachments/assets/8873c275-7db7-4673-a2f8-d20bf5c5491b" />


## Retrieving full object data

Note that we group information and only display the data from the last alert. Hence, if you need lightcurves, that is to query all the _sources_ data for the `diaObjectId` found with a conesearch, you would do it in two steps:


```python title="Retrieve full lightcurve for objects found in conesearch"
import io
import requests
import pandas as pd

# Get the diaObjectId for the alert(s) within a circle on the sky
r0 = requests.post(
    "https://api.lsst.fink-portal.org/api/v1/conesearch",
    json={
        "ra": "61.9648",
        "dec": "-48.713",
        "radius": "10",
        "columns": "r:diaObjectId,r:midpointMjdTai",
    },
)

mylist = [val["r:diaObjectId"] for val in r0.json()]
# len(mylist) = 2

# get full lightcurves for all these alerts
r1 = requests.post(
    "https://api.lsst.fink-portal.org/api/v1/sources",
    json={
        "diaObjectId": ",".join(mylist),
        "columns": "r:diaObjectId,r:midpointMjdTai,r:psfFlux,r:psfFluxErr",
        "output-format": "json",
    },
)

# Format output in a DataFrame
pdf = pd.read_json(io.BytesIO(r1.content))
# len(pdf) = 371

# group by diaObjectId
pdf.groupby("r:diaObjectId").value_counts()
```

## Crossmatch with catalogs

You can easily perform a crossmatch with a catalog of astronomical sources by looping over entries:

=== "Python"

    ```python
    mycatalog = read(...)

    for source in mycatalog:
        r0 = requests.post(
            "https://api.lsst.fink-portal.org/api/v1/conesearch",
            json={
                "ra": source["ra"],
                "dec": source["dec"],
                "radius": "5",
                "columns": "r:diaObjectId,r:midpointMjdTai",
            },
        )

        # do whatever
    ```

But note that for 10,000+ sources, this can be pretty slow, and impact other users. Instead for large catalogs, prefer the Xmatch service will be made available shortly.
