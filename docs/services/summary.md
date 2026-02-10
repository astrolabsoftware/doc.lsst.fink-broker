# Fink services

Now that we discussed the various components of Fink, it is time to access the data! We expose several services for the community. They all serve different purpose:

<div class="grid cards" markdown>

-   __Science Portal__

    ---

    The [Science Portal](science_portal.md) is a web application that allows you to perform visualisation of aggregated data in Fink, run simple queries. It is best for daily inspection. Data source is from the Fink Database, and new data is available in real-time. No login required.

    ![Screenshot](../img/science_portal_front.png)

-   __REST API__

    ---

    The [REST API](api/getting_started.md) allows you to access programmatically the processed data in Fink. It is used under the hood by the Science Portal, although it enables more. You can run simple queries, and it is best for automated workflows. Data source is from the Fink Database, and new data is available in real-time. No login required.

    ![Screenshot](../img/api.png)


-   __Livestream__

    ---

    The [Livestream service](livestream.md) allows you to access alert data (not aggregated objects) selected by [Fink filters](../science/filters.md) in real-time. It enables real-time analyses, automated workflows, and it is best for rapid follow-up observations. Data source is Fink Data Lake (different from the Database). Login required (see [fink-client](../developers/fink_client.md)).

-   __Fink Bots__

    ---

    The bots allow you to access alert or object data selected by some [Fink filters](../science/filters.md) in real-time or after the night. Information is sent to instant messaging services, such as Slack or Telegram. Best to scroll on a bus! Login can apply depending on the messaging service.

-   __Data Transfer__

    ---

    The [Data Transfer service](data_transfer.md) allows you to perform bulk download of alert data (not aggregated objects), run complex queries, and exotic analyses using Apache Spark and Apache Kafka. Data source is from the Fink Data Lake (different from the Database), and new data is available a few hours after the observing night has finished. Login required (see [fink-client](../developers/fink_client.md)).

-   __TOM Fink__

    ---

    The [TOM Fink](../developers/external/tom_fink.md) module allows you to easily connect Fink and [Target and Observation Managers :lucide-external-link:](https://tom-toolkit.readthedocs.io/en/stable/introduction/about.html){target="blank_"} (TOMs). Data source is either from the Fink Data Lake (through the Livestream service), or from the Fink Database (through the REST API).

</div>

