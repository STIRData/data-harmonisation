# STIRData data harmonisation toolset
The STIRData harmonisation toolset consists of two data transformation tools, [LinkedPipes ETL] and [D2RML].
The Czech business registry data harmonisation pipeline and the validation service are implemented using LinkedPipes ETL, while the other data harmonisation of the other business registries is handled by D2RML.
As a triplestore to store the resulting data, [OpenLink Virtuoso Open-Source] is used and recommended, however, other triplestores can be used as well, given that the data harmonisation processes are adjusted.
To deploy the data harmonisation workflow, the data harmonisation tools need to be deployed and populated with the prepared data harmonisation pipelines and mappings.

## Deployment of the tools
In this section, we describe how to deploy the data harmonisation tools using Docker.

### LinkedPipes ETL
LinkedPipes ETL [Docker] based deployment requires [Docker Compose].
Then, it can be deployed from the `main` branch like this:
```
curl https://raw.githubusercontent.com/linkedpipes/etl/main/docker-compose.yml | docker-compose -f - up
```
When deployed, LP-ETL runs on http://localhost:8080 and is ready to import pipelines.

For custom deployments, see the [full deployment documentation](https://github.com/linkedpipes/etl/tree/main#installation-and-startup). Once deployed, see the [user documentation](https://etl.linkedpipes.com/documentation/) and [tutorials](https://etl.linkedpipes.com/tutorials/).
The [documentation of the individual LP-ETL components](https://etl.linkedpipes.com/components/) is also available directly from the component's configuration dialog.

### D2RML
TODO: docker-based deployment

## Deployment of data harmonisation pipelines and mappings
In this section, we describe how to deploy the data harmonisation workflows in the deployed tools.

### Czech business registry
The harmonisation of the Czech business registry dataset is documented and the pipelines published in a [separate repository](https://github.com/STIRData/czech-br).
The [individual pipelines](https://github.com/STIRData/czech-br/tree/main/assets/pipelines) can be directly imported using their raw GitHub URLs and LP-ETL's import pipeline from URL functionality:
1. [Source data to Czech ontology](https://raw.githubusercontent.com/STIRData/czech-br/main/assets/pipelines/sgov.jsonld)
2. [Czech ontology to STIRData specification](https://raw.githubusercontent.com/STIRData/czech-br/main/assets/pipelines/stir.jsonld)
3. [SKOSification of CZ-NACE codes](https://raw.githubusercontent.com/STIRData/czech-br/main/assets/pipelines/cz-nace.jsonld)
4. [Mapping of Czech companies to NACE codes](https://raw.githubusercontent.com/STIRData/czech-br/main/assets/pipelines/nace-mapping.jsonld)
5. [Load data to Virtuoso](https://raw.githubusercontent.com/STIRData/czech-br/main/assets/pipelines/virtuoso.jsonld) - this needs to be adjusted to where the target triplestore instance is.

The pipelines can be configured to run periodically using, e.g., `cron`, `curl` and the [LinkedPipes ETL API](https://github.com/linkedpipes/etl/wiki/Pipeline-execution-with-curl).

### Greek ...

### Belgian ...

## Validation service
A validation service profiling the published datasets with regards to the [STIRData specification] is deployed using LinkedPipes ETL and produces a validation report in [HTML](https://stirdata.opendata.cz/validation/), [RDF Turtle](https://stirdata.opendata.cz/validation/report.ttl) and [CSV](https://stirdata.opendata.cz/validation/report.csv).

The [validation service pipeline](validation.jsonld) is also directly deployable in a LinkedPipes ETL instance.
Again, the validation pipeline can be configured to run periodically using, e.g., `cron`, `curl` and the [LinkedPipes ETL API](https://github.com/linkedpipes/etl/wiki/Pipeline-execution-with-curl).

[Docker]: https://www.docker.com/
[Docker Compose]: https://docs.docker.com/compose/
[LinkedPipes ETL]: https://etl.linkedpipes.com
[D2RML]: http://islab.ntua.gr/ns/d2rml/
[OpenLink Virtuoso Open-Source]: https://github.com/openlink/virtuoso-opensource
[STIRData specification]: https://stirdata.github.io/data-specification/