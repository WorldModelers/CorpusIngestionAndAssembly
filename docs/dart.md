---
title: DART
has_children: false
nav_order: 3
---
# DART

The Data Analytics and Reasoning Toolkit (DART) is the data ingestion pipeline for the World Modelers platform. 
Its primary function is to extract metadata and text from source documents, convert these data to a normalized 
data format, and pass this normalized data on to the reader technologies that can identify and extract causal 
relations for use in Causemos. It provides APIs and user interfaces for document submission, retrieval, and 
search, and it handles storage and retrieval of reader outputs. Finally, it provides APIs and user-facing tools 
for managing and developing the ontologies used by the readers to ground causal extractions. 

[Architecture diagram goes here]

The DART code repositories [can be found here](https://github.com/twosixlabs-dart).

## Workflows

<a id="w3"></a>
### [W3](index.html#w3) Document management + reading + integration/assembly

#### Running DART

[Documentation on running DART goes here]

#### Tenant Management

Prior to submitting documents, the program  manager should decide whether the intended use case demands any 
logical separation of documents or ontologies. This might be required if multiple unrelated use cases need to 
be supported in parallel or if a single use case requires considering multiple distinct knowledge bases in 
isolation with different ontologies for each knowledge base.

Logical groupings of documents and ontologies in DART are called "tenants." Every DART instance has a "global" 
tenant, which always consists of every document in the system. By default, the global tenant is the only tenant. 
The program manager can add tenants using DART CLI:

```bash
dart -p [profile] tenants add [tenant-name]
```

To retrieve a current list of available tenants:

```bash
dart -p [profile] tenants ls
```

#### Ontology Management

Once any tenants have been defined in DART, it is necessary to provide them with ontologies so that submitted 
documents can be read and assembled. DART will not notify readers of documents ingested to a tenant without 
an ontology; any such documents will therefore not propagate through reading and assembly. This may be desired 
if the user wishes to delay ontology development and reading until a suitable corpus has been defined (see 
Corpus Curation, below). In that case, DART will propagate all documents the to readers at the 
time that user first publishes an ontology to a tenant.

To manage DART's ontologies, the user can navigate to the `Concepts Explorer` tool in DART-UI. This tool can 
be accessed by opening the toolbar menu and clicking "Concepts Explorer" or by navigating to `[base-url]/concepts`. 

[Graphic showing how to access concepts explorer]

`Concepts Explorer` provides an ontology editor, which will initially be blank, and a panel allowing the user to 
access ontologies from each tenant as well as ontologies saved as user data for editing before publishing to a 
tenant. In the absence of any existing ontology, the user can either build one from scratch or upload an existing 
ontology in `yaml` format. (See [World Modelers Ontologies](https://github.com/worldmodelers/Ontologies).) To 
upload an ontology, click `Choose Ontology File,` and then `Upload Ontology` once you have selected the local 
ontology file.

[Graphic highlighting ontology upload buttons]

To edit the ontology please review the [Ontology-In-A-Day (OIAD) documentation](???), which explains how to use 
`Concepts Explorer` to manually edit an ontology and execute a machine-assisted ontology curation workflow.

Once an acceptable ontology is loaded in the editor, the user can publish it to a tenant by doing the following:

1. Find the desired tenant in the top-left panel labeled "Tenant Ontologies"
2. Click "Stage current"
3. Click "Publish staged"

[Graphic showing ontology publication]

#### Document Submission

Once an ontology has been published to a tenant. Any document uploaded to that tenant will automatically propagate 
to the readers and will be assembled. Documents can be uploaded in two ways:

##### 1. Command line submission

To upload one or more documents via DART-CLI, use the forklift command:

```bash
dart -p [profile] --tenant [tenant-name] forklift submit [file 1] [file 2] ...
```

To upload an entire directory of files:

```bash
dart -p [profile] --tenant [tenant-name] forklift submit --input-dir [directory]
```

Various kinds of metadata can be submitted along with the file, which will be incorporated into the document 
metadata within DART and propagated with document to the rest of the World Modelers system. This metadata 
can be specified via command-line options:

```bash
dart -p [profile] --tenant [tenant-name] forklift submit --genre news-article --label some-label --label another-label  --input-dir [directory]
```

##### 2. Web interface submission

To upload a component via DART-UI,








#### Corpus Curation


<a id="w4"></a>
### [W4](index.html#w4) Document management + reading + integration/assembly + HMI

Documentation...

<a id="w5"></a>
### [W5](index.html#w5) Document management + reading + integration/assembly + HMI + BYOD

Documentation...
