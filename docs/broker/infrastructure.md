# Technological consideration

!!! tip "Technological goal"
	On the technological front, Fink is dedicated to providing a robust infrastructure and cutting-edge streaming services for Rubin scientists, enabling seamless user-defined science cases within a big data context.

![Screenshot](../img/infrastructure.png#only-light)
![Screenshot](../img/infrastructure-alt.png#only-dark)

## Cloud computing for all

Driven by large-scale optical surveys such as the Zwicky Transient Facility and Rubin Observatory, Fink operates on large scientific cloud infrastructures ([VirtualData :lucide-external-link:](https://virtualdata.fr/){target="blank_"} at Paris-Saclay, and [CC-IN2P3 :lucide-external-link:](https://doc.cc.in2p3.fr/en/Hosting/cloud-iaas.html){target="blank_"}), and it is based on several established bricks such as [Apache Spark :lucide-external-link:](http://spark.apache.org/){target="blank_"}, [Apache Kafka :lucide-external-link:](https://kafka.apache.org/){target="blank_"} and [Apache HBase :lucide-external-link:](https://hbase.apache.org/){target="blank_"}.

But what is an advantage for computing at scale often translates into an inconvenience for scientific analysis. Fink engineering team leverages a wide array of cloud technologies to scale analyses, yet many users seeking to extend Fink for their own analyses lack training in these advanced technologies. This presents a significant challenge: we must create **intuitive interfaces** that enable users to seamlessly integrate their ideas into Fink without the need for specialized expertise in big data. By focusing on user-friendly design, we aim to democratize access to powerful analytical tools, allowing individuals from diverse backgrounds to actively contribute to and benefit from the Fink platform. Our goal is to enable users to express their unique insights without being hindered by technical barriers, fostering a more inclusive and innovative analytical community.

!!! info "Programming languages"
	The primary language chosen for most APIs is Python, which is widely used in the astronomy community, has a large scientific ecosystem, and easily integrates with existing tools. However, under the hood, Fink utilizes several other languages, including Scala, Java, and Rust. Modern codebases often require a variety of programming languages!



<!-- You can install and test all of these components in local mode, with moderate resources required (see [testing Fink](../developers/testing_fink.md)). -->

## Can Fink do everything?

While many analyses can be conducted end-to-end within Fink, we do not always provide all the necessary components for a complete analysis. This may be due to limitations in our expertise, the substantial effort required for integration, or the need for proprietary access to certain external data that we do not possess. Instead, we offer interoperable tools that allow you to export enriched data for further analysis elsewhere. In practice, this is where plateform such as [Astro-COLIBRI :lucide-external-link:](https://astro-colibri.science/){target="blank_"}, observation managers such as [TOMs :lucide-external-link:](https://lco.global/tomtoolkit/){target="blank_"} and marshals such as [SkyPortal :lucide-external-link:](https://skyportal.io/){target="blank_"}, come into play. These tools, which have interfaces developed for most brokers, facilitate the coordination of follow-up observations and additional scientific analyses after we enrich the data.
