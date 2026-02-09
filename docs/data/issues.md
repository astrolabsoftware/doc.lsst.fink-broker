# Known issues

Users will find a list of known issues with alert data. We will update this list as we learn new things.

### Duplicated MJD in history

Although we have no duplicated `midpointMjdTai` at the level of `diaSource` for a given `diaObjectId` , we are finding duplicated `midpointMjdTai` in `prvDiaSources` while other fields are not duplicated. For v10 data, we find several hundred sources with this problem. E.g. take the source `diaSource = 169865760548586859` (`diaObjectId = 169856995032564064`).

### Detection history size

!!! warning "`nDiaSources` field is not reliable before 2026/01/30"
    Do not rely blindly on `nDiaSources` in commissioning data. Discard any source data in doubt.

For data before 2026/01/30, there was a hidden float round-trip in the Rubin APDB interface and so precision was lost; this led to some `DIAObjects` losing history and others getting extra. As a consequence, you might find alerts with:

- `nDiaSources` equal to 1 and `prvDiaSources = []` while it is not the first time the underlying object emits an alert
- `nDiaSources` going from `N` on the previous alert, to `N + (a lot)` on the subsequent alert, while it should have been `N + 1` (and with `prvDiaSources` containing a lot of unrelated measurements).

### Missing fields

Although the alert packet has a [long schema :lucide-external-link:](https://sdm-schemas.lsst.io/apdb.html){target="blank_"} coming from Rubin, many fields are not populated as the Rubin pipeline is incomplete. This includes (but is not limited to):

- Forced photometry section (`prvForcedDiaSources`)
- Various Solar System fields in `mpc_orbits` and `ssSource`
- Various quality flags in `diaSource`
- Time of the first diaSource (`diaObject.firstDiaSourceMjdTai`)

### Designation for SSO

Prior `2026/01/30`, the field `ssSource.designation` contained packed designation instead of unpacked designation.

### Cutouts orientation

The WCS information in the header does not align correctly to the North-South direction in standard FITS visualisation tools.
The problem is explained in the Rubin Community Forum: [https://community.lsst.org/t/how-to-use-wcss-in-dp1-and-commissioning-processing/10769 :lucide-external-link:](https://community.lsst.org/t/how-to-use-wcss-in-dp1-and-commissioning-processing/10769){target="blank_"}.

### Diffraction spikes

We find many alerts around bright stars, following diffraction spikes. We advise users to check for bright stars nearby alerts of interest.