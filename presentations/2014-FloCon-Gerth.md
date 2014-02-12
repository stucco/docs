# Stucco - Situation and Threat Understanding by Correlating Contextual Observations

### Presented at [FloCon 2014](https://www.cert.org/flocon/)

### [Slides (pdf)](2014-FloCon-Gerth.pdf)

We report early results from a DHS-funded research project that is working on augmenting and annotating flow data in order to provide additional context for cybersecurity analysts. Stucco seeks to organize both structured data, such as that found in CVE, with data from unstructured text from places such as cybersecurity blogs so that it can be presented as part of a visual analytic system.  This research is led by Oak Ridge National Laboratory in collaboration with Pacific Northwest National Laboratory, Stanford University, and REN-ISAC.

Stucco addresses a fundamental problem in cyber attack discovery and situational understanding: the difficulty of putting security events in context. Security event data, such as intrusion detection system alerts, provide a starting point for analysis, but are information impoverished. Analysts must manually gather and synthesize relevant data to add context to events. Further, these sources are almost all from within an enterprise, ignoring the vast amounts of external data that could provide context to events. There is a wealth of available data that could be used to provide context for security events and to facilitate threat awareness relevant to an analyst’s environment, but this data is not currently systematically collected or integrated into cyber security systems.

Cyber security analysts need information to gain situational understanding – not more data that lacks context. The focus of Stucco is not to simply collect more data for indexing or clustering documents. Rather, Stucco will organize domain concepts that match the way analysts think. Stucco will collect data not typically integrated into security systems, extract domain concepts and relationships, and integrate that information into a cyber security knowledge graph to accelerate situational understanding.

By organizing data into a knowledge graph, security analysts will be able to rapidly search for domain concepts, speeding up access to the information needed for decision-making. Additionally, the information returned will only be that which is pertinent to their search. This approach ensures that only the minimal set of relevant information is provided to the analyst, ensuring they aren’t overwhelmed by additional data. Our approach enables analysts to more quickly identify events that can be discarded as false positives and to perform more thorough analysis with the relevant context to make decisions.

