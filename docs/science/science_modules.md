# Fink science modules

## What is a Science Module?

The alert stream is made of millions sources that vary in the field of view of the telescope. This encompasses a wide variety of phenomemon, for which no single individual possesses all knowledge. Our strategy in Fink is to bring diverse scientific expertise together to create a platform where collective intelligence can thrive. By facilitating collaboration among experts across various scientific fields, we create an ecosystem where knowledge is not only shared but also amplified: one person's trash is another person's treasure.

To facilitate this collaborative work, we have introduced the concept of Fink Science Modules. They are bricks of analysis allowing users to contribute additional capabilities and new insights based on their expertise to the Fink ecosystem (e.g. machine learning classification, identification from astronomical catalogs, or feature extraction algorithms). From the incoming raw alert data, and eventually using external resources, the task of each Science Module is to add a _scientific surplus_ to better characterise the event.

``` mermaid
graph LR
  A(RA, DEC, flux) --> B((Science module #1));
  B -..-> C((Science module #N));
  C --> D(RA, DEC, flux, labels + ML scores + flags, ...);
```

The science modules are provided by the scientific community and they focus on a wide range of scientific cases, from Solar System science to galactic and extragalactic studies. These modules can share information, allowing the input of one module to utilize the output of one or more other modules.

## Available Rubin science modules

!!! info "Open source and open data"
	Each science module provides added values in form of extra fields inside the alert packet, and these fields are freely accessible by anyone. The code sources of science modules can be found at [https://github.com/astrolabsoftware/fink-science :lucide-external-link:](https://github.com/astrolabsoftware/fink-science){target="blank_"}.

The schema associated to Fink/Rubin processed alerts can be found on the [Portal :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"}. To summarize, the outputs are organised in 3 categories:

- `xm.*`: outputs from catalog crossmatches (SIMBAD, Gaia DR3, Legacy Survey DR8, VSX, ...). If the catalogs are available at CDS, we can integrate them directly. For external catalogs, depending on their size, we can consider hosting them ourselves.
- `clf.*`: outputs from classifiers. Users can upload pre-trained models, and each alert will receive a score. There are binary models focusing on specific class of transients (e.g. SN Ia vs the rest of the world), or broad classifiers that output a vector of probabilities for a variety of classes.
- `lc_features.<band>.*`: lightcurve features from SNAD. 26 features per filter band.
- `misc.*`: outputs that are neither crossmatches, classifiers, or lightcurve features from SNAD. These modules typically issue flags, statistical fit results, or aggregated information.

## Designing a science module

Check out the [tutorial](../developers/science_module_tutorial.md) to construct a Fink Science Module.
