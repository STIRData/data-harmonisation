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

We created a cli for executing d2rml transformations and dockerized the whole environment so that a user can easily execute d2rml tranformations, without having to set up complex environments.

You can find the docker image for d2rml here: [d2rml-cli](https://hub.docker.com/r/stirdata/d2rml-cli)

To run it, use docker run command as following:

```
docker run -e "args=d2rml:test.ttl max_file_size:1000 output:test_out=axc" -v ${PWD}/DATA_DIRECTORY/:/data/ stirdata/d2rml-cli:latest
```

The args parameter contains all the arguments that we want to pass to the cli tool. These are the following parameters:

```
d2rml: the D2RML document to be executed (absolute path or url)
param: arguments provide parameter values in case the D2RML document is parametric
output: arguments specify where the generated triples will be saved (folder must exist)
max_file_size: the maximum number of triples in each generated output file. The default is 10000
temp_folder: a temp folder where downloaded files will be extracted if needed. It must exist.
```

The parameters should be given by providing the name and the value of each parameter.

Also, the user needs to add a volume to make the folder containing input and output files accessible inside the docker container. You do this by using the -v flag in docker, giving the full path of the folder and mapping it to the /data/ folder of the docker container:

```
-v ${PWD}/DATA_DIRECTORY/:/data/
```

Examples of STIRDATA executions:

Execution of Greek business registry 'Business registry data mapping'
(non parametric mapping):
```
docker run -e "args=d2rml:https://stirdata-semantic.ails.ece.ntua.gr/api/content/el/mapping/bdd4413c-45ec-47b8-b25a-56880a0a0b6e output:br=greece max_file_size:100000 temp_folder:tmp" -v ${PWD}/playground:/data/ stirdata/d2rml-cli:latest
```

Execution of Cypriot business registry 'GLEIF alignment mapping'
(parametric mapping):

```
docker run -e "args=d2rml:https://stirdata-semantic.ails.ece.ntua.gr/api/content/cy/mapping/616e882e-b1d5-4171-8713-457a6d659828 param:ID_PATTERN=[0-9]+ param:RAC_CODE=RA000181 param:ORGANIZATION_PREFIX=http://ee.data.stirdata.eu/resource/organization/ output:br=cyprus_gleif temp_folder:tmp max_file_size:100000" -v ${PWD}/playground:/data/ stirdata/d2rml-cli:latest
```

Both examples use the folders called "br" and "tmp" as output and temp folders respectively, which should exist under the folder called "playground", which will be mounted inside the docker container executing d2rml. Inside the output folder, the names of all .trig generated files (if more than one, depending on the max_file_size) will begin with the prefix "greece" in the first example, and with the prefix "cyprus_gleif" in the second example

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

### Greek (Athens area) business registry
The harmonization of the Greek business registry dataset is done by the following D2RML mappings:
1. [Business registry data mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/el/mapping/bdd4413c-45ec-47b8-b25a-56880a0a0b6e)
2. [Agencies data](https://stirdata-semantic.ails.ece.ntua.gr/api/content/el/mapping/e4905009-45fc-409a-b68f-6882db8b9b83)
3. [Dataset metadata](https://stirdata-semantic.ails.ece.ntua.gr/api/content/el/mapping/b0a0bb22-12ac-4ff3-8e45-cff81ade555c)

### Belgian business registry
The harmonization of the Belgian business registry dataset is done by the following D2RML mappings:
1. [Business registry data mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/be/mapping/564cb64a-2f41-4fde-a379-e091f626fa22) (credentials for accessing https://kbopub.economie.fgov.be/kbo-open-data are supplied by the parameters KBO_USERNAME, KBO_PASSWORD)
2. [Agencies data](https://stirdata-semantic.ails.ece.ntua.gr/api/content/be/mapping/951a4332-1d76-46fd-833d-bdb970a6bd1d)
3. [Dataset metadata](https://stirdata-semantic.ails.ece.ntua.gr/api/content/be/mapping/10b9e8e8-c71d-48cd-8e50-9b044f3b9286)
4. [GLEIF alignment mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/be/mapping/43413b33-622d-49c3-9202-56945514ffd7) (parameter values: ID_PATTERN = [0-9]{4}\\.[0-9]{3}\\.[0-9]{3}, RAC_CODE = RA000025, ORGANIZATION_PREFIX = http://be.data.stirdata.eu/resource/organization/.

### Cypriot business registry
The harmonization of the Cypriot business registry dataset is done by the following D2RML mappings:
1. [Business registry data mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/cy/mapping/99f57eaf-daaa-4ada-805a-d0dd30c2b029)
2. [Agencies data](https://stirdata-semantic.ails.ece.ntua.gr/api/content/cy/mapping/a3f2da09-a15a-470c-bd02-080b22b387b5)
3. [Dataset metadata](https://stirdata-semantic.ails.ece.ntua.gr/api/content/cy/mapping/3d43fdcb-78f1-4c0b-bde3-457a1b098074)
4. [GLEIF alignment mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/cy/mapping/616e882e-b1d5-4171-8713-457a6d659828) (parameter values: ID_PATTERN = [OBCNP][0-9]+, RAC_CODE = RA000161, ORGANIZATION_PREFIX = http://cy.data.stirdata.eu/resource/organization/.

### Estonian business registry
The harmonization of the Estonian business registry dataset is done by the following D2RML mappings:
1. [Business registry data mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/ee/mapping/63cbc5b7-4177-4a03-83da-37661f95ca66)
2. [Agencies data](https://stirdata-semantic.ails.ece.ntua.gr/api/content/ee/mapping/19d92714-438c-43a8-839d-eb78eb3e4950)
3. [Dataset metadata](https://stirdata-semantic.ails.ece.ntua.gr/api/content/ee/mapping/03db0332-4eaa-4051-a008-fc5b098f6cfc)
4. [GLEIF alignment mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/ee/mapping/12ebc247-8e1f-4632-8abd-c2c212b040f2) (parameter values: ID_PATTERN = [0-9]+, RAC_CODE = RA000181, ORGANIZATION_PREFIX = http://ee.data.stirdata.eu/resource/organization/.

### Finnish business registry
The harmonization of the Finnish business registry dataset is done by the following D2RML mappings:
1. [Business registry data mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/fi/mapping/bd911544-5da3-44e2-8c92-2da92e861d30)
2. [Agencies data](https://stirdata-semantic.ails.ece.ntua.gr/api/content/fi/mapping/31c3bf0b-acb8-4a57-8349-cc55b9d93012)
3. [Dataset metadata](https://stirdata-semantic.ails.ece.ntua.gr/api/content/fi/mapping/16fcfe93-44a9-4a55-8d77-8efd8a369c06)
4. [GLEIF alignment mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/fi/mapping/4afc0eb6-45f3-45eb-ae80-c179732b8a39) (parameter values: ID_PATTERN = [0-9]{7}\\-[0-9], RAC_CODE = RA000188, ORGANIZATION_PREFIX = http://fi.data.stirdata.eu/resource/organization/.

### French business registry
The harmonization of the French business registry dataset is done by the following D2RML mappings:
1. [Business registry data mapping (Legal entities)](https://stirdata-semantic.ails.ece.ntua.gr/api/content/fr/mapping/cc761b58-b679-409d-a934-7d578717bd8a)
2. [Business registry data mapping (Establishments)](https://stirdata-semantic.ails.ece.ntua.gr/api/content/fr/mapping/27dd18a8-372d-4872-a09a-af1a96f519ed)
3. [Agencies data](https://stirdata-semantic.ails.ece.ntua.gr/api/content/fr/mapping/78463fda-2281-4d75-bc31-2c1b8412807d)
4. [Dataset metadata](https://stirdata-semantic.ails.ece.ntua.gr/api/content/fr/mapping/16107c1b-0b13-4e60-aee9-b7b7ff63bdd1)
5. [GLEIF alignment mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/fr/mapping/ef0b9741-87c4-47a6-bd9c-42eaa70b9909) (parameter values: ID_PATTERN = [0-9]{9}, RAC_CODE = RA000189, ORGANIZATION_PREFIX = http://fr.data.stirdata.eu/resource/organization/.

### Latvian business registry
The harmonization of the Latvian business registry dataset is done by the following D2RML mappings:
1. [Business registry data mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/lv/mapping/9a68b683-ca7e-4e34-bf3f-5c86eb09eadb)
2. [Agencies data](https://stirdata-semantic.ails.ece.ntua.gr/api/content/lv/mapping/d9bacb8a-3294-47cd-971c-b49ff5c99dcd)
3. [Dataset metadata](https://stirdata-semantic.ails.ece.ntua.gr/api/content/lv/mapping/5046f9b5-7ae2-4835-a99c-f8228faeb67b)
4. [GLEIF alignment mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/lv/mapping/c8126c18-6598-45d5-960c-5300bffce2eb) (parameter values: ID_PATTERN = [0-9]{11}, RAC_CODE = RA000423, ORGANIZATION_PREFIX = http://lv.data.stirdata.eu/resource/organization/.

### Moldovian business registry
The harmonization of the Moldovian business registry dataset is done by the following D2RML mappings:
1. [Business registry data mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/md/mapping/5bfa8de0-85a3-4c08-b9bf-5ff68b338467)
2. [Agencies data](https://stirdata-semantic.ails.ece.ntua.gr/api/content/md/mapping/fe0f3ff4-35d9-4854-9c0e-b39a971b8161)
3. [Dataset metadata](https://stirdata-semantic.ails.ece.ntua.gr/api/content/md/mapping/9088aa31-e19b-4d36-8f7c-ca4770a479ab)
4. [GLEIF alignment mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/md/mapping/2588e3f1-4470-41b3-b570-da6712e24068) (parameter values: ID_PATTERN = [0-9]+, RAC_CODE = RA000451, ORGANIZATION_PREFIX = http://md.data.stirdata.eu/resource/organization/.

### Dutch business registry
The harmonization of the Dutch business registry dataset is done by the following D2RML mappings:
1. [Business registry data mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/nl/mapping/5b4fdac6-f562-4810-92bd-eef1e4f14043)
2. [Dataset metadata](https://stirdata-semantic.ails.ece.ntua.gr/api/content/nl/mapping/cbde3450-ec58-4d4a-a3ee-3142bbd07fc8)

### Norwegian business registry
The harmonization of the Norwegian business registry dataset is done by the following D2RML mappings:
1. [Business registry data mapping (Main units)](https://stirdata-semantic.ails.ece.ntua.gr/api/content/no/mapping/2635b100-7f4b-44ca-bcd6-93854ceb644c)
2. [Business registry data mapping (Subunits)](https://stirdata-semantic.ails.ece.ntua.gr/api/content/no/mapping/180f7097-dad5-4cc1-a4b5-9de530c5340f)
3. [Agencies data](https://stirdata-semantic.ails.ece.ntua.gr/api/content/no/mapping/b95baa81-1ce7-432c-978c-cf8a5a6e0bbd)
4. [Dataset metadata](https://stirdata-semantic.ails.ece.ntua.gr/api/content/no/mapping/e3d9a1d8-0e7f-4a2a-9827-080d9ba04d02)
5. [GLEIF alignment mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/no/mapping/aa3a2127-590d-4260-8a1f-282aabf618e2) (parameter values: ID_PATTERN = [0-9]{9}, RAC_CODE = RA000472, ORGANIZATION_PREFIX = http://no.data.stirdata.eu/resource/organization/.

### Romanian business registry
The harmonization of the Romanian business registry dataset is done by the following D2RML mappings:
1. [Business registry data mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/ro/mapping/a73f0cb2-ecc7-4387-b4c9-13321b6e7800)
2. [Agencies data](https://stirdata-semantic.ails.ece.ntua.gr/api/content/ro/mapping/7d2d8464-b375-45f2-8ed1-fd1a5d54342d)
3. [Dataset metadata](https://stirdata-semantic.ails.ece.ntua.gr/api/content/ro/mapping/ecc8137e-9dbd-4f02-aa3a-f98582b83c46)
4. [GLEIF alignment mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/ro/mapping/4584eff8-60ee-4f04-9b57-2ca95affae98) (parameter values: ID_PATTERN = [0-9]{8}, RAC_CODE = RA000497, ORGANIZATION_PREFIX = http://ro.data.stirdata.eu/resource/organization/.

### United Kingdom business registry
The harmonization of the United Kindgom business registry dataset is done by the following D2RML mappings:
1. [Business registry data mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/ro/mapping/a73f0cb2-ecc7-4387-b4c9-13321b6e7800)
2. [Agencies data](https://stirdata-semantic.ails.ece.ntua.gr/api/content/uk/mapping/e1f0a168-befa-42ba-9605-c30adabbfb31)
3. [Dataset metadata](https://stirdata-semantic.ails.ece.ntua.gr/api/content/uk/mapping/26058a10-9412-489a-9807-a5646a19e6ef)
4. [GLEIF alignment mapping](https://stirdata-semantic.ails.ece.ntua.gr/api/content/uk/mapping/9534bf41-796c-46d5-9d0b-200713f8fbe6) (parameter values: ID_PATTERN = [0-9]{8}, RAC_CODE = RA000585, ORGANIZATION_PREFIX = http://uk.data.stirdata.eu/resource/organization/.

## Validation service
A validation service profiling the published datasets with regards to the [STIRData specification] is deployed using LinkedPipes ETL and periodically produces a validation report in [HTML](https://stirdata.opendata.cz/validation/), [RDF Turtle](https://stirdata.opendata.cz/validation/report.ttl) and [CSV](https://stirdata.opendata.cz/validation/report.csv).

The [validation service pipeline](validation.jsonld) is also directly deployable in a LinkedPipes ETL instance.
Again, the validation pipeline can be configured to run periodically using, e.g., `cron`, `curl` and the [LinkedPipes ETL API](https://github.com/linkedpipes/etl/wiki/Pipeline-execution-with-curl).

[Docker]: https://www.docker.com/
[Docker Compose]: https://docs.docker.com/compose/
[LinkedPipes ETL]: https://etl.linkedpipes.com
[D2RML]: http://islab.ntua.gr/ns/d2rml/
[OpenLink Virtuoso Open-Source]: https://github.com/openlink/virtuoso-opensource
[STIRData specification]: https://stirdata.github.io/data-specification/
