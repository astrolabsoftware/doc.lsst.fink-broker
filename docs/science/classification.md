# Alert classification

## Sources of information

Following the analysis of alerts, we can identify three primary sources of information:

1. The original alert parameters received from LSST, such as flux, sky coordinates, emission time, and so on.
2. Supplementary data obtained from cross-referencing in Fink with external databases or catalogs, including SIMBAD, TNS, Gaia DR3, VSX, 4LAC, and others.
3. Additional information derived from user-defined processing in Fink, which may include statistical features, tags, machine learning scores, and more.

The challenge now lies in integrating these fields to derive meaningful scientific insights. There is no definitive ground truth in this context, and the possibilities for combinations are limitless, varying based on the specific target.

!!! tip "Alert vs object classification"

	An astronomical object on the sky can emit several alerts as its flux evolves, and based on available information on each alert, the classification can vary from one alert to another. We do not provide an _object_ classification method, but rather classification label for each alert. This is up to the user to decide on the nature of the object based on the list of alert classifications.

## Designing a user-driven classification scheme

To facilitate the identification of noteworthy events, users can create filters that combine multiple alert fields, allowing them to generate meaningful tags tailored to their research needs. In ZTF we were combining these tags in a ad-hoc manner to provide a single classification label per alert. With LSST, we have revised our classification strategy. It begins with user-defined [Fink Filters](science/filters.md), each generating a reusable tag that can be used unambiguously across all services. Each alert can now have multiple classification tags, and users can add as many classification tags as they require.


!!! note "Notion of candidate"
    Filter outputs should be regarded as likely candidates rather than definitive classifications. Further analyses of these candidates, such as spectroscopic follow-up, provide stronger confirmation.
