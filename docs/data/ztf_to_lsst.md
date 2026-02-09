# Migration from ZTF to LSST

If you were using Fink for ZTF, your experience with LSST will be very similar, although there are some differences to know.

## Data availability delay

If you want to receive alert data in real-time in ZTF, you must use the Livestream service. For the Science Portal and the REST API, data is available only after the observing night, as our databases are updated once daily. This practice has led to unfortunate delays for observers relying on the API for follow-up observations.

In the case of LSST, the databases are updated in real-time, allowing users to access data immediately through the Livestream, Science Portal, and REST API services. Please note that data accessible via the Data Transfer and xmatch services will remain available at the end of the observing night for post-processing purposes.

## Static vs moving objects

In ZTF, all alerts would follow the same schema. We would then apply a number of cuts to separate alerts associated to moving objects (e.g. Solar System objects - SSO - or space debris) from alerts associated to static objects on the sky (stars, galaxies, etc. things considered static given the timescale).

In LSST, things are different. LSST makes an association (1'' matching radius) with a catalog of known SOlar system objects from the MPC prior to sending alerts. As a consequence, the alert content changes based on the provenance:

- if the alert is not associated to a know SSO, it will have the field `diaObject` populated.
- if the alert is associated to a know SSO, it will have the fields `mpc_orbit` and `ssSource` populated.
- if the alert is closed to a know SSO and a static object, it will have the fields `diaObject`, `mpc_orbit` and `ssSource` populated.

This means in practice not all fields are available in all alerts.

## Solar system object ranking in crowded fields

In crowded fields, the risk is that there are many SSO within the matching radius of 1''. Therefore the field `ssSource` contains a field `diaDistanceRank` which is the rank of the `diaSourceId`-identified source in terms of its closeness to the predicted SSO position.

## Accessing SSO data in the Portal

In ZTF, you can search for a Solar System object by providing its name, resulting in a list of corresponding alerts. However, since the webpages are indexed by `objectId` (which is the identifier for static objects in the sky), you cannot directly link to a Solar System object via URL.

In LSST, webpages are indexed by both the static identifier (`diaObjectId`) and the names of Solar System objects (packed designations), e.g. try [https://lsst.fink-portal.org/J92E18H :lucide-external-link:](https://lsst.fink-portal.org/J92E18H){target="blank_"}.

## Alert classification

In ZTF, users had the ability to create filters that combined multiple alert fields, enabling them to generate meaningful tags tailored to their research needs. These tags were then used to propose a [single alert classification label :lucide-external-link:](https://doc.ztf.fink-broker.org/en/latest/broker/classification/){label="blank_"}. However, this method had three limitations:

1. **Inconsistent labeling between Livestream and API**: Not all tags were utilized in creating the final (single) classification label.
2. **Limited classification per alert**: Each alert could have only one classification label, making it impossible to represent different perspectives.
3. **Complex classification algorithm**: The classification label algorithm functioned as a non-finite decision tree, making it difficult to extend and resolve conflicts as more nodes were added.

With LSST, we have revised our classification strategy. It begins with user-defined Fink Filters, each generating a reusable tag that can be used unambiguously across all services. Each alert can now have multiple tags, and users can add as many tags as they require.

## API Differences

### Lightcurve data

In ZTF, you can obtain light curve data for a specific `objectId` by using the endpoint `/api/v1/objects`. In LSST, there is a clear distinction between `diaSource` (representing a single alert) and `diaObject` (which encompasses a collection of alerts associated with the same object in the sky). Therefore, in LSST, light curve data is provided through the endpoint `/api/v1/sources`, while summary statistics about an object (such as the number of detections and mean flux) can be accessed via `/api/v1/objects`.

!!! danger "Obtain lightcurve data for a given ID"
    `/api/v1/objects` has a different meaning for ZTF or LSST

    === "ZTF"

        ```python
        import io
        import requests
        import pandas as pd

        # get lightcurve data for ZTF21aaxtctv
        r = requests.post(
            "https://api.ztf.fink-portal.org/api/v1/objects",
            json={
                "objectId": "ZTF21aaxtctv",
                "output-format": "json"
            }
        )

        # Format output in a DataFrame
        pdf = pd.read_json(io.BytesIO(r.content))
        ```

    === "LSST"

        ```python
        import io
        import requests
        import pandas as pd

        # get lightcurve data for 169830579938263176
        r = requests.post(
            "https://api.lsst.fink-portal.org/api/v1/sources",
            json={
                "diaObjectId": "169830579938263176",
                "output-format": "json"
            }
        )

        # Format output in a DataFrame
        pdf = pd.read_json(io.BytesIO(r.content))
        ```

### Object data

In ZTF, you could not get information about the underlying object (number of detections, mean flux, etc.). In LSST, this is now possible using the `/api/v1/objects` endpoint:

```python
import io
import requests
import pandas as pd

# get summary data for 169830579938263176
r = requests.post(
    "https://api.lsst.fink-portal.org/api/v1/objects",
    json={
        "diaObjectId": "169830579938263176",
        "output-format": "json"
    }
)

# Format output in a DataFrame
pdf = pd.read_json(io.BytesIO(r.content))
```

Beware of the fact that `/api/v1/objects` had another meaning for ZTF (see above).