# Science Roadmap

!!! tip "Science Goal"
	Fink's primary scientific goal is to maximize the scientific impact of the Rubin alert data stream. One person's trash is another person's treasure, so rather than focusing on a specific area, our ambition is to explore the transient and variable sky as a whole, encompassing everything from Solar system objects to galactic and extragalactic phenomena. The list of publications can be found at [https://fink-broker.org/papers/ :lucide-external-link:](https://fink-broker.org/papers/){target="blank_"}

## Strategy

Each night, telescopes and surveys are sending alerts when they detect changes in the luminosity of astronomical objects. These alerts contain minimal information such as sky location, flux of the transient, and sometimes historical data at this location. The main role of Fink is to enrich these alerts with additional information to identify interesting candidates for follow-up observations or further scientific processing.


Defining what is considered _interesting_ varies significantly from person to person, as individual scientific interests and curiosities differ widely. Recognizing this, our strategy focuses on enabling users to extend Fink by incorporating their unique scientific logic and receive tailored outputs that resonate with their own perspectives. To facilitate this personalization, we have introduced the concept of [Fink Science Modules](broker/science_modules.md) and [Fink Filters](broker/filters.md), which allow users to contribute additional capabilities and new insights to the Fink ecosystem (e.g. machine learning classification, identification from astronomical catalogs, or feature extraction algorithms). This approach not only enriches the content but also creates a collaborative environment where diverse viewpoints are valued and integrated, ultimately enhancing the overall learning experience for all users.

``` mermaid
graph TB
  A(Optical large-scale survey alert streams) --> D((User-defined modules in Fink));
  B[(External databases & catalogs)] <-.-> D((User-defined modules in Fink));
  C(Smaller volume alert streams) --> D((User-defined modules in Fink));
  D --> E(Solar system object, Variable star, Supernova, ...);
```

## Main surveys

Although Fink is capable to listen to various streams, its design is driven by large-scale surveys sending hundreds of thousands to millions of alerts every night, such as the Rubin Observatory and the Zwicky Transient Facility.

### Zwicky Transient Facility

Since 2019, Fink is operating on the [ZTF :lucide-external-link:](https://www.ztf.caltech.edu/){target="blank_"} public alert stream, which has constituted an excellent opportunity to engage projects with the scientific community while preparing the ground for the Rubin Observatory alert data. Have a look at the [publication list :lucide-external-link:](https://fink-broker.org/papers/){target="blank_"} for example, or the [SN Ia candidates :lucide-external-link:](https://www.wis-tns.org/search?&discovered_period_value=1&discovered_period_units=years&unclassified_at=0&classified_sne=0&include_frb=0&name=&name_like=0&isTNS_AT=all&public=all&ra=&decl=&radius=&coords_unit=arcsec&reporting_groupid%5B%5D=null&groupid%5B%5D=null&classifier_groupid%5B%5D=null&objtype%5B%5D=null&at_type%5B%5D=null&date_start%5Bdate%5D=&date_end%5Bdate%5D=&discovery_mag_min=&discovery_mag_max=&internal_name=&discoverer=Fink&classifier=&spectra_count=&redshift_min=&redshift_max=&hostname=&ext_catid=&ra_range_min=&ra_range_max=&decl_range_min=&decl_range_max=&discovery_instrument%5B%5D=null&classification_instrument%5B%5D=null&associated_groups%5B%5D=null&official_discovery=0&official_classification=0&at_rep_remarks=&class_rep_remarks=&frb_repeat=all&frb_repeater_of_objid=&frb_measured_redshift=0&frb_dm_range_min=&frb_dm_range_max=&frb_rm_range_min=&frb_rm_range_max=&frb_snr_range_min=&frb_snr_range_max=&frb_flux_range_min=&frb_flux_range_max=&num_page=50&display%5Bredshift%5D=1&display%5Bhostname%5D=1&display%5Bhost_redshift%5D=1&display%5Bsource_group_name%5D=1&display%5Bclassifying_source_group_name%5D=1&display%5Bdiscovering_instrument_name%5D=0&display%5Bclassifing_instrument_name%5D=0&display%5Bprograms_name%5D=0&display%5Binternal_name%5D=1&display%5BisTNS_AT%5D=0&display%5Bpublic%5D=1&display%5Bend_pop_period%5D=0&display%5Bspectra_count%5D=1&display%5Bdiscoverymag%5D=1&display%5Bdiscmagfilter%5D=1&display%5Bdiscoverydate%5D=1&display%5Bdiscoverer%5D=1&display%5Bremarks%5D=0&display%5Bsources%5D=0&display%5Bbibcode%5D=0&display%5Bext_catalogs%5D=0){target="blank_"} sent every night to the Transient Name Server! More information is available on the [ZTF documentation website :lucide-external-link:](https://doc.ztf.fink-broker.org/en/latest/){target="blank_"}.

### Rubin Observatory

The Rubin Observatory has started its operations at the beginning of 2026 and it will scan the sky for the next decade. Every night, 7 Community Brokers have unrestricted access to the complete alert stream: [ALeRCE :lucide-external-link:](https://alerce.science/){target="blank_"}, [AMPEL :lucide-external-link:](https://github.com/AmpelProject){target="blank_"}, [ANTARES :lucide-external-link:](https://antares.noirlab.edu/){target="blank_"}, [BABAMUL/BOOM :lucide-external-link:](https://github.com/boom-astro){target="blank_"}, [Fink :lucide-external-link:](https://fink-broker.org){target="blank_"}, [Lasair :lucide-external-link:](https://lasair.readthedocs.io){target="blank_"}, and [Pitt-Google :lucide-external-link:](https://github.com/mwvgroup/Pitt-Google-Broker){target="blank_"}.

The LSST alert data is an unfiltered, 5-sigma alert stream, ([original content :lucide-external-link:](https://sdm-schemas.lsst.io/apdb.html){target="blank_"}). Alerts are coming from all over the sky, from many different phenomena. The role of Fink is to assist scientists in efficiently analyzing alert data. Among its various functions, Fink collects and stores alert data, [enriches](broker/science_modules.md) it with information from other surveys and catalogs, as well as user-defined enhancements like machine-learning classification scores. It also [redistributes](broker/filters.md) the most promising events for further analysis, including follow-up observations. Full schema, including enrichment from Fink, can be found [online :lucide-external-link:](https://lss.fink-portal.org/schemas){target="blank_"} and it is also accessible programmatically ([API](services/api/schema.md)).

In this context we have made strong partnerships with other missions (e.g. [SVOM :lucide-external-link:](https://www.svom.eu/en/home/){target="blank_"}) or network of telescopes (e.g. [GRANDMA :lucide-external-link:](https://grandma.ijclab.in2p3.fr/){target="blank_"}). We are open to contributions in all science cases already included in Fink, but also to new contributions that are not listed here. If you have a science proposal and you would like to integrate it with the broker, [contact us :lucide-external-link:](mailto:contact@fink-broker.org).

