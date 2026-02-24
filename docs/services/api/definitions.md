## On the use of non-relational database

The data accessible from the REST API is stored in tables within the Apache HBase database. HBase is a non-relational database (i.e., it is not SQL-like). This has many advantages, notably that it does not rely on a fixed schema, allowing for easy schema migration. However, this also means that we must handle the description of the data ourselves when you run a query

To give you a more practical view, a long-lived variable object that emits alerts for years will consist of alert data with different fields: some fields may have been created, while others may have been deleted over the course of the event. This is handled at the query runtime.

## Fields definition

!!! info "What are the available fields in Fink?"
    You can also browse fields per endpoint online at [https://lsst.fink-portal.org/schemas](https://lsst.fink-portal.org/schemas), or you can programmatically access them. For example the fields returned by the endpoint `/api/v1/sources`:

    ```bash
    curl -H "Content-Type: application/json" -X POST -d '{"endpoint": "/api/v1/sources"}'\
        https://api.lsst.fink-portal.org/api/v1/schema -o sources_schema.json
    ```

### Prefix meaning (r:, f:, ...)

The fields are all prefixed with a letter determining the provenance of the data:

- `r:` original data from LSST
- `f:` added values from [Fink Science modules](../../science/science_modules.md)
- `v:` added values from Fink generated at the query runtime
- `b:` cutout data

These are called column families.

## Tag definition

!!! info "What are the available user-defined in Fink?"
    The list of Fink class can be found at [https://api.ztf.fink-portal.org/api/v1/tags](https://api.ztf.fink-portal.org/api/v1/tags). We recommend also to read how the [classification scheme](../../science/classification.md) is built.

    You can programmatically access the list of all the Fink tags using e.g.:

    ```bash
    curl -H "Content-Type: application/json" -X GET \
        https://api.ztf.fink-portal.org/api/v1/tags -o fink_tags.json
    ```
