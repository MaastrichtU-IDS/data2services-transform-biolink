# CWL workflows for RDF BioLink conversion

The [Common Workflow Language](https://www.commonwl.org/) is used to describe workflows to transform heterogeneous structured data (CSV, TSV, RDB, XML, JSON) to the [BioLink](https://biolink.github.io/biolink-model/docs/) RDF data model. The user defines [SPARQL queries](https://github.com/MaastrichtU-IDS/data2services-transform-biolink/blob/master/mapping/pharmgkb/insert-pharmgkb.rq) to transform the generic RDF generated depending on the input data structure (AutoR2RML, xml2rdf) to the target BioLink model.

* Install [cwltool](https://github.com/common-workflow-language/cwltool#install) to get cwl-runner.
* Install [Docker](https://docs.docker.com/install/) to run the modules.
* Those workflows use Data2Services modules, see the [data2services-pipeline](https://github.com/MaastrichtU-IDS/data2services-pipeline) project.

---

## Start services

[Apache Drill](https://github.com/amalic/apache-drill) and [GraphDB](https://github.com/MaastrichtU-IDS/graphdb/) services must be running before executing CWL workflows.

Download GraphDB as a stand-alone server free version (zip): https://ontotext.com/products/graphdb/

```shell
# Start Apache Drill sharing volume with this repository.
# Here shared locally at /data/data2services-transform-biolink
docker run -dit --rm -v /data/data2services-transform-biolink:/data:ro -p 8047:8047 -p 31010:31010 --name drill vemonet/apache-drill

# GraphDB needs to be downloaded manually and built. 
# Here shared locally at /data/graphdb and /data/graphdb-import
docker build -t graphdb --build-arg version=8.11.0 .
docker run -d --rm --name graphdb -p 7200:7200 -v /data/graphdb:/opt/graphdb/home -v /data/graphdb-import:/root/graphdb-import graphdb
```

---

## Run with [CWL](https://www.commonwl.org/)

* Go to the `data2services-transform-biolink` root folder (the root of the cloned repository)
  * e.g. `/data/data2services-transform-biolink` to run the CWL workflows.

* You will need to put the SPARQL mapping queries in `/mappings/$dataset_name` and provide 3 parameters:
  * `--outdir`: the [output directory](https://github.com/MaastrichtU-IDS/data2services-transform-biolink/tree/master/output/stitch) for files outputted by the workflow (except for the downloaded source files that goes automatically to `/input`). 
    * e.g. `output/$dataset_name`.
  * The `.cwl` [workflow file](https://github.com/MaastrichtU-IDS/data2services-transform-biolink/blob/master/support/cwl/workflow-xml.cwl)
    * e.g. `support/cwl/workflow-xml.cwl`
  * The `.yml` [configuration file](https://github.com/MaastrichtU-IDS/data2services-transform-biolink/blob/master/support/cwl/config/config-transform-xml-drugbank.yml) with all parameters required to run the workflow
    * e.g. `support/cwl/config/config-transform-xml-drugbank.yml`

* 3 types of workflows can be run depending on the input data:
  * [Convert XML to RDF](https://github.com/MaastrichtU-IDS/data2services-transform-biolink#convert-xml-with-xml2rdf)
  * [Convert CSV to RDF](https://github.com/MaastrichtU-IDS/data2services-transform-biolink#convert-csvtsv-with-autor2rml)
  * [Convert CSV to RDF and split a property]()

### Convert XML with [xml2rdf](https://github.com/MaastrichtU-IDS/xml2rdf)

```shell
cwl-runner --outdir output/drugbank support/cwl/workflow-xml.cwl support/cwl/config/config-transform-xml-drugbank.yml
```

See [config file](https://github.com/MaastrichtU-IDS/data2services-transform-biolink/blob/master/support/cwl/config/config-transform-xml-drugbank.yml):

```yaml
working_directory: /data/data2services-transform-biolink
dataset: drugbank-sample

# d2s-download params
download_username: vincent.emonet@maastrichtuniversity.nl
download_password: PASSWORD

# xml2rdf params
sparql_tmp_graph_uri: "https://w3id.org/data2services/graph/xml2rdf/drugbank"

# RdfUpload params
sparql_triplestore_url: http://graphdb:7200
sparql_triplestore_repository: test

# Execute SPARQL conversion queries
sparql_username: import_user
sparql_password: PASSWORD
sparql_output_graph_uri: https://w3id.org/data2services/graph/biolink/drugbank
sparql_service_url: "http://localhost:7200/repositories/test"

sparql_transform_queries_path: /data/mapping/drugbank
# Could be https://github.com/MaastrichtU-IDS/data2services-transform-repository/tree/master/sparql/insert-biolink/drugbank/
sparql_insert_metadata_path: https://github.com/MaastrichtU-IDS/data2services-transform-repository/tree/master/sparql/insert-biolink/drugbank/5.0
```

### Convert CSV/TSV with [AutoR2RML](https://github.com/amalic/autor2rml)

```shell
cwl-runner --outdir output/stitch support/cwl/workflow-csv.cwl support/config/config-transform-csv-stitch.yml
```

See [config file](https://github.com/MaastrichtU-IDS/data2services-transform-biolink/blob/master/support/cwl/config/config-transform-csv-stitch.yml):

```yaml
working_directory: /data/data2services-transform-biolink
dataset: stitch

# R2RML params
input_data_jdbc: "jdbc:drill:drillbit=drill:31010"
sparql_tmp_graph_uri: "https://w3id.org/data2services/graph/autor2rml/stitch"

# RdfUpload params
sparql_triplestore_url: http://graphdb:7200
sparql_triplestore_repository: test

# Execute SPARQL conversion queries
sparql_username: import_user
sparql_password: PASSWORD
sparql_output_graph_uri: https://w3id.org/data2services/graph/biolink/stitch
sparql_service_url: "http://localhost:7200/repositories/test"

sparql_transform_queries_path: /data/mapping/stitch
sparql_insert_metadata_path: https://github.com/MaastrichtU-IDS/data2services-transform-repository/tree/master/sparql/insert-biolink/stitch/1.0
```

### Convert CSV/TSV with [AutoR2RML](https://github.com/amalic/autor2rml) and split a property

```shell
cwl-runner --outdir output/eggnog support/cwl/workflow-csv-split.cwl support/cwl/config/config-transform-split-eggnog.yml
```

See [config file](https://github.com/MaastrichtU-IDS/data2services-transform-biolink/blob/master/support/cwl/config/config-transform-split-eggnog.yml):

```yaml
working_directory: /data/data2services-transform-biolink
dataset: eggnog

# R2RML params
input_data_jdbc: "jdbc:drill:drillbit=drill:31010"
sparql_tmp_graph_uri: "https://w3id.org/data2services/graph/autor2rml/eggnog"

# RdfUpload params
sparql_triplestore_url: http://graphdb:7200
sparql_triplestore_repository: test

# Split params
split_property: https://w3id.org/data2services/model/Proteinids
split_class: https://w3id.org/data2services/data/input/eggnog/NOG.members.extract101.tsv
split_delimiter: ","
split_quote: '"'

# Execute SPARQL conversion queries
sparql_username: emonet
sparql_password: PASSWORD
sparql_output_graph_uri: https://w3id.org/data2services/graph/biolink/eggnog
sparql_service_url: "http://localhost:7200/repositories/test"

sparql_transform_queries_path: /data/mapping/eggnog
sparql_insert_metadata_path: https://github.com/MaastrichtU-IDS/data2services-transform-repository/tree/master/sparql/insert-biolink/eggnog/1.0
```

### Run in the background

Will write all terminal output to `nohup.out`.

```shell
nohup cwl-runner --outdir output/drugbank support/cwl/workflow-xml.cwl support/cwl/config/config-transform-xml-drugbank.yml &
```



---

# Argo workflows

See [Argo README](https://github.com/MaastrichtU-IDS/data2services-transform-biolink/tree/master/support/argo) to run workflows with Argo.