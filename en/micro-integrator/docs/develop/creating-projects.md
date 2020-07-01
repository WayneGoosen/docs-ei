# Creating Integration Projects

Create the project directories that are required for storing the various synapse artifacts that build your integration use case.

!!! Tip
    If you want to simultaneously create all the projects required for your use case, read about the [integration project solution](../../develop/creating-project-solution).

## Config Project

This project directory stores the synapse configurations (artifacts) that are used when defining a mediation flow.

<ol>
    <li>
        Open <b>WSO2 Integration Studio</b> and click <b>Miscellaneous → Create Config Project</b> in the <b>Getting Started</b> view as shown below.</br></br><img src="../../assets/img/create_project/new_config_project.png">
    </li> 
    <li>
        In the dialog box that opens, select <b>New ESB Config Project</b> and click <b>Next</b>.
    </li>
    <li>
        Enter a name for the Config project.
    </li>
    <li>
        Click <b>Finish</b> and see that the project is now listed in the project explorer.
    </li>
</ol>

You can now start creating the [integration artifacts](../develop/creating-artifacts/creating-an-api.md) in your Config project.

## Registry Resource Project

Create this project directory if you want to create registry resources for your mediation flow. You can later use these registry artifacts when you define your mediation sequences in the Config project.
            
<ol>
    <li>
        Open <b>WSO2 Integration Studio</b> and click <b>Miscellaneous → Create Registry Project</b> in the <b>Getting Started</b> view as shown below.</br></br><img src="../../assets/img/create_project/new_registy_project.png">
    </li>
    <li>
        In the dialog box that opens, enter a name for the registry project. 
    </li>
    <li>
        Click <b>Finish</b> and see that the project is now listed in the project explorer.  
    </li>
</ol>

See the instructions on <a href="../../develop/creating-artifacts/creating-registry-resources/#creating-registry-resources">creating and using registry artifacts</a>.

## Data Services Project

Create this project directory to start creating data services (.dbs files) for exposing various datasources as a service.</br>
<ol>
    <li>
        Open <b>WSO2 Integration Studio</b> and click <b>Data Service → Create Data Service Project</b> in the <b>Getting Started</b> view as shown below.
        <img src="../../assets/img/create_project/data_services_project.png">
    </li>
    <li>
        In the dialog box that opens, enter a project name and click <b>Next</b>.
    </li>
    <li>
        Click <b>Finish</b> and see that the project is now listed in the project explorer.
    </li>
</ol>

You can now start <a href="../../develop/creating-artifacts/data-services/creating-data-services/">creating data service artifacts</a> using <b>WSO2 Integration Studio</b>.

## Datasource Project

Create this project directory to start creating datasources that you can expose through a data service.

1. Open **WSO2 Integration Studio** and click **Data Service → Create Data Source** in the **Getting Started** view as shown below.
    <img src="../../assets/img/create_project/datasource-project.png">
2. In the dialog that opens, enter a project name and click **Next**.
    <img src="../../assets/img/create_project/datasource-project-dialog.png">
3. Click **Finish** and see that the project is now listed in the project explorer.
    <img src="../../assets/img/create_project/datasource-project-explorer.png" width="400">

## Connector Exporter Project

Create this project directory if you have used **Connectors** in your medition sequence (defined in the Config project). All connector artifacts need to be stored in a connector exporter project before packaging. See the instructions on [using connectors](../../develop/creating-artifacts/adding-connectors).

## Composite Application Project

This project directory allows you to package all the artifacts (stored in other integration projects) into one composite application (C-APP). This C-APP can then be deployed in the Micro Integrator server. See the instructions on [packaging integration artifacts](../../develop/packaging-artifacts).

## Alternative: Creating Projects

You can use the above integration projects and other various projects as follows:

1.  Right-click on the **Project Explorer** and click **New → Project** as shown below.
    ![Create new project](../assets/img/create_project/new_project_root.png)
2.  In the **New Project** dialog that opens, select the required project.  
    ![Create new project dialog](../assets/img/create_project/new_project_root_dialog.png)
