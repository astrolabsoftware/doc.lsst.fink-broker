!!! info "List of arguments"
    The list of arguments for running a conesearch can be found on the [schema page :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"} and you can also retrieve it [programmatically](definitions.md).

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
        "columns": "r:diaObjectId,r:midpointMjdTai,r:psfFlux,r:psfFluxErr", # (1)!
      }
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

Note that in case of several objects matching, the results will be sorted according to the column
`v:separation_degree`, which is the angular separation in degree between the input (ra, dec) and the objects found. In addition, you can specify time boundaries:

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    # Get all objects falling within (center, radius) = ((ra, dec), radius)
    # between 2025-12-10 05:59:37.000 (included) and 2025-12-17 05:59:37.000 (excluded)
    r = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/conesearch",
      json={
        "ra": "7.4550",
        "dec": "-44.635",
        "radius": "150",
        "startdate": "2025-12-10 05:59:37.000",
        "window": 7  # in days
      }
    )

    # Format output in a DataFrame
    pdf = pd.read_json(io.BytesIO(r.content))
    ```

Instead of `window`, you can also use `stopdate`.

!!! warning "Time boundaries and first detection"
    When specifying time boundaries, you will restrict the search to alerts whose first detection was within the specified range of dates (and not all transients seen during this period).

Note that we group information and only display the data from the last alert. Hence, if you need lightcurves, that is to query all the _sources_ data for the `diaObjectId` found with a conesearch, you would do it in two steps:

=== "Python"

    ```python
    import io
    import requests
    import pandas as pd

    # Get the diaObjectId for the alert(s) within a circle on the sky
    r0 = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/conesearch",
      json={
        "ra": "7.4550",
        "dec": "-44.635",
        "radius": "5",
        "columns": "r:diaObjectId,r:midpointMjdTai"
      }
    )

    mylist = [val["r:diaObjectId"] for val in r0.json()]
    # len(mylist) = 26

    # get full lightcurves for all these alerts
    r1 = requests.post(
      "https://api.lsst.fink-portal.org/api/v1/sources",
      json={
        "diaObjectId": ",".join(mylist),
        "columns": "r:diaObjectId,r:midpointMjdTai,r:psfFlux,r:psfFluxErr",
        "output-format": "json"
      }
    )

    # Format output in a DataFrame
    pdf = pd.read_json(io.BytesIO(r1.content))
    # len(pdf) = 34

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
          "columns": "r:diaObjectId,r:midpointMjdTai"
        }
      )

      # do whatever
    ```

But note that for 10,000+ sources, this can be pretty slow, and impact other users. Instead for large catalogs, prefer the [Xmatch service](../../services/xmatch.md).