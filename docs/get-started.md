---
title: Getting Started
layout: documentation
---
## Getting Started

<br>

### What is PADAS?
--8<-- "whatispadas.md"
<br><br><br>

---

### Components
Padas has 2 main components:
1. **Manager UI**: All configuration changes (CRUD - Create, Read, Update, Delete operations) can be performed through Manager web interface.  This is an optional but recommended component to manage configurations through Engine API.
2. **Engine**: Reads configurations from existing Padas topics and runs assigned (based on `group` setting) and enabled topologies.  Each topology reads from a single source topic, runs one or more pipeline(s), and writes the resulting outputs to one or more output topic(s).  Each pipeline consists of one or more task(s) where each task can perform a filter, transform, enrichment, or detection (rules) function.  Please see below for details on concepts.

A Manager UI can be configured to connect to a single Engine component.  Engine components can be scaled up or down as needed with group assignments to distribute work-load.
<p class="text-center"><img src="/assets/img/padas_design_details.png" class="img-fluid py-5"></p>

### Basic Concepts
Let's take a closer look at Padas configuration and engine's processing concepts.  At a high-level, Padas Engine reads an input topic, processes data (pipelines and tasks) and writes to one or more output topics.

<p class="text-center"><img src="/assets/img/padas_topology_pipeline_task.png" width="67%"></p>

**Topologies, Pipelines, Tasks**

---

### <a name="quick-start"></a>
<br>
### Quick start
This quick start guide assumes all components (Confluent Kafka and Padas) will be installed on the same machine.  In production, it is recommended to separate out these components on different nodes/hosts.

