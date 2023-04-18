# STIRData data harmonisation toolset
The STIRData harmonisation toolset consists of two data transformation tools, [LinkedPipes ETL]() and [D2RML]().
The Czech business registry data harmonisation pipeline and the validation service are implemented using LinkedPipes ETL, while the other data harmonisation of the other business registries is handled by D2RML.
As a triplestore to store the resulting data, [OpenLink Virtuoso Open-Source]() is used and recommended, however, other triplestores can be used as well, given that the data harmonisation processes are adjusted.
To deploy the data harmonisation workflow, the data harmonisation tools need to be deployed and populated with the prepared data harmonisation pipelines and mappings.

## Deployment of the tools
In this section, we describe how to deploy the data harmonisation tools using Docker.

### LinkedPipes ETL
Docker-based deployment from https://github.com/linkedpipes/etl#docker

### D2RML
TODO: docker-based deployment

## Deployment of data harmonisation pipelines and mappings
In this section, we describe how to deploy the data harmonisation workflows in the deployed tools.

### Czech business registry
The LinkedPipes ETL pipelines are published and documented in a [separate repository](https://github.com/STIRData/czech-br).

### Greek ...

### Belgian ...

## Validation service
A validation service profiling the published datasets with regards to the [STIRData specification](https://stirdata.github.io/data-specification/) is deployed using LinkedPipes ETL and produces a validation report [available as a CSV file]().

The [validation service pipeline]() is also directly deployable in a LinkedPipes ETL instance.