---
title: CauseMos
has_children: false
nav_order: 2
---
# CauseMos
Causemos resources can be found under
- TopDownModelingAndHMI: https://github.com/WorldModelers/TopDownModelingAndHMI
- Causemos repos: https://github.com/uncharted-causemos

## Workflows

<a id="w4"></a>
### [W4](index.html#w4) Document management + reading + integration/assembly + HMI

Data ingesting is done via [anansi](https://github.com/uncharted-causemos/anansi), which contains a suite of scripts for handling DART-CDRs, 
INDRA statements. 

### Iniital setup
There are a few things you need to setup before running the ingestion:

- Ensure you have ElasticSearch setup and have the mappings properly as per [atlas](https://github.com/uncharted-causemos/atlas). 

```
Install index mappings `ES=<host:port> python es_mapper.py`
```


- Ensure you have the geolocation reference data, an **anasi** there is a `geo_loader.py` to do use this. You will need to download the two geolocation data files manually as outlined in the script documentation.

```
python geo_loader.py
```

### Running data ingestion
Causemos ingest INDRA statements dataset and DART CDR dataset to create a single Knowledge Base. Specifically it will update a global `corpus` index
that houses document metadata, and creates a new `indra-<id>` index as the Knowledge Base.


For all intent and purposes here, SOURCE and TARGET should have the same values.

```
#!/usr/bin/env bash

SOURCE_ES=xyz \
SOURCE_USERNAME=xyz \
SOURCE_PASSWORD=xyz \
TARGET_ES=xyz \
TARGET_USERNAME=xyz \
TARGET_PASSWORD=xyz \
DART_DATA=<path_to_cdr.json> \
INDRA_DATASET=<path_to_indra_directory> \
python src/knowledge_pipeline.py
```

Against running IDNRA/DART services, you can
- Download INDRA datasets via `scripts/download_indra_s3.py`
- Download DART CDR dataset via `scripts/build_dart.sh`


Causemos can also in addition generate recommendation indices, that can be used as suggestions for doing curations in bulk. For more 
information please see [this repository](https://github.com/uncharted-causemos/wm-curation-recommendation).


<a id="w5"></a>
### [W5](index.html#w5) Document management + reading + integration/assembly + HMI + BYOD
In this workflow, it is assumed that both INDRA and DART are running as web services.


### Iniital setup
BYOD + incremental assembly processing takes place outside of Causemos app due to heavy data processing and high latency. Causemos makes use
of Prefect server to schedule asynchronous jobs. See notes for setting up Prefect and sequential agent [here](https://github.com/uncharted-causemos/slow-tortoise).



### Running Causemos with BYOD
BYOD is an optional feature in Causemos that integrates with INDRA and DART services. To
eanble this feature Causemos needs to start with the "dart" command line option, this will enable the periodic
synchronizations against DART and INDRA.

```
# Usage: yarn start-server --schedules foo,bar
yarn start-server --schedules dart
```

When a document is uploaded through Causemos, the request is sent to DART, then behind the scenes Causeos server will poll DART
to see what new reader output are available on the Kafka queue, cross reference them against Causemos internal document upload tracker,
and then send the valid entries to INDRA to do incremental assembly.
