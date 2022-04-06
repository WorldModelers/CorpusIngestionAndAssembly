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

1. Ensure you have ElasticSearch setup and have the mappings properly as per [atlas](https://github.com/uncharted-causemos/atlas). 

```
Install index mappings `ES=<host:port> python es_mapper.py`
```

2. Ensure you have the geolocation reference data, an **anasi** there is a `geo_loader.py` to do use this. You will need to download the two geolocation data files manually as outlined in the script documentation.
```
python geo_loader.py
```

### Running dta ingestion
For all intent and purposes here, SOURCE and TARGET will be the same.

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


<a id="w5"></a>
### [W5](index.html#w5) Document management + reading + integration/assembly + HMI + BYOD

Bring your own data is an optional feature in Causemos that integrates with INDRA and DART services. To
eanble this feature Causemos needs to start with the "dart" command line option, this will enable the periodic
synchronizations against DART and INDRA.

```
yarn start-server --schedules dart
```
