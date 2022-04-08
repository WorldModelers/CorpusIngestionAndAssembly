---
title: CauseMos
has_children: false
nav_order: 2
---
# CauseMos
Causemos is the main HMI for the World Modelers program, built and maintined by Uncharted Software.
It is an ecosystem consists of Causemos web application plus a suite of services and utilities. The essential ones for handling unstructured data are:
- Atlas [Initial setup, schemas and mappings](https://github.com/uncharted-causemos/atlas)
- Anansi [Knowledge ingestion and incremental assembly](https://github.com/uncharted-causemos/anansi)
- Causemos [Causemos web app](https://github.com/uncharted-causemos/causemos)

BYOD - Requires the infrastructure parts:
- [Causemos clustering infrastructure](https://github.com/uncharted-causemos/slow-tortoise)

Recommendation/curation - optional:
- [Recommendation service](https://github.com/uncharted-causemos/wm-curation-recommendation)

For running and using Causemos, the documentation can be found in [TopDownModelingAndHMI](https://github.com/WorldModelers/TopDownModelingAndHMI)

## Workflows

<a id="w4"></a>
### [W4](index.html#w4) Document management + reading + integration/assembly + HMI

In this workflow, Causemos ingests, combines and enriches INDRA statements dataset and DART CDR dataset to create a new Knowledge Base dataset


#### Iniital setup
Ensure you have ElasticSearch setup and have the mappings properly as per [atlas](https://github.com/uncharted-causemos/atlas). 

Install mappings.

```
ES=<host:port> python es_mapper.py
```


#### Running data ingestion
Perfrom a one-time data load of gelocation references, this upsert into a `geo` index. Per documentation in the script, you will need to download and extract from:

```
http://download.geonames.org/export/dump/allCountries.zip
http://clulab.cs.arizona.edu/models/gadm_woredas.txt
```

Once extracted

```
ES=<es_url> ES_USER=<user> ES_PASSWORD=<password> python geo_loader.py
```

Assume you have INDRA and DART datasets n the file system, the ingestion process can be kicked off with the following snippet.

Note: For all intent and purposes here, SOURCE and TARGET should have the same values.

Note: DART_DATA is expected to be in JSONL format, one CDR per line.

```
#!/usr/bin/env bash

SOURCE_ES=xyz \
SOURCE_USERNAME=xyz \
SOURCE_PASSWORD=xyz \
TARGET_ES=xyz \
TARGET_USERNAME=xyz \
TARGET_PASSWORD=xyz \
DART_DATA=<path_to_dart_cdr.json> \
INDRA_DATASET=<path_to_indra_directory> \
python src/knowledge_pipeline.py
```

Once done, build and start the Causemos application. You will find the new Knowledge Base under "New Analysis Projec", the Knowledge Base
will appear with the name given by INDRA's metadata.


Against running INDRA/DART service instances, you can
- Download INDRA datasets via `scripts/download_indra_s3.py`
- Download DART CDR dataset via `scripts/build_dart.sh`


#### Post processing - Optional
Causemos can also in addition generate recommendation indices, that can be used as suggestions for doing curations in bulk. For more 
information please see [this repository](https://github.com/uncharted-causemos/wm-curation-recommendation).


<a id="w5"></a>
### [W5](index.html#w5) Document management + reading + integration/assembly + HMI + BYOD
In this workflow, it is assumed that both INDRA and DART are running as web services.


#### Iniital setup
BYOD + incremental assembly processing takes place outside of Causemos app due to heavy data processing and higher latency. 
This process uses the Prefect infrastructure for task scheduling and runs the `incremental_pipeline.py` script in the anansi project

For setting up Prefect infrastructure, see example instructions [here](https://github.com/uncharted-causemos/slow-tortoise/blob/master/infra/prefect/setup.md)

To create an env
- Ssh to Prefect-server
- conda create -n prefect-seq -c conda-forge "python>=3.8.0" prefect "elasticsearch==7.11.0" "boto3==1.17.18" "smart_open==5.0.0" python-dateutil requests


You also need a env/config file on the prefect server, with connection credentials to DART, INDRA, and others

```
export SOURCE_ES=
export SOURCE_USERNAME=
export SOURCE_PASSWORD=
export TARGET_ES=
export TARGET_USERNAME=
export TARGET_PASSWORD=
export DART_HOST=
export DART_USER=
export DART_PASS=
export INDRA_HOST=

# optional
export CURATION_HOST=
```

To setup Prefect agent
- Copy anansi/src to the Prefect-server
- Copy credentials to Prefect-server
- Ssh to Prefect-server
- Stop Prefect local-agent
- Source env: `source <env/config file>`
- Restart local-agent: `PREFECT__ENGINE__EXECUTOR__DEFAULT_CLASS="prefect.executors.LocalExecutor" PYTHONPATH="${PYTHONPATH}:<path_to_anansi_src>" prefect agent local start --api "http://<Prefect-server>:4200/graphql" --label "non-dask"`

To register `incremental_pipeline.py`:
- Ssh to Prefect-server
- Set the "shouldRegister" flag to True in the python file
- Activate the env `conda activate prefect-seq`
- Re(register) `python incremental_pipeline.py`
 

#### Running Causemos with BYOD
BYOD is an optional feature in Causemos that integrates with INDRA and DART services. To
eanble this feature Causemos sever needs to start with the "dart" command line option, this will enable the periodic
synchronizations against DART and INDRA.

```
# Usage: yarn start-server --schedules foo,bar
yarn start-server --schedules dart
```

When a document is uploaded through Causemos, the request is sent to DART. Then behind the scenes Causeos server will poll DART
to see what new reader output are available on the Kafka queues, cross-reference them against Causemos internal document upload tracker,
and then send the valid entries to INDRA to do incremental assembly. 
