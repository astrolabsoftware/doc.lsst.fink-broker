# Alert data quality

In ZTF, we rely a lot on the real/bogus score inside the alert packet to disentangle between artifacts from astrophysical signals.
In LSST alert packet, the equivalent of the real/bogus score is the `reliability` field, which is a score between 0 (artifact) and 1 (real).
This field is provided by the Rubin team, and send to us.
Unfortunately, as the instrument is new and its tuning changes frequently, it is still quite difficult to provide a reliable `reliability` score.
We advise users to NOT use it blindly. As a matter of fact, the Rubin project launched a Zooniverse campaign using Rubin data to improve the Real/Bogus model ([https://www.zooniverse.org/projects/ebellm/rubin-difference-detectives :lucide-external-link:](https://www.zooniverse.org/projects/ebellm/rubin-difference-detectives){target="blank_"}) to collect enough classifications to do a better model. Stay tuned.

!!! warning "`reliability` score is not reliable yet"
    Do not use `reliability` score blindly as the survey starts. It will get updated frequently to account for what the Rubin project learns.

On our side, the Fink commissioning team has been working hard to correlate various alert packet fields with their quality. We design a Fink block, [b_good_quality :lucide-external-link:](https://lsst.fink-portal.org/schemas){target="blank_"} that you can use when designing your filters (Livestream & Data Transfer). A badge on the portal also appears on every object.