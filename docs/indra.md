---
title: INDRA
has_children: false
nav_order: 6
---
# INDRA

## Workflows

<a id="w2"></a>
### [W2](index.html#w2) Reading + integration/assembly

In this workflow INDRA World processes outputs from one or more reading systems
(Eidos, Hume or Sofia) as input. We assume that each reading system was
already run and produced a number of output files. INDRA World assembly can be
done using the [command line interface](https://github.com/indralab/indra_world#command-line-interface)
either natively (if an appropriate Python environment and INDRA World and its
dependencies are installed) or through Docker.

```
indra_world --reader-output-files READER_OUTPUT_FILES --ontology-path ONTOLOGY_PATH --output-folder OUTPUT_FOLDER
```

In the above, `READER_OUTPUT_FILES` refers to a JSON file with the following
structure
```
{
 "eidos":
 [
 "/path/to/file1",
 "/path/to/file2",
 ],
 "hume": [
 ...
 ]
}
```
where keys are reading systems and values are lists of paths to their output
files.

`ONTOLOGY_PATH` is a path to a standard ontology YAML file.
`OUTPUT_FOLDER` refers to a folder in which the resulting `statements.json`
JSON-L dump of assembled INDRA Statements will be written. For more optional
arguments, see the CLI documentation.

<a id="w3"></a>
### [W3](index.html#w3) Document management + reading + integration/assembly

In this workflow, INDRA World has access to DART, which keeps track of and provides
access to documents, reader outputs, and ontologies through a standardized
API. The [INDRA World CLI](https://github.com/indralab/indra_world#command-line-interface) can be parameterized to retrieve reader outputs as well as
an ontology through the DART API, as follows.

The argument `--reader-output-dart-query READER_OUTPUT_DART_QUERY` is
a path to a JSON file in which the parameters for querying DART are
specified following the format of [this function](https://indra-world.readthedocs.io/en/latest/modules/sources/dart.html#indra_world.sources.dart.client.DartClient.get_reader_output_records). For example,

```
{
 "readers": ["eidos", "hume"],
 "ontology_id": "49277ea4-7182-46d2-ba4e-87800ee5a315"
}
```
will query DART for outputs from the `eidos` and `hume` readers produced
with the ontology ID `49277ea4-7182-46d2-ba4e-87800ee5a315`.

Alternatively, if the user wants to hand pick which specific DART reader output
records to use for assembly (instead of finding these through a structured
query as described above), the `--reader-output-dart-keys READER_OUTPUT_DART_KEYS`
argument can be used which points to a simple text file in which the unique storage
keys associated with a list of DART records are given as e.g., 

```
d6c90753-fe23-43c1-9457-8518b3eaeb65.jsonld
5b901d9a-44b9-4d57-9257-1e0881ed2d06.jsonld
...
```

In this workflow, the ontology is also registered in DART and can be specified
simply through its ID using the `--ontology-id` argument.

The output folder for INDRA is specified as described in [W2](index.html#w2).

<a id="w4"></a>
### [W4](index.html#w4) Document management + reading + integration/assembly + HMI

This workflow is different from [W3](index.html#w3) only in that we need to
make sure the INDRA World CLI produces its output in an appropriate folder structure and
output files compatible with Causemos. To achieve this, in addition to the 
`--output-folder` argument, we need to supply the `--causemos-metadata CAUSEMOS_METADATA`
argument. Here, `CAUSEMOS_METADATA` is the path to a JSON file containing
metadata that Causemos uses to identify and describe the assembled result.
The structure of this JSON file follows the `metadata.json` entry described
[here](https://indra-world.readthedocs.io/en/latest/service.html#structure-of-each-corpus).

<a id="w5"></a>
### [W5](index.html#w5) Document management + reading + integration/assembly + HMI + BYOD

This workflow is meaningfully different from W2-5 in that INDRA World here has
to be running as a service to support uploading documents in Causemos
and performing incremental assembly with respect to an existing Causemos
project based on reader output for the new documents.

To run the INDRA World service, follow the instructions [here](https://github.com/indralab/indra_world/tree/master/docker#dockerized-indra-world-service).
