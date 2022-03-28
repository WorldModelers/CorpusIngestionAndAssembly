---
title: HUME
has_children: false
nav_order: 5
---
# HUME

## Workflows

<a id="w1"></a>
### [W1](index.html#w1) Reading only

In this workflow Hume runs as a standalone reading system.

#### System Preparation

For this mode, you will need to prepare a folder of CDR files (a richer article representation from DART), a ontology metadata file, and a config file.

You will also need to create a new working folder, we'll use `runtime` as an example in this document.

##### CDR Files

Detailed information of the schema of CDR can be found from DART side. For Hume's purposes, the fields below are required:

```json
{
  "document_id": "required,str: unique identifier of article",
  "source_uri": "optional,str: how to fetch the document",
  "extracted_metadata": {
    "Author": "optional,str: author of article",
    "CreationDate": "optional,str: in '%Y-%m-%d' without quote format. Will help time resolution if provided",
    "Pages": "optional,int: Will help for genre determination"
  },
  "extracted_text": "required,str: original text of article",
  "content_type": "optional,str: mime_type string, will be used genre determination"
}
```

Files should be named `[document_id].json`, where `[document_id]` is replaced with the actual document_id.

Copy all json files into your `runtime/corpus` directory.

##### Ontology Metadata File

The ontology metadata file should be formatted according to the following schema: https://github.com/WorldModelers/Ontologies/blob/master/CompositionalOntology_metadata.yml

##### Docker Config File

The config file is a json file that must be visible at runtime from `/extra/config.json` inside the container. An example is provided as follows:

```json
{
  "hume.domain": "WM",
  "hume.num_of_vcpus": "int, required: Number of cpu cores available to you. It at least needs to be 2, and please only include number of physical cores instead of SMT cores.",
  "hume.tmp_dir": "required,str: a bind point for sharing data in between your local system and docker instance",
  "hume.manual.cdr_dir": "required,str: A path, from inside docker that contains dir of docs you want to process, if you're following above, it should be /extra/corpus",
  "hume.manual.keep_pipeline_data": "optional,bool default to false: when enabled, we'll keep intermediate process file in between runs so you don't start from beginning. But if your previous run is finished and you'll kick off the new run, please set it to false for removing intermediate process file and run everything from scratch.",
  "hume.external_ontology_path": "optional, str: When provided, hume will use the ontology metadata file you provided instead of pre-shipped one. This path needs to be accessible from inside docker",
  "hume.external_ontology_version": "optional, str, but mandatory when you specify hume.external_ontology_path: id of the external ontology",
  "hume.use_regrounding_cache": "bool, optional: When enabled, the system will save intermediate files of processing result into a directory that if we see the same document later, we can skip certain processing steps and only kick off regrounding pipeline.",
  "hume.regrounding_cache_path": "str, required when hume.use_regrounding_cache is true: a persist directory for hosting regrounding cache. Ideally to be a persist storage that can be shared in between docker instances. Also please don't reuse grounding cache from OIAD."
}
```

Please copy the config file to `runtime/config.json`

#### To Run the Hume System

Run Hume with the following command:

```bash
docker run -it -v runtime:/extra docker.io/wmbbn/hume:R2022_03_21 /usr/local/envs/py3-jni/bin/python3 /wm_rootfs/git/Hume/src/python/dart_integration/manual_processing.py
```

