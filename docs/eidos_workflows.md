---
title: Eidos Workflows
has_children: false
parent: Eidos
nav_order: 1
---
# Eidos Workflows

## Contents
* [Integrated system](#integrated-system)
* [Reading and assembly](#reading-and-assembly)
* [Reading only](#reading-only)

[Eidos](https://github.com/clulab/eidos) is typically incorporated into the workflows documented below.  Especially for the latter ones in which more direct access to Eidos is provided or when considering additional workflows, it may be helpful to keep in mind that Eidos is written in Scala and will run with the Scala Build Tool, [sbt](https://www.scala-sbt.org/), with the [Scala interpreter](https://www.scala-lang.org/download/install.html), or directly on the `Java` virtual machine (JVM), depending on how it has been packaged.  There is also a REST API, and it can be containerized.  For the the integrated system workflow, a [Docker](https://www.docker.com/) image that runs several instances of `Java` has been chosen as most appropriate for deployment.  It can be downloaded from [Docker hub](https://hub.docker.com/r/clulab/eidos-dart) or built locally with `sbt`.  For the reading and assembly and reading only workflows, `sbt` has been chosen as the most appropriate tool.  The [visualizer](http://eidos.cs.arizona.edu:9000/), not part of any workflow here,  makes use of the REST interface.


<a id="integrated-system"></a>
## [Integrated system](TODO)

Eidos is built into the integrated system by virtue of it being incorporated into CREATE, which organizes numerous docker images, including [one containing Eidos](https://hub.docker.com/r/clulab/eidos-dart), into a complete "ecosystem".  Eidos should therefore be pre-configured for this workflow.  How that came about is detailed further below.  To understand the configuration, one should first be familiar with how Eidos interacts with DART and Causemos, the two other components of the integrated system.

Document management for this workflow is provided by [DART](./dart.html#w3), which supplies both documents and ontologies to Eidos and then accepts output from Eidos and forwards it to INDRA for assembly.  Input documents are in CDR (Common Data Representation) format, which provides metadata such as document creation time and title, while the [JSON-LD](https://github.com/clulab/eidos/wiki/JSON-LD) output format is unchanged from other workflows.  Three handshaking stages are added to the regular Eidos reading stage.  DART provides notifications, via [Kafka](https://kafka.apache.org/) messages, of the availability of new documents and ontologies which Eidos processes with a `KafkaConsumer`.  The notifications includes details of how to download the actual documents and ontologies via a [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) interface, which Eidos accesses as a `RestConsumer`.  After the reading stage, Eidos returns results to DART via another REST interface, but this time as a `RestProducer`.

DART &rarr; KafkaConsumer &rarr; RestConsumer &rarr; Eidos &rarr; RestProducer &rarr; DART

The HMI (human machine interface) component of this workflow is provided by [Causemos](TODO), which has facilities for modifying ontologies.  As a result, requests for "regrounding" of documents are part of this workflow.  New ontologies are registered with DART and then forwarded to readers through the pipeline.  Components internal to Eidos cache intermediate results, particularly the "reading" of a document, and reuse them when an ontology changes.  The reader either reads the document or retrieves the cached reading and submits it to the grounder which takes the new ontology into account.

DART &rarr; KafkaConsumer &rarr; RestConsumer &rarr; Eidos (&rarr; reader &rarr; grounder) &rarr; RestProducer &rarr; DART

This pipeline, the major (re)configurable component of this workflow, is implemented as four independent, asynchronous processes.  That makes it awkward for the `sbt` commonly used for Scala projects.  Instead, a `Docker` container has been chosen to run four instances of Java that execute the Scala code.  Because there are quite a few settings involved, many are recorded in a shell script [export.sh](https://github.com/clulab/eidos/blob/master/wmexchanger/export.sh) where they might be edited.  These are transferred to [docker-compose-eidos2.yml](https://github.com/clulab/eidos/blob/master/wmexchanger/Docker/docker-compose-eidos2.yml) as part of starting up the container, which is done like this if Eidos is the only container running:

```shell
$ cd ./wmexchanger
$  export EIDOS_PASSWORD=<EidosPassword>
$ source ./export.sh
$ docker-compose -f ./docker-compose-eidos2.yml up
$ cd ..
```
* Here \<EidosPassword\> is whatever password is needed for Eidos to access the DART component.
* The file `export.sh` can be [downloaded](https://raw.githubusercontent.com/clulab/eidos/master/wmexchanger/export.sh) or copied from this abbreviated version and edited:
    ```shell
    export KAFKA_HOSTNAME=wm-ingest-pipeline-streaming-1.prod.dart.worldmodelers.com
    export KAFKA_CONSUMER_BOOTSTRAP_SERVERS=${KAFKA_HOSTNAME:-localhost}:9093
    export KAFKA_APP_TOPIC=dart.cdr.streaming.updates
    export REST_HOSTNAME=wm-ingest-pipeline-rest-1.prod.dart.worldmodelers.com
    export REST_CONSUMER_DOCUMENT_SERVICE=https://${REST_HOSTNAME:-localhost}/dart/api/v1/cdrs
    export REST_CONSUMER_ONTOLOGY_SERVICE=https://${REST_HOSTNAME:-localhost}/dart/api/v1/ontologies
    export REST_PRODUCER_SERVICE=https://${REST_HOSTNAME:-localhost}/dart/api/v1/readers/upload
    export EIDOS_VERSION=dart
    export ONTOLOGY_VERSION=4.0
    export EIDOS_USERNAME=eidos
    export EIDOS_BASE_DIR=../corpora/corpus
    export KAFKA_CONSUMER_SASL_JAAS_CONFIG=org.apache.kafka.common.security.plain.PlainLoginModule\ required\ username=\"$EIDOS_USERNAME\"\ password=\"$EIDOS_PASSWORD\"\;
    ```
* The file `docker-compose-eidos2.yml` can similarly be [downloaded](https://raw.githubusercontent.com/clulab/eidos/master/wmexchanger/Docker/docker-compose-eidos2.yml) or copied.  Values may in fact be entered into this file rather than being transferred from `export.sh` via environment variables if that is simpler.
    ```yaml
    version: '3.4'
    services:
      eidos:
        container_name: eidos
        hostname: eidos
        image: clulab/eidos-dart
        environment:
          # Run export.sh before using this.
          KAFKA_CONSUMER_BOOTSTRAP_SERVERS: ${KAFKA_CONSUMER_BOOTSTRAP_SERVERS}
          KAFKA_APP_TOPIC: ${KAFKA_APP_TOPIC}
          KAFKA_CONSUMER_SASL_JAAS_CONFIG: ${KAFKA_CONSUMER_SASL_JAAS_CONFIG}
          KAFKA_HOSTNAME: ${KAFKA_HOSTNAME}
          REST_CONSUMER_DOCUMENT_SERVICE: ${REST_CONSUMER_DOCUMENT_SERVICE}
          REST_CONSUMER_ONTOLOGY_SERVICE: ${REST_CONSUMER_ONTOLOGY_SERVICE}
          REST_PRODUCER_SERVICE: ${REST_PRODUCER_SERVICE}
          ONTOLOGY_VERSION: ${ONTOLOGY_VERSION}
          EIDOS_VERSION: ${EIDOS_VERSION}
          EIDOS_USERNAME: ${EIDOS_USERNAME}
          EIDOS_PASSWORD: ${EIDOS_PASSWORD}
          EIDOS_BASE_DIR: ../corpora/corpus
          
          EIDOS_MEMORY: -Xmx20g
          EIDOS_THREADS: 4
          # Another possibility is earliest.  Use latest for OIAD (Ontology In A Day).
          KAFKA_CONSUMER_AUTO_OFFSET_RESET: latest
        networks:
          - readers-net
    networks:
      readers-net:
    ```

After environment variables are processed on the host machine, they are used by the entrypoint of the running container, [start-loop-all2.sh](https://github.com/clulab/eidos/blob/master/wmexchanger/bin/start-loop-all2.sh).  There they are specialized for each of the four stages.  Some are added to the local environment and others are passed as command line arguments.  Those added to the environment are made available to the programs via the [application.conf](https://github.com/clulab/eidos/blob/master/wmexchanger/src/main/resources/application.conf) resource.  The entire chain is

export.sh &rarr; docker-compose-eidos2.yml &rarr; start-loop-all2.sh &rarr; application.conf


The image, [clulab/eidos-dart](https://hub.docker.com/r/clulab/eidos-dart), has been uploaded to `dockerhub` and should be automatically accessible to `docker-compose`.  It can be customized and rebuilt locally with any necessary changes.  The current image was build with these instructions in [dockerize.sh](https://github.com/clulab/eidos/tree/master/wmexchanger/dockerize.sh):

```shell
$ sbt "project wmexchanger" clean dist
$ cd ./wmexchanger/target/universal
$ unzip eidos-wmexchanger*.zip
$ mv eidos-wmexchanger*/bin eidos-wmexchanger*/lib .
$ mkdir eidos
$ mv ./lib/org.clulab.eidos-*.jar eidos
$ rm eidos-wmexchanger*.zip
$ rm -r eidos-wmexchanger* scripts
$ cd ../..
$ docker build -f ./Docker/DockerfileLoopDist2 -t clulab/eidos-dart .
$ cd ..
```

As mentioned previously, these instructions should only be necessary for a reconfiguration of Eidos within CREATE.  They include several assumptions about pre-installed tools.  In order to run the image, only [Docker](https://docs.docker.com/get-docker/) and [docker-compose](https://docs.docker.com/compose/install/) need to have been installed.  To build the image, one needs [sbt](https://www.scala-sbt.org/download.html), `Java`, and `unzip`, and then Eidos needs to have been downloaded, probably from [GitHub](https://github.com/clulab/eidos) with a tool like [git](https://git-scm.com/downloads) or one of its many GUIs.  Command sequences begin from the main Eidos project directory.


<a id="reading-and-assembly"></a>
## [Reading and assembly](TODO)

The assembly component in question is [INDRA](./indra.html#w2), which is able to be configured to read output files from Eidos on a filesystem.  Just get the files generated by the following reading only workflow to a place where INDRA can read them (via email attachments, thumb drive, FTP, Google Drive, RCP, etc.) and then configure INDRA appropriately.  It reads Eidos files in their native [JSON-LD](https://github.com/clulab/eidos/wiki/JSON-LD) format.  It will want to know which [ontology](https://github.com/WorldModelers/Ontologies) to use, so be sure to coordinate that between the two components.


<a id="reading-only"></a>
## [Reading only](TODO)

Eidos can run stand-alone on a single computer independently of the other World Modelers (WM) components.  In this case, the most applicable tool to use is `sbt`.  Input files are most likely plain text documents (or potentially CDRs for Common Data Representation) and for output there are various choices, but the native format is [JSON-LD](https://github.com/clulab/eidos/wiki/JSON-LD).  It is assumed that the ontology is known at build time and remains constant during a run.  Given these conditions, the best entry point to use is `ExtractTxtMetaFromDirectory` and the command is
```
$ sbt "runMain org.clulab.wm.eidos.apps.batch.ExtractTxtMetaFromDirectory <inputDir> <outputDir> <timeFile> <threadCount>"
```

If CDRs (JSON files of a particular format most associated with the integrated system workflow) are available, they are preferable because they facilitate specification of document creation time.  This helps Eidos pin down relative dates in the text.  The CDRs might be found in a sample corpus or be exported from other WM tools.  The above command need only be slightly altered for CDRs:
```
$ sbt "runMain org.clulab.wm.eidos.apps.batch.ExtractCdrMetaFromDirectory <inputDir> <outputDir> <timeFile> <threadCount>"
```

* The \<inputDir\> should contain some `*.txt` files (or `*.json` files for CDRs).  If there is not one already, a `done` subdirectory will be created inside the \<inputDir\>.  As the files are read, those completed will be moved to `done`.  If processing should need to be stopped and later restarted, it can usually continue from where it left off.
* Output files will be written to the \<outputDir\>.  It should be created in advance.  Output file names match their input counterparts except that the extension is changed to `.jsonld`.
* Statistics about runtime performance will be written to the \<timeFile\>. 
* Eidos processes documents in parallel and the number of documents is determined by the \<threadCount\>.  Depending on the computer, performance tops out at about 5 threads.  After that it is more efficient to start multiple copies of Eidos.  More threads require more memory.  It depends on document size and other factors, but one thread might work best with 12GB and five with 20GB.  One way to achieve this is to use an environment variable `_JAVA_OPTIONS=-Xmx12g`.

In this situation, the ontology to be used is built into the executable as a resource by `sbt` based on an instruction in `build.sbt`
```
"com.github.WorldModelers" % "Ontologies" % "master-SNAPSHOT"
```
so that the [latest version](https://github.com/WorldModelers/Ontologies/blob/master/CompositionalOntology_metadata.yml) in the master branch of the [Ontologies repo](https://github.com/WorldModelers/Ontologies) is automatically included.  If this is not the desired behavior, the line can be commented out and the actual file placed in
```
./src/main/resources/org/clulab/wm/eidos/english/ontologies
```
as specified in the configuration file `eidos.conf` where it can be changed if need be.

Finally, there are several implied prerequisites to these tasks.  [sbt](https://www.scala-sbt.org/download.html) needs to have been installed before these instructions will work.  `sbt` in turn requires `Java` which must be similarly installed.  Some of the libraries used work best with Java 8, so it is highly favored.  Eidos needs to have been downloaded, probably from [GitHub](https://github.com/clulab/eidos) with a tool like [git](https://git-scm.com/downloads) or one of its many GUIs.  The `sbt` command needs to be run in the Eidos project directory, the one with the `.git` subdirectory.
