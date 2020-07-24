!!! note
    This section is still a work in progress and not reviewed!

# Managing Streaming Data with Errors

## Introduction

In this tutorial, let's learn how you can handle streaming data that has errors (e.g., events that do not have values for certain attributes). WSO2 Streaming Integrator allows you to log such events, direct them to a separate stream or store them in a data store. If these errors occur at the time of publishing (e.g., due to a connection error), WSO2 SI also provides the option to wait and then resume to publish once the connection is stable again. For detailed information about different ways to handle errors, see the [Handling Errors guide](../guides/handling-errors.md).

In this scenario, you are handling erroneous events by directing them to a MySQL store.

!!! Tip "Before you begin:"
    In order to save streaming data with errors in a MySQL store, complete the following prerequisites.<br/>    
    - Start the SI server by navigating to the `<SI_HOME>/bin` directory and issuing one of the following commands as appropriate, based on your operating system:<br/>
        <br/>
          - For Windows: `streaming-integrator.bat`<br/>
        <br/>
          - For Linux:  `sh server.sh`<br/>
        <br/>
      The following log appears in the Streaming Integrator console once you have successfully started the server. <br/>
      <br/>
      `INFO {org.wso2.carbon.kernel.internal.CarbonStartupHandler} - WSO2 Streaming Integrator started in 4.240 sec`
      <br/>
    - You need to have access to a MySQL instance.<br/>
    - To simulate REST API calls, download and install [Postman](https://www.postman.com/downloads/).
    
## Tutorial steps
      
### Step 1: Create the data store

Let's create the MySQL data store in which the events with errors can be saved. To do this, follow the steps below:

1. Download the MySQL JDBC driver from [the MySQL site](https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.45.tar.gz).

2. Unzip the archive.<br/>

3. Copy the `mysql-connector-java-5.1.45-bin.jar` to the `<SI_HOME>/lib` directory.<br/>

4. Start the MySQL server as follows:

    `mysql -u <USERNAME> -p <PASSWORD>`
    
5. Create a new database named `use errorstoredb;` by issuing the following command in the MySQL console.

    ``mysql> create database errorstoredb;``
    
6. To switch to the new database, issue the following command.

    `mysql> use errorstoredb;`

### Step 2: Enable the error store

To enable the error store, open the `<SI_HOME>/conf/server/deployment.yaml` file and add a configuration as follows:

```
error.store:
  enabled: true
  bufferSize: 1024
  dropWhenBufferFull: true
  errorStore: org.wso2.carbon.streaming.integrator.core.siddhi.error.handler.DBErrorStore
  config:
    datasource: ERROR_STORE_DB
    table: ERROR_STORE_TABLE
```

This configuration refers to a data source named `Error_Store_DB`. Define this data source as follows under `Data sources` in the `<SI_HOME>/conf/server/deployment.yaml` file.

```
- name: SIDDHI_ERROR_STORE_DB
  description: The datasource used for Siddhi error handling feature
  jndiConfig:
    name: jdbc/SiddhiErrorStoreDB
  definition:
    type: RDBMS
    configuration:
      jdbcUrl: 'jdbc:mysql://localhost:3306/errorstoredb?useSSL=false'
      username: root
      password: root
      driverClassName: com.mysql.jdbc.Driver
      minIdle: 5
      maxPoolSize: 50
      idleTimeout: 60000
      connectionTestQuery: SELECT 1
      validationTimeout: 30000
      isAutoCommit: false
```

### Step 3: Create and deploy the Siddhi application

To create and deploy a Siddhi application, follow the steps below:

1. Start the Streaming Integrator Tooling by navigating to the `<SI_TOOLING_HOME>/bin` directory and issuing one of the following commands as appropriate, based on your operating system:

    - For Windows: `streaming-integrator-tooling.bat`

    - For Linux: `./streaming-integrator-tooling.sh`
    
    Then Access the Streaming Integrator Tooling via the URL that appears in the start up log with the text `Editor Started on:`.
    
2. Open a new file. Then copy and paste the following Siddhi application to it.

    ```
        @App:name("MappingErrorTest")
        
        @Source(type = 'http',
                 receiver.url='http://localhost:8006/productionStream',
                 basic.auth.enabled='false',
        	 @map(type='json', @attributes(name='name', amount='amount')))
        define stream ProductionStream(name string, amount double);
        
        @sink(type='log', prefix='Successful mapping: ')
        define stream LogStream(name string, amount double);
        
        from ProductionStream
        select *
        insert into LogStream;
    ```
3. Save the Siddhi file.

4. To deploy the Siddhi file, follow the procedure below:

    1. Click the **Deploy** menu and then click **Deploy to Server**. This opens the **Deploy Siddhi Apps to Server** dialog box.
    
    2. In the **Add New Server** section, enter the host, port, user name and the password of your Streaming Integrator server as shown below.
    
        ![Adding a New Server](../../images/handling-requests-with-errors/add-a-new-server.png)
        
        Then click **Add**.
        
    3. In the **Siddhi Apps to Deploy** section, select the checkbox for the **MappingErrorTest.siddhi** application. In the **Servers** section, select the check box for the server you added. Then click **Deploy**.
    
        ![Select Siddhi Application and Server](../../images/handling-requests-with-errors/select-siddhi-app-and-server.png)
        
        
### Step 4: Generate events with errors

Let's simulate an event with an error to observe how it is handled.

To simulate REST API calls for the purpose, follow the procedure below:

1. Download the two `.json` files from here and save them in a preferred location in your machine.

2. Open Postman. Click **Import** to open the **Import** dialog box, click **Upload Files**.

    ![Import Collections](../../images/handling-requests-with-errors/import-collections.png)

    Then browse and select the two files you downloaded. In the  **Import** dialog box that appears, click **Import**. 
    
    ![Import Collections](../../images/handling-requests-with-errors/confirm-import.png)
    
    As a result, the two collections are displayed in the left panel as follows.

    ![Imported Collections](../../images/handling-requests-with-errors/Postman.png)
    
3. Under **Siddhi-Re-Stream Events**, select **Invalid Attribute** and click **Send**. This executes the collection to send an event with erroneous mapping. The error is indicated as follows:

    `Error: connect ECONNREFUSED <HOST_NAME>:8006`

4. Under **Siddhi Re-Stream**, click **Get Erroneous Events from Error Store**. Enter `http://localhost:9090/error-handler/erroneous-events?siddhiApp=MappingErrorTest` as the URL and click **Send**.

    This generates an output event payload as shown below.
    
    ```
        [
            {
                "id": 1,
                "timestamp": 1594638613532,
                "siddhiAppName": "MappingErrorTest",
                "streamName": "InvalidMappingCaller",
                "event": "{\"foo\":\"Cake\",\"amount\":20.02}",
                "cause": "No results for path: $['name']",
                "stackTrace": "com.jayway.jsonpath.PathNotFoundException: No results for path: $['name']\n\tat com.jayway.jsonpath.internal.path.EvaluationContextImpl.getValue(EvaluationContextImpl.java:133)\n\tat com.jayway.jsonpath.JsonPath.read(JsonPath.java:187)\n\tat com.jayway.jsonpath.internal.JsonContext.read(JsonContext.java:164)\n\tat com.jayway.jsonpath.internal.JsonContext.read(JsonContext.java:151)\n\tat io.siddhi.extension.map.json.sourcemapper.JsonSourceMapper.processCustomEvent(JsonSourceMapper.java:555)\n\tat io.siddhi.extension.map.json.sourcemapper.JsonSourceMapper.convertToEvent(JsonSourceMapper.java:314)\n\tat io.siddhi.extension.map.json.sourcemapper.JsonSourceMapper.mapAndProcess(JsonSourceMapper.java:233)\n\tat io.siddhi.core.stream.input.source.SourceMapper.onEvent(SourceMapper.java:200)\n\tat io.siddhi.core.stream.input.source.SourceMapper.onEvent(SourceMapper.java:144)\n\tat io.siddhi.extension.io.http.source.HttpWorkerThread.run(HttpWorkerThread.java:62)\n\tat java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)\n\tat java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)\n\tat java.base/java.lang.Thread.run(Thread.java:834)\n",
                "errorOccurrence": "BEFORE_SOURCE_MAPPING",
                "eventType": "PAYLOAD_STRING",
                "errorType": "MAPPING"
            }
        ]
    ```
    
    This indicates that a mappinng error has occured. The reason for the mapping error is because in the input event, the attribute `name` is incorrectly replaced with `foo`.
    
5. To replay this event, do the following:

    1. Copy the output payload given above.
    
    2. Click **Siddhi Re-Stream**, and then click **ReStream Event(s)**. Paste the output event payload you copied in the body of the request. In `event": "{\"foo\":\"Cake\",\"amount\":20.02}`, replace `foo` with `name`.
    
    3. Click **Send**. As a result `Successful mapping` is logged in the console.
    
6. Start the Streaming Integrator Tooling server by navigating to the `<SI_TOOLING_HOME>/bin` directory and issuing one of the following commands as appropriate, based on your operating system:
                                                 
     - For Windows: `streaming-integrator-tooling.bat`
    
     - For Linux: `./streaming-integrator-tooling.sh`
         
    Then Access the Streaming Integrator Tooling via the URL that appears in the start up log with the text `Editor Started on:`.