After the run finishes, the results will be accessible at `[hume.tmp_dir]/results/[TIMESTAMP]/results`. The resulting output will be in JSON-LD format, as described [here](https://github.com/BBN-E/Hume/wiki#output-format-json-ld).


<a id="w2"></a>
### [W2](index.html#w2) Reading + integration/assembly

The W2 workflow is achieved by following the instructions for [W1](hume.html#w1) above and then referring the INDRA instructions for [W2](indra.html#w2).

<a id="w3"></a>
### [W3](index.html#w3) Document management + reading + integration/assembly

In this workflow, Hume is integrated with DART and INDRA in a BYOD system.

#### System Preparation

For this mode, you will need to prepare a config file.

You will also need to create a new working folder, we'll use `runtime` as an example in this document.

##### Docker Config File

The config file is a json file that must be visible at runtime from `/extra/config.json` inside the container. It's modified from https://github.com/twosixlabs-dart/python-kafka-consumer/blob/master/pyconsumer/resources/env/test.json. An example is provided as follows:

Note: Do not change unannotated fields in the example unless you really know what you're doing.

```json
{
  "kafka.bootstrap.servers": "str,required: example is wm-ingest-pipeline-streaming-1.prod.dart.worldmodelers.com:9093",
  "auth": {
    "username": "str,required. If auth is not required, please delete auth dict directly",
    "password": "str,required"
  },
  "app": {
    "id": "hume",
    "auto_offset_reset": "earliest",
    "enable_auto_commit": false
  },
  "topic": {
    "from": "str,required: topic to listen to from kafka"
  },
  "CDR_retrieval": "str,required: example is https://wm-ingest-pipeline-rest-1.prod.dart.worldmodelers.com/dart/api/v1/cdrs",
  "DART_upload": "str,required: example is https://wm-ingest-pipeline-rest-1.prod.dart.worldmodelers.com/dart/api/v1/readers",
  "Ontology_retrieval": "str,required: example is https://wm-ingest-pipeline-rest-1.prod.dart.worldmodelers.com/dart/api/v1/ontologies",
  "DART_upload_labels": [
    "from_docker"
  ],
  "hume.domain": "WM",
  "hume.num_of_vcpus": "int, required: Number of cpu cores available to you. It at least needs to be 2, and please only include number of physical cores instead of SMT cores.",
  "hume.tmp_dir": "required,str: a bind point for sharing data in between your local system and docker instance",
  "hume.streaming.mini_batch_size": "int, required: For streaming mode, this deterimines the size of a mini batch (which is used for improved efficiency)",
  "hume.streaming.mini_batch_wait_time": "int, required: For streaming mode, even if we received fewer documents than mini_batch_size, the system will start processing after mini_batch_wait_time seconds.",
  "hume.streaming.keep_pipeline_data": "bool, optional: When enabled all intermediate files during processing will be preserved. This is mainly a debugging switch for developer.",
  "hume.streaming.skip_processed": "bool, optional: When enabled, we'll keep a record of processed (document_id, ontology_id) tuples in a file under hume.tmp_dir/processed.log and if we see the same kafka reading request, we'll ignore it. ",
  "hume.use_regrounding_cache": "bool, optional: When enabled, the system will save intermediate files of processing result into a directory that if we see the same document later, we can skip certain processing steps and only kick off regrounding pipeline.",
  "hume.regrounding_cache_path": "str, required when hume.use_regrounding_cache is true: a persist directory for hosting regrounding cache. Ideally to be a persist storage that can be shared in between docker instances. Also please don't reuse grounding cache from OIAD.",
  "hume.cdr_cache_dir": "str or null, optional: When specified, Hume will use the path from inside docker to cache CDRs locally so when new reading requests come as long as it's found in local cache, it won't goto API for fetching it again"
}
```


Please copy the config file to `runtime/config.json`

##### To Run the Hume System

Run Hume with the following command:

```bash
docker run -it -v runtime:/extra docker.io/wmbbn/hume:R2022_03_21 /usr/local/envs/py3-jni/bin/python3 /wm_rootfs/git/Hume/src/python/dart_integration/streaming_processing.py
```

Upon succeessful deployment, any resulting json_ld CAGs will be uploaded back to DART at `DART_upload` and the system will continue to listen for kafka messages until you stop it.


<a id="w4"></a>
### [W4](index.html#w4) Document management + reading + integration/assembly + HMI

The difference between W3, W4 and W5 workflows is transparent to Hume. Please see the documentation for [W3](hume.html#w3) above.

<a id="w5"></a>
### [W5](index.html#w5) Document management + reading + integration/assembly + HMI + BYOD

The difference between W3, W4 and W5 workflows is transparent to Hume. Please see the documentation for [W3](hume.html#w3) above.
