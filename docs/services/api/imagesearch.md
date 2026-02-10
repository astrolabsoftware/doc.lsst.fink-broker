# Image data

!!! info "List of arguments"
    The list of arguments for running a search by tag can be found at [https://api.lsst.fink-portal.org :lucide-external-link:](https://api.lsst.fink-portal.org){target="blank_"}. The schema of the returned payload can be found on the [schema page :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"} and you can also retrieve it [programmatically](definitions.md).

With every alert, Rubin sends 3 cutouts: one for the current observation, one for the template used to perform difference image analysis, and one for the difference. Each cutout is a FITS file, with at least 30x30 pixels. It contains data, but also mask and variance planes.

!!! note "Cutouts are accessed via diaSourceId"
    Cutouts are stored in a datalake in Fink, rather than in the database. They are accessible via `diaSourceId`, that is the unique ID for each alert (and not `diaObjectId`). Therefore, you need first to retrieve the list of `diaSourceId` for an object before accessing it cutouts:

```python title="Obtain diaSourceId"
# Get all diaSourceId for 169830579938263176
r = requests.post(
    'https://api.lsst.fink-portal.org/api/v1/sources',
    json={
        'diaObjectId': '169830579938263176',
        'columns': 'r:diaSourceId,r:midpointMjdTai'
    }
)
```

Note that output is sorted from more recent to least recent:

```python title="Last observation"
last_diaObjectId = r.json()[0]["r:diaSourceId"]
```

### FITS

You can retrieve the original FITS file stored in the alert:

=== "curl"

    ```bash
    curl -H "Content-Type: application/json" \
        -X POST -d \
        '{"diaSourceId":"169936132527095844", "kind":"Science", "output-format": "FITS"}' \
        https://api.lsst.fink-portal.org/api/v1/cutouts -o 169936132527095844_cutoutScience.fits
    ```

=== "wget"

    ```bash
    wget "https://api.lsst.fink-portal.org/api/v1/cutouts?diaSourceId=169936132527095844&kind=Science&output-format=FITS" -O 169936132527095844_cutoutScience.fits
    ```

=== "Python"

    ```python
    import io
    import requests
    from astropy.io import fits

    # get data for 169936132527095844
    r = requests.post(
        'https://api.lsst.fink-portal.org/api/v1/cutouts',
        json={
            'diaSourceId': '169936132527095844',
            'kind': 'Science',
            'output-format': 'FITS'
        }
    )

    data = fits.open(io.BytesIO(r.content), ignore_missing_simple=True)
    data.writeto('169936132527095844_cutoutScience.fits')
    ```

and then you would read it as usual:

```python
from astropy.io import fits

with fits.open("169936132527095844_cutoutScience.fits") as hdu:
    header = hdu[0].header
    data = hdu[0].data
```

### PNG

We provide a method to save PNG directly:

=== "curl"

    ```bash
    curl -H "Content-Type: application/json" \
        -X POST -d \
        '{"diaSourceId":"169936132527095844", "kind":"Science"}' \
        https://api.lsst.fink-portal.org/api/v1/cutouts -o 169936132527095844_cutoutScience.png
    ```

=== "wget"

    ```bash
    # you can also specify parameters in the URL, e.g. with wget:
    wget "https://api.lsst.fink-portal.org/api/v1/cutouts?diaSourceId=169936132527095844_cutoutScience&kind=Science" -O 169936132527095844_cutoutScience.png
    ```

=== "Python"

    ```python
    import io
    import requests
    from PIL import Image as im

    # get data for 169936132527095844
    r = requests.post(
        'https://api.lsst.fink-portal.org/api/v1/cutouts',
        json={
            'diaSourceId': '169936132527095844',
            'kind': 'Science',
        }
    )

    image = im.open(io.BytesIO(r.content))
    image.save('169936132527095844_cutoutScience.png')
    ```

![image](../../img/169936132527095844_cutoutScience.png)

!!! tip "Display in Jupyter Notebook"
    In a notebook, you would use `display(image)` to display the cutout in the page. If it is too small, resize it using `image.resize((height, width))`.

Note you can choose between the `Science`, `Template`, or `Difference` images.
You can also customise the image treatment before downloading, e.g.

```python
import io
import requests
from PIL import Image as im

# get data for 169936132527095844
r = requests.post(
    'https://api.lsst.fink-portal.org/api/v1/cutouts',
    json={
        'diaSourceId': '169936132527095844',
        'kind': 'Science', # (1)!
        'stretch': 'sigmoid', # (2)!
        'colormap': 'viridis', # (3)!
        'pmin': 0.5, # (4)!
        'pmax': 99.5, # (5)!
        'convolution_kernel': 'gauss' # (6)!
    }
)

image = im.open(io.BytesIO(r.content))
image.save('mysupercutout.png')
```

1. Science, Template, Difference
2. sigmoid[default], linear, sqrt, power, log
3. Valid matplotlib colormap name (see matplotlib.cm). Default is grayscale.
4. The percentile value used to determine the pixel value of minimum cut level. Default is 0.5. No effect for sigmoid.
5. The percentile value used to determine the pixel value of maximum cut level. Default is 99.5. No effect for sigmoid.
6. Convolve the image with a kernel (gauss or box). Default is None (not specified).


### 2D array

You can also retrieve only the data block stored in the alert:

```python
import requests

# get data for 169936132527095844
r = requests.post(
    'https://api.lsst.fink-portal.org/api/v1/cutouts',
    json={
        'diaSourceId': '169936132527095844',
        'kind': 'Science',
        'output-format': 'array'
    }
)

array = r.json()["b:cutoutScience"]
```

You can also request the 3 cutouts in one call now by specifying `kind=All`:

```python
import requests

# get data for 169936132527095844
r = requests.post(
    'https://api.lsst.fink-portal.org/api/v1/cutouts',
    json={
        'diaSourceId': '169936132527095844',
        'kind': 'All',
        'output-format': 'array'
    }
)

data = r.json()
data.keys()
# dict_keys(['b:cutoutDifference', 'b:cutoutScience', 'b:cutoutTemplate'])
```

