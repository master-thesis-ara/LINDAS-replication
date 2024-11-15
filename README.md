# LINDAS Replication

## Table of Contents

- [Requirements](#requirements)
- [Setup](#setup)
- [Working with Datasets](#working-with-datasets)
  - [Setting up the Electricity Price Dataset](#setting-up-the-electricity-price-dataset)
- [Triplestore Options](#triplestore-options)
  - [Option 1: Apache Jena Fuseki](#option-1-apache-jena-fuseki)
  - [Option 2: Virtuoso](#option-2-virtuoso)
- [Example SPARQL Queries](#example-sparql-queries)
  - [List Unique Dimensions](#list-unique-dimensions)
  - [Tariff Analysis](#tariff-analysis)

## Requirements

- Docker and Docker Compose
- cURL

## Setup

```bash
git clone https://github.com/master-thesis-ara/LINDAS-replication.git
cd LINDAS-replication
mkdir -p ./data
```

## Working with Datasets

### Setting up the Electricity Price Dataset

1. Create dataset directory:

```bash
mkdir -p ./data/datasets
```

2. Download the dataset:

```bash
curl -X POST \
  -H "Content-Type: application/sparql-query" \
  --data "PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
    CONSTRUCT { ?s ?p ?o . }
    FROM <https://lindas.admin.ch/elcom/electricityprice>
    WHERE { ?s ?p ?o . }" \
  https://lindas.admin.ch/query > ./data/datasets/electricityprice.ttl
```

## Triplestore Options

### Option 1: Apache Jena Fuseki

1. Load data:

```bash
mkdir -p ./data/databases
docker run --rm -v "$(pwd)":/tmp stain/jena tdb2.tdbloader --loc /tmp/data/databases/electricityprice /tmp/data/datasets/electricityprice.ttl
```

2. Start services:

```bash
docker compose -f ./infrastructure/docker-compose-fuseki.yaml up -d
```

Access points:

- YASGUI Query Interface: [http://localhost:3000](http://localhost:3000)
- Fuseki Admin: [http://localhost:3030/](http://localhost:3030/) (User: `admin`, Password: `admin`)
- Graph Explorer: [http://localhost:80/explorer](http://localhost:80/explorer)

### Option 2: Virtuoso

1. Start services:

```bash
docker compose -f ./infrastructure/docker-compose-virtuoso.yaml up -d
```

2. Load data:

```bash
docker exec -it virtuoso isql 1111 dba admin exec="DB.DBA.TTLP(file_to_string_output('/usr/share/proj/electricityprice.ttl'), '', 'http://example.org/graph', 0); rdf_loader_run();"
```

Access point:

- Query Editor: [http://localhost:3030/sparql](http://localhost:3030/sparql)

## Example SPARQL Queries

### List Unique Dimensions

```sparql
SELECT DISTINCT ?dimension WHERE {
  ?observation <https://cube.link/observation> ?obs .
  ?obs ?dimension ?value .
}
```

### Tariff Analysis

```sparql
# Canton Total Tariffs
SELECT ?canton ?total WHERE {
  ?observation <https://cube.link/observation> ?obs .
  ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/canton> ?canton .
  ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/total> ?total .
}

# Municipality Total Tariffs
SELECT ?municipality ?total WHERE {
  ?observation <https://cube.link/observation> ?obs .
  ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/municipality> ?municipality .
  ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/total> ?total .
}

# Canton Median Tariffs
SELECT ?canton (AVG(?total) AS ?median) WHERE {
  ?observation <https://cube.link/observation> ?obs .
  ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/canton> ?canton .
  ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/total> ?total .
}
GROUP BY ?canton
```
