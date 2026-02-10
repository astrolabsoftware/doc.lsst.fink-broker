# Resolving astronomical names

!!! info "List of arguments"
    The list of arguments for resolving name can be found at [https://api.lsst.fink-portal.org :lucide-external-link:](https://api.lsst.fink-portal.org){target="blank_"}. The schema of the returned payload can be found on the [schema page :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"} and you can also retrieve it [programmatically](definitions.md).

Naming objects is a complex endeavor, often resulting in multiple names or designations for the same object. In the era of big data, this challenge becomes even more pronounced, as the need to quickly assign names to millions of objects can lead to non-intuitive designation processes.

Instead of proposing a new naming scheme, we aim to provide a service that allows users to explore existing names for a given object. Currently, you can resolve LSST `diaObjectId` within the SIMBAD database, the Transient Name Server, and Solar System databases recognized by the Quaero service from SSODNET. Additionally, this service allows you to determine if a corresponding LSST `diaObjectId` exists for any object found in these three databases.

We are committed to expanding this service and will continue to add new sources of information.

### TNS to ZTF

Question: I have a TNS identifier, are there LSST `diaObjectId` corresponding?

```python
import io
import requests
import pandas as pd

r = requests.post(
  'https://api.lsst.fink-portal.org/api/v1/resolver',
  json={
    'resolver': 'tns',
    'name_or_id': 'SN 2022and'
  }
)

# Format output in a DataFrame
pdf = pd.read_json(io.BytesIO(r.content)) # (1)!
```

1. Output:
```
   f:declination          f:discoverydate  f:fullname f:internalname        f:ra  f:redshift f:salt f:type
0       3.757143  2022-01-27 10:49:26.400  SN 2022and     ATLAS22cwl  150.027554       0.043      d  SN II
1       3.757143  2022-01-27 10:49:26.400  SN 2022and        PS22agq  150.027554       0.043      d  SN II
2       3.757143  2022-01-27 10:49:26.400  SN 2022and   ZTF22aaaifcx  150.027554       0.043      d  SN II
```

!!! tip "Downloading the full TNS table"
    You can also download the full TNS table by specifying an empty name:

    ```python title="Download full TNS catalog"
    import io
    import requests
    import pandas as pd

    r = requests.post(
      'https://api.lsst.fink-portal.org/api/v1/resolver',
      json={
          'resolver': 'tns',
          'name_or_id': '',
          'nmax': 1000000
      }
    )

    # > 200k rows
    pdf = pd.read_json(io.BytesIO(r.content))
    ```

### ZTF to TNS

Question: I have a LSST `diaObjectId` name, are there counterparts in TNS?


```python
import io
import requests
import pandas as pd

r = requests.post(
  'https://api.lsst.fink-portal.org/api/v1/resolver',
  json={
    'resolver': 'tns',
    'reverse': True,
    'name_or_id': '169865760522371491'
  }
)

# Format output in a DataFrame
pdf = pd.read_json(io.BytesIO(r.content))
```

### SIMBAD to ZTF

I have an astronomical object name referenced in SIMBAD, are there counterparts in LSST? As these objects can be extended, we typically provide coordinates, and then you need to run a conesearch:


```python
import io
import requests
import pandas as pd

r = requests.post(
  'https://api.lsst.fink-portal.org/api/v1/resolver',
  json={
    'resolver': 'simbad',
    'name_or_id': 'LEDA 1258009'
  }
)

if r.json() != []:
    print('Object found!')
    print(r.json())
    print()

    r = requests.post(
      'https://api.lsst.fink-portal.org/api/v1/conesearch',
      json={
        'ra': r.json()[0]['jradeg'],
        'dec': r.json()[0]['jdedeg'],
        'radius': 60,
        'columns': 'r:diaObjectId'
      }
    )

    # Format output in a DataFrame
    pdf = pd.read_json(io.BytesIO(r.content))
    print('Object(s) in LSST: {}'.format(pdf['r:diaObjectId'].to_numpy()))
else:
    print('No objects found')
```

Leading in this example to:

```
Object found!
[
  {
    'name': 'Si=Simbad, all IDs (via url)',
    'oid': 10915827,
    'oname': 'LEDA 1258009',
    'otype': 'G',
    'jpos': '10:00:06.70 +03:45:25.9',
    'jradeg': 150.02792, '
    jdedeg': 3.75722,
    'refPos': '2003A&A...412...45P',
    'z': None,
    'nrefs': 3
  }
]

Object(s) in LSST: [...]
```

### LSST to SIMBAD

Question: I have a LSST `diaObjectId` name, are there counterparts in SIMBAD within 1.5''?


```python
import io
import requests
import pandas as pd

r = requests.post(
  'https://api.lsst.fink-portal.org/api/v1/resolver',
  json={
    'resolver': 'simbad',
    'reverse': True,
    'name_or_id': '169773438939431059'
  }
)
pdf = pd.read_json(io.BytesIO(r.content))
```

### SSO to LSST

Question" I have a SSO name or number, are there LSST alerts corresponding?

No need to use a resolver here, as we have a table in our database indexed by SSO designation (see [Search by Solar System name](solar_system.md)):

```python
import io
import requests
import pandas as pd

# get LSST IDs for provisional designation 2003 UT84
r = requests.post(
  "https://api.lsst.fink-portal.org/api/v1/sso",
  json={
    "n_or_d": "2003 UT84",
    "columns": "r:diaSourceId,r:ssObjectId",
    "output-format": "json"
  }
)

pdf = pd.read_json(io.BytesIO(r.content)) # (1)!
```

1. Output:
```bash
         r:diaSourceId r:packed_primary_provisional_designation       r:ssObjectId f:sso_name
0   169892191587532827                                  K03U84T  21163620284511316  2003 UT84
1   169777833548185633                                  K03U84T  21163620284511316  2003 UT84
2   169848217053167670                                  K03U84T  21163620284511316  2003 UT84
...
# etc
```

### LSST to SSO

I have a LSST `ssObjectId`, is there a counterpart in the SsODNet quaero database, and what are all the known aliases to Fink?

```python
import io
import requests
import pandas as pd

r = requests.post(
  'https://api.lsst.fink-portal.org/api/v1/resolver',
  json={
    'resolver': 'ssodnet',
    'reverse': True,
    'name_or_id': '21163620284511316'
  }
)

if r.json() != []:
    name = r.json()[0]['r:unpacked_primary_provisional_designation']
    print('Asteroid counterpart found with designation {}'.format(name))
```

Leading to:

```
Asteroid counterpart found with designation 2003 UT84
```
