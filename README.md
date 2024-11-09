## Choose dataset

1. Go to [Swiss Open Data Portal](https://opendata.swiss/en/dataset?q=electric&linked_data=SPARQL&sort=score+desc%2C+metadata_modified+desc)
2. Choose a dataset
3. Go to `SPARQL Endpoint with graph preselection` of the `Resources` section
4. Copy cURL command from share button

### Median electricity tariff per canton

1. Download the dataset from [Median electricity tariff per canton](https://opendata.swiss/en/dataset/median-strompreis-per-kanton) by running the following command:
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
        https://lindas.admin.ch/query > electricityprice.ttl
   ```

## Choose a Triplestore

- [Apache Jena Fuseki](https://jena.apache.org)
- [Stardog](https://www.stardog.com)

### Apache Jena Fuseki

1. Download [jena-fuseki-docker-5.2.0.zip](https://repo1.maven.org/maven2/org/apache/jena/jena-fuseki-docker/5.2.0/jena-fuseki-docker-5.2.0.zip)
2. Unzip the file in the root directory
3. Modify docker-compose.yml by adding the Jena version and the dataset path. Ensure it is like the following:

```yaml
version: "3.0"
services:
  fuseki:
    command: ["--tdb2", "--update", "--loc", "databases/DB2", "/ds"]
    build:
      context: .
      dockerfile: Dockerfile
      args:
        JENA_VERSION: 3.17.0
    image: fuseki
    ports:
      - "3030:3030"
    volumes:
      - ./logs:/fuseki/logs
      - ./databases:/fuseki/databases
```

3. Run the following commands from the `infrastructure` directory:

```bash
mkdir -p ../jena-fuseki-docker-5.2.0/databases/DB2
tdb2.tdbloader --loc ../jena-fuseki-docker-5.2.0/databases/DB2 ../datasets/electricityprice.ttl

docker compose up -d
```

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