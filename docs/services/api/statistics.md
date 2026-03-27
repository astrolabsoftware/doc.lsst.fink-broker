# Statistics data

!!! info "List of arguments"
    The list of arguments to access statistics can be found at [https://api.lsst.fink-portal.org :lucide-external-link:](https://api.lsst.fink-portal.org){target="blank_"}. The schema of the returned payload can be found on the [schema page :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"} and you can also retrieve it [programmatically](definitions.md).

The [statistics](https://lsst.fink-portal.org/stats) page makes use of the REST API. If you want to further explore Fink statistics, get numbers for a talk, or create your own dashboard based on Fink data, you can do all of these yourself using the REST API. Here is an example using Python:

```python
import io
import requests
import pandas as pd

# get stats for all the year 2026
r = requests.post(
    "https://api.lsst.fink-portal.org/api/v1/statistics",
    json={"date": "2026", "output-format": "json"},
)

# Format output in a DataFrame
pdf = pd.read_json(io.BytesIO(r.content))
```

Each row is an observing night. Note `date` can be either a given night (`YYYYMMDD`), month (`YYYYMM`), year (`YYYY`), or an empty string (all observing night).