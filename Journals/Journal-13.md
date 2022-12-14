# Week 13 - 19/10/2022

This week it was asked that the solution was documented as a whole. This meant that each aspect of the product were explained in the instance that someone were to look at the system and want to know how it works.
This includes such things as:

- File Structure
- File Names
- Purposes of Files
- Where to go if you want to modify a certain task (i.e. the method for making requests to a certain plugin)
- Naming conventions
- Configurations (Sitecore API Key, URL, Max visits)
- A glossary of terms.

The following was produced



# Dynamics Solution

## File Structure:

![img](https://imgur.com/7Xvjgks.png)

### Custom Entities

- **Sitecore Behavior** 
- **Sitecore Behavior Profile Key**
- **Sitecore Goal**
- **Sitecore Visit**
- **Sitecore XDB Contact**

### Custom Processes (Actions)

- **GetData**
  - Action used when request is made to get analytics details of a given lead. Triggers plugin for individual visits. Expects a `leadid` input and optionally has a `data` output. The data output for this entity would either be a success response with the Sitecore summary details object in a JSON string format, or an error response with an error message.
- **GetEvents**
  - Action used when request is made to get details of an individual visit. Triggers plugin for individual visits. Expects a `visitid` input and optionally has a `data` output. The data output for this entity would either be a success response with the events as an array (JSON string format), or an error response with an error message. 
- **RefreshAnalytics**
  - Action used when request is made to refresh the analytics. Triggers plugin for analytics refresh. Expects a `leadid` input and optionally has a `data` output. As this plugin does not return actual data, the output would only ever be a response message (success/failed)

### Custom Control (PCF Component)

- **SitecoreLeadSummaryComponent**
  - Display Name: dev_Fuseit.S4D.Dynamics.SitecoreLeadSummaryComponent
  - Does not require a set value when added to the form. Selecting a value will not do anything in the way of functionality. Selecting `lead` may cause the component to get stuck on the form as it is a required field (do not try). 

### Environment Variables

- **Sitecore Configuration**
  - Required to make outbound requests to the web server.
  - Set in the "current value" textbox.
  - Attribute naming must be exactly the same as: `{"SitecoreApiUrl" : "<URL>","SitecoreApiKey" : "<API Key>","maxVisits" : <Number of visits> }`
    - The API URL and the API Key must be a string format (between quotations " ")
    - The maxVisits must be an integer type (no decimals and no quotation marks " ")

### Other

- Contact and Lead are added when you create lookup fields on an entity.

------

## GitHub Folder

### General File Structure

```
????FuseInfoTech/S4D-Dynamics:BRANCH:dev/proof-of-concept 
???   ????README.md  
????????????????Plugins
???   ????????????????Plugin-GetAnalytics
???   ????????????????Plugin-IndividualEvents
???   ????????????????Plugin-RefreshAnalytics 
????????????????Components
???   ????????????????SitecoreLeadSummaryComponent 
????????????Tools
???   ????????????????ConfigurationMigration 
???   ????????????????CoreTools 
???   ????????????????PackageDeployment 
???   ????????????????PluginRegistration 
```

- The `Plugins` folder contains the custom plugins for performing each of the required tasks
- The `Components` folder contains the custom controls (Power Apps Components)
- The `Tools` folder contains tools used in code development. These tools include:
  - Code Generation tool `CrmSvcUtil.exe`
  - Configuration Migration tool `DataMigrationUtility.exe`
  - Package Deployer `PackageDeployer.exe`
  - Plug-in Registration tool `PluginRegistration.exe`
  - SolutionPackager tool `SolutionPackager.exe`

------

### Plugin_Retrieve_Analytics

#### Purpose

The purpose of this plugin is to retrieve visits relevant to a given Lead from the Dynamics database and perform calculations to get the statistics before returning the data to the sender.

*Source:* *https://github.com/FuseInfoTech/S4D-Dynamics/tree/dev/proof-of-concept/Plugins/Plugin-GetAnalytics*

*File Path:* *[C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Plugins\Plugin-GetAnalytics\Plugin_Retrieve_Analytics.sln](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Plugins\Plugin-GetAnalytics\Plugin_Retrieve_Analytics.sln)*

#### File Structure

**NOTE**: I have only listed the important files for this solution. These are ones that I have worked with and will likely only need to me added to/modified.

```
????Plugin_Retrieve_Analytics
| ????PluginBase.cs
|
????????????????Context
|	| ????PluginContext.cs
|
????????????????Entities
|	????????????????BaseEntities
|	|	| ????SitecoreSummary.cs
|	|
|	????????????????SitecoreEntities
|	|	| ????SitecoreVisitEntity.cs
|	|	| ????SitecoreXDBContactEntity.cs
|	|
????????????????Services
|	| ????DynamicsService.cs
|	| ????EntityService.cs
|	| ????OptionSets.cs
|
????????????????Utilities
|	| ????JsonUtil.cs
|	| ????StringUtil.cs
```

The `PluginBase.cs` file is the base class that implements the IPlugin interface, which is the base interface for a plugin. This contains the executable function that instantiates a new `PluginContext` instance that will be used throughout the plugin. This class has minimal functionality as its purpose is to call the services that handle getting the input parameters, making outbound requests, manipulating data, and defining output parameters. 

The `Context` folder contains a file called `PluginContext.cs.` This contains a class called `PluginContext` that is used for creating the services related to the Dynamics solution. This includes:

-  **_entityService:** Represents a source of entities bound to a CRM service. It tracks and manages changes made to the retrieved entities.

-  **_tracingService:** Interface for plug-ins to provide trace information. Used for debugging and/or troubleshooting plug-in issues or behaviors are complicated without rich and insightful logging or tracing.

-  **_context:** Defines the contextual information passed to a plug-in at run-time. Contains information that describes the run-time environment that the plug-in is executing in, information related to the execution pipeline, and entity business information.

-  **_factory:** Represents a factory for creating IOrganizationService instances

-  **_userOrganizationService:** Provides programmatic access to the metadata and data for an organization. Contains methods provided by Organization Service. Used in locations where user is granted access

-  **_systemOrganizationService:** Provides programmatic access to the metadata and data for an organization. Contains methods provided by Organization Service. Used in locations where only system can access.

-  **_httpClient:** Provides a class for sending HTTP requests and receiving HTTP responses from a resource identified by a URI.

The `Entities` folder contains two subfolders, `BaseEntities` and `SitecoreEntities`. The `BaseEntities` folder contain classes that are not specific to the custom entities in Dynamics or classes that are versions of Dynamics entities, containing only the attributes that we want to work with. The `SitecoreSummary` folder contains a class that returns the statistics details of a given lead as well as a list of all the visits. The `SitecoreEntities` folder contains all of the early bound entities generated by XrmToolBox.

The `Services` folder contains three different services: 

- `DynamicsService.cs`: Contains methods that handle data in relation to the Dynamics side of operations. For example, retrieving entities from entity sets.
- `EntityService.cs`: Represents a source of entities bound to a CRM service. It tracks and manages changes made to the retrieved entities.
- `OptionSets.cs`: Generated using XrmToolBox, creating enumerators for optionsets.

The `Utilities ` folder is used to store code utilities. In this case, we have one file called `JsonUtil.cs` which contains a method for deserialization JSON strings to objects and serialize objects into JSON strings. We have another file called `StringUtil.cs` which handles string conversion, null checks, and checks for whitespace.

#### Modifying the System

| Action                                                       | Name                                                | Line     | Location                    |
| ------------------------------------------------------------ | --------------------------------------------------- | -------- | :-------------------------- |
| Any details related to the plugin context. Contains information that describes the run-time environment that the plug-in is executing in, information related to the execution pipeline, and entity business information. | PluginContext.cs                                    | 12 - 129 | /Context/                   |
| Add or modify attributes of the object that is serialized and returned to the caller | SitecoreSummary.cs                                  | N/A      | /Entities/BaseEntities/     |
| Add or modify Sitecore Entities                              | SitecoreVisitEntity.cs, SitecoreXDBContactEntity.cs | N/A      | /Entities/SitecoreEntities/ |
| Modify actions relevant to Dynamics 365. This includes the calculations for the statistics, getting the visits from the entity service, and getting/setting the input and output parameters | DynamicsService.cs                                  | N/A      | /Services/                  |
| Modifying the entity service that represents a source of entities bound to a CRM service. It tracks and manages changes made to the retrieved entities. This is when you want to add or remove entity sets | EntitySevice.cs                                     | N/A      | /Services/                  |
| Add or modify existing JSON utilities                        | JsonUtil.cs                                         | N/A      | /Utilities/                 |
| Add or modify existing string utilities                      | StringUtil.cs                                       | N/A      | /Utilities/                 |
| Add or remove functionality from  the plugin base. This contains the execute method that requests the plug-in to Execute business logic. | PluginBase.cs                                       | N/A      | /                           |



------

### Plugin_Individual_Events

#### Purpose

The purpose of this plugin is to make an on-demand outbound request to the Sitecore Web Server to retrieve events relative to a given visit. The Web server returns an array of event objects containing the URL, visit type, and timestamp of the event. This data is then passed back to the sender.

*Source:* *https://github.com/FuseInfoTech/S4D-Dynamics/tree/dev/proof-of-concept/Plugins/Plugin-IndividualEvents/Plugin_Individual_Events/Plugin_Individual_Events*

*File Path:* [*C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Plugins\Plugin-IndividualEvents\Plugin_Individual_Events/Plugin_Individual_Events.sln*](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Plugins\Plugin-IndividualEvents\Plugin_Individual_Events/Plugin_Individual_Events.sln)

#### File Structure

**NOTE**: I have only listed the important files for this solution. These are ones that I have worked with and will likely only need to me added to/modified.

```
????Plugin_Individual_Events
| ????PluginBase.cs
|
????????????????Context
|	| ????PluginContext.cs
|
????????????????Entities
|	????????????????BaseEntities
|	|	| ????EventEntityBase.cs
|	|	| ????ResponseMessage.cs
|	|	| ????SitecoreConfig.cs 
|	|
|	????????????????SitecoreEntities
|	|	| ????EnvironmentVariableDefinitionEntity.cs
|	|	| ????EnvironmentVariableValueEntity.cs
|	|	| ????SitecoreVisitEntity.cs
|	|	| ????SitecoreXDBContactEntity.cs
|
????????????????Services
|	| ????DynamicsService.cs
|	| ????EntityService.cs
|	| ????OptionSets.cs
|	| ????SitecoreService.cs
|
????????????????Utilities
|	| ????JsonUtil.cs
|	| ????StringUtil.cs
```

The `PluginBase.cs` file is the base class that implements the IPlugin interface, which is the base interface for a plugin. This contains the executable function that instantiates a new `PluginContext` instance that will be used throughout the plugin. This class has minimal functionality as its purpose is to call the services that handle getting the input parameters, making outbound requests, manipulating data, and defining output parameters. 

The `Context` folder contains a file called `PluginContext.cs.` This contains a class called `PluginContext` that is used for creating the services related to the Dynamics solution. This includes:

-  **_entityService:** Represents a source of entities bound to a CRM service. It tracks and manages changes made to the retrieved entities.

-  **_tracingService:** Interface for plug-ins to provide trace information. Used for debugging and/or troubleshooting plug-in issues or behaviors are complicated without rich and insightful logging or tracing.

-  **_context:** Defines the contextual information passed to a plug-in at run-time. Contains information that describes the run-time environment that the plug-in is executing in, information related to the execution pipeline, and entity business information.

-  **_factory:** Represents a factory for creating IOrganizationService instances

-  **_userOrganizationService:** Provides programmatic access to the metadata and data for an organization. Contains methods provided by Organization Service. Used in locations where user is granted access

-  **_systemOrganizationService:** Provides programmatic access to the metadata and data for an organization. Contains methods provided by Organization Service. Used in locations where only system can access.

-  **_httpClient:** Provides a class for sending HTTP requests and receiving HTTP responses from a resource identified by a URI.

The `Entities` folder contains two subfolders, `BaseEntities` and `SitecoreEntities`. The `BaseEntities` folder contain classes that are not specific to the custom entities in Dynamics or classes that are versions of Dynamics entities, containing only the attributes that we want to work with.  The `SitecoreEntities` folder contains all of the early bound entities generated by XrmToolBox. There are three files in the `BaseEntities` folder:

- `EventEntityBase.cs`: base class for a given event. Used when serializing and deserializing JSON from the web server to the custom control.
- `ResponseMessage.cs`: base class representing a response message that is returned from the plugin in a JSON string format.
- `SitecoreConfig.cs`: base class containing attributes that match the Sitecore configuration specified in the Environment Variables

The `Services` folder contains four different services: 

- `DynamicsService.cs`: Contains methods that handle data in relation to the Dynamics side of operations. For example, retrieving entities from entity sets.
- `SitecoreService.cs`: Contains method that handle data in relation to the Sitecore side of operations. This includes making outbound requests to the web server
- `EntityService.cs`: Represents a source of entities bound to a CRM service. It tracks and manages changes made to the retrieved entities.
- `OptionSets.cs`: Generated using XrmToolBox, creating enumerators for optionsets.

The `Utilities ` folder is used to store code utilities. In this case, we have one file called `JsonUtil.cs` which contains a method for deserialization JSON strings to objects and serialize objects into JSON strings. We have another file called `StringUtil.cs` which handles string conversion, null checks, and checks for whitespace.

#### Modifying the System

| Action                                                       | Name                                                         | Line     | Location                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ | -------- | :-------------------------- |
| Any details related to the plugin context. Contains information that describes the run-time environment that the plug-in is executing in, information related to the execution pipeline, and entity business information. | PluginContext.cs                                             | 12 - 129 | /Context/                   |
| Add or modify the object that the JSON is mapped to when returned from the web server. | EventEntityBase.cs                                           | N/A      | /Entities/BaseEntities/     |
| The response message that is returned to the caller. This is a simple response status (success/failed) and a message containing a success message (action executed successfully), or an error message (e.g. Failed to contact the server) | ResponseMessage.cs                                           | N/A      | /Entities/BaseEntities/     |
| Add or modify the object that the Sitecore config is mapped to. The config is the set of environment variables retrieved from Dynamics. | SitecoreConfig.cs                                            | N/A      | /Entities/BaseEntities/     |
| Add or modify Sitecore Entities                              | EnvironmentVariableDefinitionEntity.cs, EnvironmentVariableValueEntity.cs, SitecoreVisitEntity.cs, SitecoreXDBContactEntity.cs | N/A      | /Entities/SitecoreEntities/ |
| Modify actions relevant to Dynamics 365. This includes the calculations for the statistics, getting the visits from the entity service, and getting/setting the input and output parameters | DynamicsService.cs                                           | N/A      | /Services/                  |
| Modifying the entity service that represents a source of entities bound to a CRM service. It tracks and manages changes made to the retrieved entities. This is when you want to add or remove entity sets | EntitySevice.cs                                              | N/A      | /Services/                  |
| Modify actions related to Sitecore. This includes any http requests made to the server. | SitecoreService.cs                                           | N/A      | /Services/                  |
| Add or modify existing JSON utilities                        | JsonUtil.cs                                                  | N/A      | /Utilities/                 |
| Add or modify existing string utilities                      | StringUtil.cs                                                | N/A      | /Utilities/                 |
| Add or remove functionality from  the plugin base. This contains the execute method that requests the plug-in to Execute business logic. | PluginBase.cs                                                | N/A      | /                           |



------

### Plugin_Refresh_Analytics

#### Purpose

The purpose of this plugin is to make an outbound request to the Sitecore web service, return a set of visits, determine whether they exist, and update or insert the visits into Dynamics. This plugin does not return a set of data, just a response message (success/error).

*Source:* *https://github.com/FuseInfoTech/S4D-Dynamics/tree/dev/proof-of-concept/Plugins/Plugin-RefreshSitecoreAnalytics/Plugin_Refresh_Analytics/Plugin_Refresh_Analytics*

*File Path:* [*C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Plugins\Plugin-RefreshSitecoreAnalytics\Plugin_Refresh_Analytics/Plugin_Refresh_Analytics.sln*](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Plugins\Plugin-RefreshSitecoreAnalytics\Plugin_Refresh_Analytics/Plugin_Refresh_Analytics.sln)

#### File Structure

**NOTE**: I have only listed the important files for this solution. These are ones that I have worked with and will likely only need to me added to/modified.

```
????Plugin_Refresh_Analytics
| ????PluginBase.cs
|
????????????????Context
|	| ????PluginContext.cs
|
????????????????Entities
|	????????????????BaseEntities
|	|	| ????GoalEntityBase.cs
|	|	| ????ResponseEntity
|	|	| ????ResponseMessage.cs
|	|	| ????SitecoreConfig.cs
|	|	| ????VisitEntityBase.cs
|	|
|	????????????????SitecoreEntities
|	|	| ????EnvironmentVariableDefinitionEntity.cs
|	|	| ????EnvironmentVariableValueEntity.cs
|	|	| ????SitecoreVisitEntity.cs
|	|	| ????SitecoreXDBContactEntity.cs
|
????????????????Services
|	| ????DynamicsService.cs
|	| ????EntityService.cs
|	| ????OptionSets.cs
|	| ????SitecoreService.cs
|
????????????????Utilities
|	| ????JsonUtil.cs
|	| ????StringUtil.cs
```

The `PluginBase.cs` file is the base class that implements the IPlugin interface, which is the base interface for a plugin. This contains the executable function that instantiates a new `PluginContext` instance that will be used throughout the plugin. This class has minimal functionality as its purpose is to call the services that handle getting the input parameters, making outbound requests, manipulating data, and defining output parameters.  

The `Context` folder contains a file called `PluginContext.cs.` This contains a class called `PluginContext` that is used for creating the services related to the Dynamics solution. This includes:

-  **_entityService:** Represents a source of entities bound to a CRM service. It tracks and manages changes made to the retrieved entities.

-  **_tracingService:** Interface for plug-ins to provide trace information. Used for debugging and/or troubleshooting plug-in issues or behaviors are complicated without rich and insightful logging or tracing.

-  **_context:** Defines the contextual information passed to a plug-in at run-time. Contains information that describes the run-time environment that the plug-in is executing in, information related to the execution pipeline, and entity business information.

-  **_factory:** Represents a factory for creating IOrganizationService instances

-  **_userOrganizationService:** Provides programmatic access to the metadata and data for an organization. Contains methods provided by Organization Service. Used in locations where user is granted access

-  **_systemOrganizationService:** Provides programmatic access to the metadata and data for an organization. Contains methods provided by Organization Service. Used in locations where only system can access.

-  **_httpClient:** Provides a class for sending HTTP requests and receiving HTTP responses from a resource identified by a URI.

The `Entities` folder contains two subfolders, `BaseEntities` and `SitecoreEntities`. The `BaseEntities` folder contain classes that are not specific to the custom entities in Dynamics or classes that are versions of Dynamics entities, containing only the attributes that we want to work with. The `SitecoreSummary` folder contains a class that returns the statistics details of a given lead as well as a list of all the visits. The `SitecoreEntities` folder contains all of the early bound entities generated by XrmToolBox.

The `Services` folder contains four different services: 

- `DynamicsService.cs`: Contains methods that handle data in relation to the Dynamics side of operations. For example, retrieving entities from entity sets.
- `SitecoreService.cs`: Contains method that handle data in relation to the Sitecore side of operations. This includes making outbound requests to the web server
- `EntityService.cs`: Represents a source of entities bound to a CRM service. It tracks and manages changes made to the retrieved entities.
- `OptionSets.cs`: Generated using XrmToolBox, creating enumerators for optionsets.

The `Utilities ` folder is used to store code utilities. In this case, we have one file called `JsonUtil.cs` which contains a method for deserialization JSON strings to objects and serialize objects into JSON strings. We have another file called `StringUtil.cs` which handles string conversion, null checks, and checks for whitespace.

#### Modifying the System

| Action                                                       | Name                                                         | Line     | Location                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ | -------- | :-------------------------- |
| Any details related to the plugin context. Contains information that describes the run-time environment that the plug-in is executing in, information related to the execution pipeline, and entity business information. | PluginContext.cs                                             | 12 - 129 | /Context/                   |
| Add or modify the object that the JSON is mapped to when returned from the web server. This is contained within the VisitEntityBase | GoalEntityBase.cs                                            | N/A      | /Entities/BaseEntities/     |
| Add or modify the object that the JSON is mapped to when returned from the web server. This is the base entity that contains the VisitEntityBase | ResponseEntity.cs                                            | N/A      | /Entities/BaseEntities/     |
| The response message that is returned to the caller. This is a simple response status (success/failed) and a message containing a success message (action executed successfully), or an error message (e.g. Failed to contact the server) | ResponseMessage.cs                                           | N/A      | /Entities/BaseEntities/     |
| Add or modify the object that the Sitecore config is mapped to. The config is the set of environment variables retrieved from Dynamics. | SitecoreConfig.cs                                            | N/A      | /Entities/BaseEntities/     |
| Modify the base entity that the JSON is mapped to when returned from the web server. This is contained within the ResponseEntity, and contains the GoalEntityBase | VisitEntityBase.cs                                           | N/A      | /Entities/BaseEntities/     |
| Add or modify Sitecore Entities                              | EnvironmentVariableDefinitionEntity.cs, EnvironmentVariableValueEntity.cs, SitecoreVisitEntity.cs, SitecoreXDBContactEntity.cs | N/A      | /Entities/SitecoreEntities/ |
| Modify actions relevant to Dynamics 365. This includes the calculations for the statistics, getting the visits from the entity service, and getting/setting the input and output parameters | DynamicsService.cs                                           | N/A      | /Services/                  |
| Modifying the entity service that represents a source of entities bound to a CRM service. It tracks and manages changes made to the retrieved entities. This is when you want to add or remove entity sets | EntitySevice.cs                                              | N/A      | /Services/                  |
| Modify actions related to Sitecore. This includes any http requests made to the server. | SitecoreService.cs                                           | N/A      | /Services/                  |
| Add or modify existing JSON utilities                        | JsonUtil.cs                                                  | N/A      | /Utilities/                 |
| Add or modify existing string utilities                      | StringUtil.cs                                                | N/A      | /Utilities/                 |
| Add or remove functionality from  the plugin base. This contains the execute method that requests the plug-in to Execute business logic. | PluginBase.cs                                                | N/A      | /                           |





------

### PluginRegistration

Executable: `PluginRegistration.exe`

*File Path:* [*C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Tools\PluginRegistration\PluginRegistration.exe*](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Tools\PluginRegistration\PluginRegistration.exe)

Use: Used for registering the plugins mentioned above. Use is required to register them to the Microsoft Dataverse.

![PluginRegistrationTool](https://imgur.com/vLsOgQL.png)

#### Config

| Plugin                    | Output File                                                  | Step Message         | Primary/Secondary Entity | Step Stage Of Execution |
| ------------------------- | :----------------------------------------------------------- | -------------------- | ------------------------ | ----------------------- |
| Plugin_Refresh_Analytics  | C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Plugins\Plugin-RefreshSitecoreAnalytics\Plugin_Refresh_Analytics\Plugin_Refresh_Analytics\bin\Debug\Plugin_Refresh_Analytics.dll | dev_RefreshAnalytics | none                     | PostOperation           |
| Plugin_Individual_Events  | C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Plugins\Plugin-IndividualEvents\Plugin_Individual_Events\Plugin_Individual_Events\bin\Debug\Plugin_Individual_Events.dll | dev_GetEvents        | none                     | PostOperation           |
| Plugin_Retrieve_Analytics | C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Plugins\Plugin-GetAnalytics\DataRetrievalPlugin\bin\Debug\Plugin_Retrieve_Analytics.dll | dev_GetData          | none                     | PostOperation           |

**No other configuration is required**



------

### SitecoreLeadSummaryComponent

#### Purpose

The purpose of this component is to presents Sitecore analytics data for a Lead, as referenced by `sitecorexdbcontact.leadid`.

This displays the following details:

- Simple overview statistics:
  - Total Goal value
  - Total Visit count
  - Average visit time
  - Total page view count
- A tabulated list of the Visits, including: 
  - Start date
  - Channel
  - Duration
  - Total page view count
  - Browser
  - Total Goal value
  - The visit ID which is a link that is used to view details of a single visit on-demand.
- A button or link to update analytics data for single Lead on-demand

![component](https://imgur.com/OUXBLnm.png)

*Source:* *https://github.com/FuseInfoTech/S4D-Dynamics/tree/dev/proof-of-concept/Components/SitecoreSummaryComponent*

*File Path:* [*C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent*](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent)

#### File Structure

**NOTE**: I have only listed the important files for this solution. These are ones that I have worked with and will likely only need to me added to/modified.

```
????SitecoreLeadSummaryComponent
????????????????SitecoreSummaryComponent
|	????ControlManifest.Input.xml
|	????index.ts
|	????????????????Components
|	|	| ????ComponentBase.tsx
|	|	| ????ComponentBase.tsx
|	|
|	????????????????Solutions
|	|	????????????????src
|	|	| ????????????????Other
|	|	|	| ????Solution.xml
```

- The control manifest is an XML file that contains the metadata of the code component. It also defines the behavior of the code component.

  - The control node defines the namespace, version, and display name of the code component.

    - **namespace**: Namespace of the code component.
    - **Constructor**: Constructor of the code component.
    - **Version**: Version of the component. Whenever you update the component, you need to update the version to see the latest changes in the runtime.
    - **display-name-key**: Name of the code component that is displayed on the UI.
    - **description-name-key**: Description of the code component that is displayed on the UI.
    - **control-type**: The code component type. Only *standard* types of code components are supported.

  - The property node defines the properties of the code component like defining the data type of the column. The property node is specified as the child element under the `control` element.

    - **name**: Name of the property.

    - **display-name-key**: Display name of the property that is displayed on the UI.

    - **description-name-key**: Description of the property that is displayed on the UI.

    - **of-type-group**: The `of-type-group` is used when you want to have more than two data type columns. The `of-type-group` specifies the component value and can contain whole, currency, floating point, or decimal values.

    - **usage**: Has two properties, *bound* and *input*. Bound properties are bound only to the value of the column. Input properties are either bound to a column or allow a static value.

    - **required**: Defines whether the property is required.

  - The resources node defines the visualization of the code component. It contains all the resources that build the visualization and styling of the code component. The code is specified as a child element under the resources element.

    - **code**: Refers to the path where all the resource files are located.

    The manifest file for this component looks like this:

    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <manifest>
      <control namespace="Fuseit.S4D.Dynamics" constructor="SitecoreLeadSummaryComponent" version="0.0.1" display-name-key="SitecoreLeadSummaryComponent" description-key="SitecoreLeadSummaryComponent description" control-type="virtual" >
        <!--external-service-usage node declares whether this 3rd party PCF control is using external service or not, if yes, this control will be considered as premium and please also add the external domain it is using.
        If it is not using any external service, please set the enabled="false" and DO NOT add any domain below. The "enabled" will be false by default.
        Example1:
          <external-service-usage enabled="true">
            <domain>www.Microsoft.com</domain>
          </external-service-usage>
        Example2:
          <external-service-usage enabled="false">
          </external-service-usage>
        -->
        <external-service-usage enabled="false">
          <!--UNCOMMENT TO ADD EXTERNAL DOMAINS
          <domain></domain>
          <domain></domain>
          -->
        </external-service-usage>
        <!-- property node identifies a specific, configurable piece of data that the control expects from CDS -->
        <property name="notRequired" display-name-key="Not Required" description-key="Selecting this will not do anything" of-type="SingleLine.Text" usage="bound" required="false" />
        <!--
          Property node's of-type attribute can be of-type-group attribute.
          Example:
          <type-group name="numbers">
            <type>Whole.None</type>
            <type>Currency</type>
            <type>FP</type>
            <type>Decimal</type>
          </type-group>
          <property name="sampleProperty" display-name-key="Property_Display_Key" description-key="Property_Desc_Key" of-type-group="numbers" usage="bound" required="true" />
        -->
        <resources>
          <code path="index.ts" order="1"/>
          <platform-library name="React" version="16.8.6" />
          <platform-library name="Fluent" version="8.29.0" />
          <!-- UNCOMMENT TO ADD MORE RESOURCES
          <css path="css/SitecoreLeadSummaryComponent.css" order="1" />
          <resx path="strings/SitecoreLeadSummaryComponent.1033.resx" version="1.0.0" />
          -->
        </resources>
        <!-- UNCOMMENT TO ENABLE THE SPECIFIED API
        <feature-usage>
          <uses-feature name="Device.captureAudio" required="true" />
          <uses-feature name="Device.captureImage" required="true" />
          <uses-feature name="Device.captureVideo" required="true" />
          <uses-feature name="Device.getBarcodeValue" required="true" />
          <uses-feature name="Device.getCurrentPosition" required="true" />
          <uses-feature name="Device.pickFile" required="true" />
          <uses-feature name="Utility" required="true" />
          <uses-feature name="WebAPI" required="true" />
        </feature-usage>
        -->
      </control>
    </manifest>
    ```

    - The property is set to `required=false`.  If we were to set it to true and select a primary key field, we would not have the ability to remove it as it would throw an error saying that you can't remove a required field.
    - The React and Fluent platform libraries are added to allow the use of these in our component. This enables both the use of the React Framework and the Fluent UI Framework.

#### Adding to Dynamics

Once the component is built using the CLI (https://learn.microsoft.com/en-us/power-apps/developer/component-framework/implementing-controls-using-typescript), you follow these steps:

1. Import the solution. 

   1. Click the `Import Solution` button![](https://i.imgur.com/63PSvwU.png)

   2. Click the `Browse` button to open the file explorer![](https://i.imgur.com/HRPFlG8.png)

   3. Navigate to the debug folder and select the `Solutions.zip` folder. Should be something like: 

       `S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Solutions\bin\Debug\Solutions.zip`![](https://i.imgur.com/yWbRi6S.png)

   4. Click `Next` button![](https://i.imgur.com/V4YZOgJ.png)

   5. Click the import button. Make sure the checkbox is checked in the advanced settings![](https://i.imgur.com/g8vdRrf.png)

   6. You will see that the component has been added to the solution![](https://i.imgur.com/7CbEjS8.png)

2. Open the form editor `Tables > Lead > Forms`.

   1. ![](https://i.imgur.com/SkwQGl0.png)
   2. Click the `Sales Insights` form. If this is not there, click `Add Existing` and navigate to this form and add it to the solution.![](https://i.imgur.com/w7rFad0.png)
   3. In the components tab, click `Get more components`![](https://i.imgur.com/JkubQSu.png)
   4. If the previous steps have been done correctly, you should see the component here. Select the component![](https://i.imgur.com/0rDQwMl.png)
   5. Click `Add`![](https://i.imgur.com/DIh7hBE.png)
   6. You will see that it has added to the `More components` dropdown.![](https://i.imgur.com/2LEvsRY.png)
   7. Create a `1-column tab`, then drag and drop the component onto the form. This pop-up will display. Do not select a column, just leave it blank and click done![](https://i.imgur.com/rRvgaIQ.png)
   8. You will see that the component has been added. Click `Publish` to publish the changes to Dynamics![](https://i.imgur.com/tFatvR0.png)

3. View the component

   1. Go to  the Leads pipeline and select a lead. Make sure this dropdown is set to `Sales Insight` as that is the form we edited

      ![](https://i.imgur.com/zV3gfvA.png)

   2. Select the tab that says `Sitecore Visitor`. You will now see the component has loaded. 

      ![](https://i.imgur.com/YI7fvYQ.png)

#### Modifying the System

| Action                                                       | Name                                                         | Line      | Location                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | --------- | :----------------------------------------------------------- |
| Method for getting the LeadID                                | `init()`                                                     | 49        | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts) |
| Method for updating the display                              | `updateView()`                                               | 60        | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts) |
| Main props for the component                                 | `updateView()`                                               | 61        | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts) |
| Method for retrieving the data from Dynamics (triggers dev_GetData) | `GetData()`                                                  | 88        | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts) |
| Execution that calls the custom unbound action               | `GetData()`                                                  | 109       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts) |
| Mapping of JSON to Interfaces                                | `GetData()`                                                  | 120/132   | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts) |
| Set the values of the ISitecoreStatistics object to the values from the response | `GetData()`                                                  | 161       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts) |
| Requests re-rendering of the control with updated data.      | `GetData()`                                                  | 170       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\index.ts) |
| ?????????????????????????????????                                                  | ???????????????????????????????????????                                                | ???????????????     | ??????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????? |
| Modify detailslists                                          | General                                                      | N/A       | C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx |
| Modify columns on the individual visits details detailslist  | `VisitsDetailsList()`                                        | 41        | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| Modify the column on the table that displays all visits      | `VisitsDetailsList()`                                        | 80        | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| The method for viewing details of a single visit on-demand   | `VisitsDetailsList()`                                        | 94        | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| Modify the html for the details list component               | `VisitsDetailsList()`                                        | 235       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| Modify the html for the details list for all visits          | `VisitsDetailsList()`                                        | 238       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| Modify the html for the modal popup                          | `VisitsDetailsList()`                                        | 251       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| Modify the html for the details list for individual visit details | `VisitsDetailsList()`                                        | 271       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| Change the styling for the details list component            | `contentStyles`, `iconButtonStyles`, `stackTokens`, `stackStyles` | 296+      | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| ?????????????????????????????????                                                  | ???????????????????????????????????????                                                | ???????????????     | ??????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????? |
| Modify Base Component                                        | General                                                      | N/A       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx) |
| Interfaces                                                   | `ISitecoreStatistics`, `ISitecoreVisit`, `IProps`, `IState`  | 22-73     | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx) |
| Modify Base Component Class                                  | ComponentBase                                                | 81        | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx) |
| The method that refreshes the analytics                      | `refresh()`                                                  | 93        | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx) |
| Define the request variable                                  | `refresh()`                                                  | 99        | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx) |
| Execute the request above (refresh analytics)                | `refresh()`                                                  | 118       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx) |
| Call the GetData method to retrieve the new data             | `refresh()`                                                  | 128       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx) |
| Refreshes the component                                      | `refresh()`                                                  | 130       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx) |
| Modify the html for the component                            | `render()`                                                   | 139 - 219 | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx) |
| Change the styling for the component base component          | `palette()`, `styles()`                                      | 221+      | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\ComponentBase.tsx) |



## Glossary

| Name                    | Description                                                  | Alias                                                      |
| ----------------------- | ------------------------------------------------------------ | ---------------------------------------------------------- |
| Component (Power Apps)  | Components are a type of solution component, which means they can be included in a solution file and imported into different environments. | Custom Control, Code Component, PCF Component, PCF Control |
| Component (React)       | React components are small, reusable pieces of code that return a React element to be rendered to the page. |                                                            |
| Custom Process          | Custom process actions open a range of possibilities for composing business logic. With custom process actions, you can perform operations, such as Create, Update, Delete, Assign, or Perform Action. | Custom Actions, Actions                                    |
| Entity                  | An entity defines the information you want to track in the form of records, which typically include properties such as company name, location, products, email, and phone. | Table                                                      |
| Props                   | Props are inputs to a React component. They are data passed down from a parent component to a child component. |                                                            |
| State                   | A component needs state when some data associated with it changes over time. For example, a `Checkbox` component might need `isChecked` in its state, and a `NewsFeed` component might want to keep track of `fetchedPosts` in its state. |                                                            |
| Environment Variables   | Apps and flows often require different configuration settings across environments. Environment variables allow you to transport application configuration data with solutions and optionally manipulate the values in your Application Lifecycle Management (ALM) pipeline. They act as configurable input parameters allowing you to reference an environment variable within other solution components. You can update a value without modifying other components directly when using environment variables. | Sitecore Configuration                                     |
| TSX                     | The file extension for Typescript files. Similar use as JSX, but using TypeScript |                                                            |
| Step Message            | All data operations in the organization service are defined as messages. While the IOrganizationService provides 7 methods to perform data operations (Create, Retrieve, Retrieve Multiple, Update, Delete, Associate, Disassociate), each of these methods is a convenience wrapper around the underlying message which is ultimately invoked using the Execute method. |                                                            |
| Primary Entity          | The primary entity that the plugin is bound to. If valid tables apply, you set this when you want to limit the number of times the plug-in is called. When left blank, messages like `Update`, `Delete`, `Retrieve`, and `RetrieveMultiple` or any message that can be applied with the message the plug-in will be invoked for all the tables that support this message. |                                                            |
| Secondary Entity        | This field remains for backward compatibility for deprecated messages that accepted an array of EntityReference as the `Target` parameter. This field is typically not used anymore. |                                                            |
| Step Stage of Execution | The stage in the event pipeline that the plug-in is executed. PreValiadion is for the initial operation, this stage will occur before the main system operation. PreOperation occurs before the main system operation and within the database transaction. PostOperation occurs after the main system operation and within the database transaction. | Event Pipeline Stage of execution                          |

## Conclusion

There is not much to discuss about this week. Documenting the system as a whole took up all the time and that was the primary focus. The only information that was learned was the details in the description of each item in the glossary. These details were from reading Microsoft documentation. In terms of the progress of the project, it is nearing completion. The physical product is a working proof-of-concept that meets the requirements of the project.

This week was successful in the sense that the system could be handed to someone and they would have a good understanding of how it works, and could take over from where it has been left.