This quickstart consists of the following steps:
1. [**Step 1**](#step1): Download and define components
2. [**Step 2**](#step2): Start Manager
3. [**Step 3**](#step3): Start Detect Engine
4. [**Step 4**](#step4): Start Transform Engine
5. [**Step 5**](#step5): Generate Events
6. [**Step 6**](#step6): View Alerts

<br>
#### Prerequisites
- Internet connectivity
- Supported [Operating System](/docs/installation.html#operating-systems)
- A supported version of [Java](https://www.oracle.com/technetwork/java/javase/downloads/index.html). Java 8 and Java 11 are supported in this version.
- Confluent Kafka must be installed and running (locally) as described in [Quick Start for Confluent Platform](https://docs.confluent.io/platform/current/quickstart).  You should have at least the following services up and running.
    ```sh
    confluent local services status
    ...
    Kafka is [UP]
    Schema Registry is [UP]
    ZooKeeper is [UP]
    ...
    ```

<br>
#### Overview of Quickstart
Below diagram shows what will be accomplished with this quick start guide.

 <img src="/assets/img/padas_quickstart_setup.png" width="67%">

### <a name="step1"></a>
<br>

**Step 1**: Download and define components
1. [Download](/index.html#download) the latest version (e.g. `padas-{{ site.data.versions.latest_version }}.tgz`)
2. Use the `tar` command to decompress the archive file

    ```sh
    tar -xvzf padas-{{ site.data.versions.latest_version }}.tgz
    ```
3. Since we have everything on a single host, make a copy of the extracted folder for manager, transform engine, and detect engine

    ```sh
    cp -r padas padas-manager
    cp -r padas padas-transform
    mv padas padas-detect
    ```
    **NOTE**: Last renaming step is not necessary but gives a descriptive name to the folder's functionality.
4. Edit manager properties (`padas-manager/etc/padas.properties`) to make sure the `padas.instance.role` is set to `manager` and `padas.license` is set to the license you received.

    ```sh
    vi padas-manager/etc/padas.properties
    ...
    ```

    After editing, properties file (`padas-manager/etc/padas.properties`) entries should be:
    ```properties
    padas.instance.role=manager
    bootstrap.servers=localhost:9092
    schema.registry.url=http://localhost:8081
    padas.license=<LICENSE KEY SHOULD GO HERE>
    ```
5. Edit transform properties (`padas-transform/etc/padas.properties`) to make sure the `padas.instance.role` is set to `transform`.

    ```sh
    vi padas-transform/etc/padas.properties
    ...
    ```

    After editing, properties file (`padas-transform/etc/padas.properties`) entries should be:
    ```properties
    padas.instance.role=transform
    bootstrap.servers=localhost:9092
    schema.registry.url=http://localhost:8081
    ```
6. From your current working directory, now you should have 3 PADAS folders, e.g.

    ```sh
    ls
    padas-detect   padas-manager   padas-transform
    ```
    Note that you don't have to make any configuration changes to `padas-detect` folder, as the default behavior is set to Detect Engine with localhost.

At this stage, make sure you have Confluent Kafka running locally as mentioned in prerequisites.

### <a name="step2"></a>
<br>

**Step 2**: Start Manager
1. Start manager node on the console.  The script will ask you to accept the license agreement (enter `y`) and define an administrator user to login; enter the desired password to continue
```
cd padas-manager/
```
{% include docs/padas_manager_start_console.md %}
2. **Login**: Go to [http://localhost:9000](http://localhost:9000) and login with the credentials used in previous step (e.g. admin)

    <img src="/assets/img/login_sample.png" width="67%">

3. **Create Topics**: Upon initial login, Manager will go to Topics menu in order to create necessary Kafka topics.  

    <img src="/assets/img/topics_pre_sample.png" width="67%">
    <br/>
    Hit <span class="btn btn-padas">Save</span> button to continue with defaults.
    <br/>
    <img src="/assets/img/topics_post_sample.png" width="67%">

4. **Create a Rule**: Go to <span class="fw-bold"><i class="bi bi-file-ruled"></i>Rules</span> menu link in order to add a sample rule.  Enter the following values for the required fields:
    - **Rule Name**: `Test Rule`
    - **PDL Query**: `field1="value"`
    - **Datamodel List**: `testdm`

    Other provided fields are optional but feel free to review and add/modify as needed.  A list of rules for MITRE ATT&CK can be found here: [padasRules.json](/assets/config/padasRules.json)

    <br/>
    Hit <span class="btn btn-padas">Save</span> button to continue. 

    <img src="/assets/img/rules_pre_sample.png" width="67%">

    <br/>
    You should be able to view the rule you specified, similar to the following screenshot.

    <img src="/assets/img/rules_post_sample.png" width="67%">

5. **Add a Transformation**: Go to <span class="fw-bold"><i class="bi bi-filter"></i>Properties</span> and first hit <span class="btn btn-padas">Edit</span>, then select <span class="btn btn-padas"><i class="bi bi-plus-lg"></i>Add New Transformation</span>.  Expand "Input Topic: 0" and enter the following values for the required fields:
    - **Topic Name**: `testtopic`
    - **Datamodel Name**: `testdm`

    <br/>
    <img src="/assets/img/props_pre_sample.png" width="67%">

    <br/>
    You should be able to view the newly added property (Input Topic: testtopic, similar to the following screenshot.

    <img src="/assets/img/props_post_sample.png" width="67%">

### <a name="step3"></a>
<br>

**Step 3**: Start Detect Engine
1. Start Detect Engine on the console (separate window, since Manager is running on the console as well).  The script will ask you to accept the license agreement (enter `y`). 
```
cd padas-detect/
```
{% include docs/padas_detect_start_console.md %}

### <a name="step4"></a>
<br>

**Step 4**: Start Transform Engine
1. Before starting Transform Engine we must first create the specified input topic (i.e. `testtopic`) in Kafka.  You can do this from [Confluent Control Center](https://docs.confluent.io/platform/current/control-center/topics/create.html) or from the console as shown below.

    ```sh
    kafka-topics --create --bootstrap-server localhost:9092 --topic testtopic --partitions 1 --replication-factor 1
    Created topic testtopic.
    ```
    
2. Start Transform Engine on the console (separate window, since Manager and Detect Engine are running on the console as well).  The script will ask you to accept the license agreement (enter `y`). 
```
cd padas-transform/
```
{% include docs/padas_transform_start_console.md %}

### <a name="step5"></a>
<br>

**Step 5**: Generate Sample Event

1. Let's generate a sample event with a simple JSON message.  Note that this JSON will match the PDL (`field1="value1"`) specified above.
```sh
echo '{"field1":"value1","field2":"value1"}' |  kafka-console-producer --bootstrap-server localhost:9092 --topic testtopic
```

### <a name="step6"></a>
<br>

**Step 6**: View Alerts

1. Once the sample event is ingested, PADAS Detect Engine will run the rules for matching datamodels in real-time and populate `padas_alerts` topic with matching event and alert information.  You can simply view this alert with the following command:

    ```sh
    kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic padas_alerts --from-beginning | jq
    ```

    Output will be similar to the following.  Note the use of `jq` above for pretty display of JSON data.
    ```json
    {
      "timestamp": "2021-11-28T14:25:23.199+0300",
      "name": "Test Rule",
      "description": "",
      "references": null,
      "customAnnotations": null,
      "mitreAnnotations": null,
      "platforms": {
        "string": ""
      },
      "domain": "mitre_attack",
      "analyticType": {
        "string": ""
      },
      "severity": {
        "string": ""
      },
      "datamodelReferences": null,
      "events": [
        {
          "timestamp": "2021-11-28T14:18:30.309+0300",
          "datamodel": "testdm",
          "source": "unknown",
          "host": "padas.local",
          "src": null,
          "dest": null,
          "user": null,
          "rawdata": "{\"field1\":\"value1\",\"field2\":\"value1\"}",
          "jsondata": "{\"field1\":\"value1\",\"field2\":\"value1\"}"
        }
      ]
    }
    ```

### <a name="nextsteps"></a>
<br>


**Next Steps**:
- <a href="/docs/installation.html">Install</a> in production.
- <a href="/docs/user-guide.html">Utilize PADAS</a> with out-of-the-box [padasRules.json](/assets/config/padasRules.json)
- <a href="/docs/admin-guide.html#integrate-to-external-systems">Integrations</a> with ingest pipelines ([Sample Sysmon Config with Winlogbeat](/assets/config/sysmonconfig-export-exclude-winlogbeat.xml)) and ready-to-use transformations ([Winlogbeat Sysmon and Security](/assets/config/padas_transformation.properties))