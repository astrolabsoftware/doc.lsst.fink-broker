# Tips for performance

## Select only fields you need

If `columns` is not specified, we transfer all available data fields (original LSST fields and Fink science module outputs) to you! This is easily over 100 fields, and I doubt you need all of them. After a phase of debug to learn which fields are useful, it is wiser to select only a subset of the fields:

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

Note that the fields should be comma-separated, and unknown field names are ignored. See the [definition](definitions.md) page for the list of available fields per endpoint.

## Playing with the cache

We cache query results at the level of the database. When you hit a timeout (120 seconds), if you relaunch the query, it will progress faster as it will re-use data already fetched. Do not abuse, but relaunching a few times sometimes does not harm :-)

