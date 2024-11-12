## Choose dataset

1. Go to [Swiss Open Data Portal](https://opendata.swiss/en/dataset?q=electric&linked_data=SPARQL&sort=score+desc%2C+metadata_modified+desc)
2. Choose a dataset
3. Go to `SPARQL Endpoint with graph preselection` of the `Resources` section
4. Copy cURL command from share button

### Median electricity tariff per canton

1. Download the dataset from [Median electricity tariff per canton](https://opendata.swiss/en/dataset/median-strompreis-per-kanton) by running the following command in the root directory:

   ```bash
   curl -X POST \
       -H "Content-Type: application/sparql-query" \
       --data "PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
         PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
         CONSTRUCT {
           ?s ?p ?o .
         }
         FROM <https://lindas.admin.ch/elcom/electricityprice>
         WHERE {
           ?s ?p ?o .
         }" \
       https://lindas.admin.ch/query > ./data/datasets/electricityprice.ttl
   ```

## Choose a Triplestore

- [Apache Jena Fuseki](https://jena.apache.org)
- [OpenLink Virtuoso](https://virtuoso.openlinksw.com)

### Apache Jena Fuseki

1. Run the following commands from the root directory:

   ```bash
   mkdir -p ./data/databases
   docker run --rm -v "$(pwd)":/tmp stain/jena tdb2.tdbloader --loc /tmp/data/databases/electricityprice /tmp/data/datasets/electricityprice.ttl
   ```

2. Spin up the fuseki infrastructure by running the `docker-compose-fuseki.yaml` file from the `infrastructure` directory:

   ```bash
   docker compose -f docker-compose-fuseki.yaml up -d
   ```

3. Visualize the triplestore by navigating to the local instance of graph-explorer running at [http://localhost:80/explorer](http://localhost:80/explorer)

4. Set up a connection to the triplestore by providing the following details:

   - **Name**: <name-of-your-choice>
   - **Graph Type**: SPARQL - RDF (Resource Description Framework)
   - **Public or Proxy Endpoint**: http://localhost:3030/electricityprice

5. Query the triplestore by navigating to the local instance of YASGUI running at [http://localhost:3000](http://localhost:3000)

### Virtuoso

1. Spin up the virtuoso infrastructure by running the `docker-compose-virtuoso.yaml` file from the `infrastructure` directory:

   ```bash
   docker compose -f docker-compose-virtuoso.yaml up -d
   ```

2. Load the dataset into the triplestore by running the following command:

   ```bash
   docker exec -it virtuoso isql 1111 dba admin exec="DB.DBA.TTLP(file_to_string_output('/usr/share/proj/electricityprice.ttl'), '', 'http://example.org/graph', 0); rdf_loader_run();"
   ```

3. Query the triplestore by navigating to the local instance of [Virtuoso SPARQL Query Editor](http://localhost:3030/sparql)

## Query the Triplestore

- All unique dimensions:

  ```bash
  PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
  PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

  SELECT DISTINCT ?dimension WHERE {
    ?observation <https://cube.link/observation> ?obs .
    ?obs ?dimension ?value .
  }
  ```

- Total tariff per canton:

  ```bash
  PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
  PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

  SELECT ?canton ?total WHERE {
    ?observation <https://cube.link/observation> ?obs .
    ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/canton> ?canton .
    ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/total> ?total .
  }
  ```

- Total tariff per municipality:

  ```bash
  PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
  PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

  SELECT ?municipality ?total WHERE {
    ?observation <https://cube.link/observation> ?obs .
    ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/municipality> ?municipality .
    ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/total> ?total .
  }
  ```

- Total tariff per canton per period:

  ```bash
  PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
  PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

  SELECT ?canton ?period ?total WHERE {
    ?observation <https://cube.link/observation> ?obs .
    ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/canton> ?canton .
    ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/period> ?period .
    ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/total> ?total .
  }
  ```

- Median tariff per canton:

  ```bash
  PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
  PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

  SELECT ?canton (AVG(?total) AS ?median) WHERE {
    ?observation <https://cube.link/observation> ?obs .
    ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/canton> ?canton .
    ?obs <https://energy.ld.admin.ch/elcom/electricityprice/dimension/total> ?total .
  }
  GROUP BY ?canton
  ```
