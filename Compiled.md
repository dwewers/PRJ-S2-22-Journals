# Journal 1 - 27/07/2022

This week was an introduction to the team and learning the requirements of the project. Over the semester,there will be the responsibility for the development of a prototype that transfers Sitecore web behavior to Microsoft Dynamics 365 and, if time allows, personalize Sitecore from Dynamics. The ability to create and interpret specifications based on business requirements expressed by FuseIT in writing or in person is a key part of the placement. The responsibility of the work placement is for both defect resolution and the creation of new functionality.

The following are the requirements of the placement:

- Quickly learn the required Microsoft and Sitecore technologies;
- Some .NET experience working with C#;
- Understand source control and code repositories;
- Ability to self-maintain work schedule and meet deadlines;
- Able to accept guidance from more senior members of the team;
- Good written and spoken English skills;

Below are the responsibilities as an intern at FuseIT:

- Business analysis and interpretation of business requirements;
- Implementing the requirement in code;
- Implement unit and automated test strategies as appropriate;
- Ensure project is delivered to some stage of completion;
- Maintain strong coding standards for software development;

------

# Journal 2 - 03/08/2022

The purpose of this week was an introduction too plugin development. This involved reading up on Microsoft documentation as well as any other tutorials present on the internet as well as implementing test plugins. This was a steep learning curve as there are many different purposes for plugins which require you to program in completely different ways. After reading plenty of documentation, it was then time to implement a test plugin for the dynamics environment. As it was a requirement that there would be custom entities, an ERD was presented that displayed the tables that were specific too FuseIT's needs (Dynamics has default tables made when a solution is created). Therefor, the "visits" entity was chosen for the test and would be used to learn the integration with Dynamics. The objective for this test plugin was to generate the tables when the solution is first created. 

### The steps involved for creating a plugin is rather simple:

#### Creating the Plugin

1. Firstly, you create a mew Class library (.NET Framework) project using .NET Framework 4.6.2. It is important that you use this version as other versions are not compatible with the plugin registration tool that is used later on.

   ![](https://i.imgur.com/mNrKkFs.png)

2. The next step involves installing the required NuGet Packages. This involves right clicking the project (BasicPlugin) in the file explorer and selecting **Manage NuGet Packages**

   ![](https://i.imgur.com/KzsbQ7W.png)

3. The required package is as follows:

   ```html
   Microsoft.CrmSdk.CoreAssemblies
   ```

4. The next step performed was writing the code. This would be done in the **Class1** file, however this was changed to fit the naming of the given plugin:

   ```c#
   public class GenerateEntitiesPlugin : IPlugin
   {
       public void Execute(IServiceProvider serviceProvider)
       {
           //NEED TO ADD MY CODE HERE!!!!!!!!!!!!!!
       }
   }
   ```

   5. If the code is valid, it should build when `F6` is pressed. However, if there are any issues, this will throw related errors.

   6. All assemblies containing plugins must be strongly signed. This is done using a **KeyFile**. To do this, you again right click on the project, but this time click **Properties**

      ![](https://i.imgur.com/V1lqWV2.png)

   7. Under the **Signing** tab, you click **Sign the assembly** followed by **<New...>** in the **Create Strong Name Key** dialog.

   8. Click ok and a Key is generated.

   9. Once again, build the plugin with `F6`

   10. The plugin can be found in the file explorer at `\bin\Debug\BasicPlugin.dll`

   #### Registering the Plugin

   1. After downloading the Registration Tool through the Microsoft Website, run `PluginRegistration.exe`
   2. Click **Create new Connection**
   3. Select **Office 365** as the Deployment Type
   4. Sign in using your login credentials
   5. After you are connected, you will see any existing registered plug-ins, custom workflow activities and data providers.
   6. In the **Register** drop-down, select the **New Assembly** option or press `Ctrl + A`
   7. In the **Register New Assembly** dialog, select the ellipses (**…**) button and browse to the assembly you built in the previous step.
   8. Click **Register Selected Plugins**
   9. You will see a **Registered Plugins** confirmation
   10. Click **OK** to close the dialog and close the **Register New Assembly** dialog.
   11. You should now see the **(Assembly) BasicPlugin** assembly which you can expand to view the **(Plugin) BasicPlugin.GenerateEntitiesPlugin** plugin.

   #### Registering a New Step

   1. Right-click the **(Plugin) BasicPlugin.GenerateEntitiesPlugin** and select **Register New Step**.

   2. In the **Register New Step** dialog, set the following fields (however this will be different for the actual implementation as we don't want this to be executed every time an account is created. This is just for testing purposes):

      | Setting                           | Value         |
      | --------------------------------- | ------------- |
      | Message                           | Create        |
      | Primary Entity                    | account       |
      | Event Pipeline Stage of Execution | PostOperation |
      | Execution Mode                    | Asynchronous  |

   3. Click **Register New Step** to complete the registration and close the **Register New Step** dialog.

   4. You can now see the registered step.

   5. After some time, the plugin should now work.

### Issues Faced

There were a few issues throughout the week, these were as follows:

```asciiarmor
ISSUE: Additions to the plugin were not changing in the application side

FIX: The plugin needs to be updated using the plugin registration tool every time there are changes made
```

```asciiarmor
ISSUE: When the plugin is executed, it creates the table. However, if the plugin is run again (as for testing purposes it is executed when an account is created) it throws a "Business Error" (Which is an error related to the plugin displayed on the Dynamics Interface side)

FIX: This issue has not been solved  as it will not be an issue when it is correctly implemented (as in tied to the creation of the solution and never executed again)
```

```asciiarmor
ISSUE: An error message was thrown when trying to import a solution

ERROR: Failed to find solution "GoalsandVisits_1_0_0_0.zip". Solution is not found when trying to export package

FIX: Started the export wizard again and it worked
```

```asciiarmor
ISSUE: Postman was returning a 402 (No Content) when trying to post a new visit with a link to a lead

PROBLEM: It was thought that the post would past the name of the field and the value as parameters in the body of the post method like shown below: 
	"new_new_parent_leadid": "7da38ae0-4d0e-ea11-a813-000d3a1bbd52"
	
FIX:It was discovered that you are supposted to use "odata.bind." Like it was originally thought, you still pass these parameters in the body of the post method, however, you write it as follows:
	"new_new_parent_leadid@odata.bind": "/leads(7da38ae0-4d0e-ea11-a813-000d3a1bbd52)"
```

Overall, this week was a big learning curve, but a lot of new knowledge was gained and a lot of experience was gained using Microsoft Dynamics 365.

------

# Journal 3 - 10/08/2022

### WebAPI

This weeks focus was taking the design from the Sitecore component and building it in the Power Apps Component Framework for the Dynamics 365 instance. Of the time working on the project, this week felt like the most progress made. It started off with playing around with a took called **XRM Toolbox**. This is a Windows application that is used to connect to the Microsoft Dataverse and perform many different tasks such as installing plug-ins, creating fetch requests, and other general customizations. The tool that was used was this week was the **FetchXML Builder.** The purpose of this tool is to build queries for the Microsoft Dataverse and the Power Platform. There are many different useful features in this tool such as constructing advanced queries with aggregates and outer joins, the ability to query CDS/CRM where information may not be found within the user interface, and code generation from the queries.

Bellow is more in depth details from XrmToolBox (2022):

The tool will assist in three major areas:

1. Constructing queries, including more advanced features like:
   - Aggregates
   - Outer joins
   - Complex "not-in"-queries
   - Attributes from multi-level linked entities
   - Update existing views with altered queries

2. Querying CDS/CRM for information not (easily) found in the UI
   - System / internal entities
   - Attributes hidden in CRM UI
   - Join on other fields than relationships

3. Developer assistance generating code for
   - C# QueryExpression code
   - WebAPI / OData query string
   - Power Automate List Records
   - SQL, JavaScript and C# stubs
   - Easy to use UI to compose queries for reports in CRM

This was a very useful tool as it would allow the creation of queries by entering parameters, selecting the required entities and attributes from a list, selection of constraints, and any other relevant details to your query. From this, you can choose different code generation such as C# and JavaScript. In the case of the project, it was originally tested using FetchXml in JavaScript, however this was found to not work. 

The code tested was as follows:

```typescript
var fetchData = {
  "new_new_parent_leadid": this._context.parameters.sampleProperty.raw
};
var fetchXml = [
"<fetch>",
"  <entity name='lead'>",
"    <link-entity name='new_sitecorevisit' from='new_new_parent_leadid' to='leadid'>",
"      <attribute name='new_visitduration'/>",
"      <attribute name='createdon'/>",
"      <filter>",
"        <condition attribute='new_new_parent_leadid' operator='eq' value='", fetchData.new_new_parent_leadid/*9fa18ae0-4d0e-ea11-a813-000d3a1bbd52*/, "' uiname='Gerald Stephens' uitype='lead'/>",
"      </filter>",
"    </link-entity>",
"  </entity>",
"</fetch>"
].join("");
```

However, this threw the following error:

```apl
Error occured during initialization of control: dev_LeadSummary2.LeadComponent2;Message: Option Parameter should begin with "?"
```

It wasn't obvious what was causing this issue, but my understanding is that when using WebAPI, you write the requests as follows:

```apl
https://{organisation}/api/data/v9.2/{entity}?{conditions}
```

It is likely that there was something wrong with the format when it was being converted into a string. The following is what it was changed to in order for it to work as expected:

```typescript
var fetchData = {
  "new_new_parent_leadid": this._context.parameters.sampleProperty.raw
};

let queryString = `?$expand=new_lead_new_sitecorevisit_new_parent_leadid($select=new_visitduration,createdon;$filter=(_new_new_parent_leadid_value eq ${fetchData.new_new_parent_leadid}))&$filter=(new_lead_new_sitecorevisit_new_parent_leadid/any(o1:(o1/_new_new_parent_leadid_value eq ${fetchData.new_new_parent_leadid})))`;
```

However, a mistake was made in the following code that would throw another error. This code is what calls the webAPI passing two parameters, firstly the entity, and secondly the conditions:

```typescript
//ERROR: The entity "leads" cannot be found. Specify a valid query, and try again
this._context.webAPI.retrieveMultipleRecords("leads", queryString).then(function (lead) {
   //Business Logic 
}                                                                        

//SOLUTION: Changing "leads" to "lead"
this._context.webAPI.retrieveMultipleRecords("lead", queryString).then(function (lead) {
   	//Business Logic
}

```

### UX/UI Design

Now that the query was built, and the data displayed was as expected (lead details and related visits), it was time to recreate the design from the Sitecore component into a Power Apps component. For the most part, this was not too difficult of a task as it was mostly html/css and some Bootstrap, however, there were some issues when it came to data binding.

The first issue encountered was that there was nothing showing at all on the display. The reason for this was that the following code was incorrect:

```javascript
// CAUSE:
this._container.innerHTML = "";

// SOLUTION:
container.innerHTML = "";
```

The next issue encountered was that it was throwing the following error:

```javascript
// ERROR: Cannot read properties of undefined (reading 'createdon')
// CAUSE:
lead.new_lead_new_sitecorevisit_new_parent_leadid[0].createdon

// SOLUTION:
details.visits[0].createdon

```

The following is the current state of the component:

![](https://i.imgur.com/YFgz3xo.png)

As we can see, **Goal Values**, **Avg Visit Time**, and **Views** are not showing what we would expect. This is to do with the data calculations and is something that will be fixed next week. 

Overall, it has been a pretty successful week with a lot learned and a significant amount of progress made.

XrmToolBox. (2022, July 15). *FetchXML Builder · XrmToolBox*. Retrieved August 18, 2022, from https://www.xrmtoolbox.com/plugins/Cinteros.Xrm.FetchXmlBuilder/

------

# Journal 4 - 17/08/2022

### React component

During the learning of the Power Apps component, the main way that most tutorials or documentation showed was the use of TypeScript with creating each element and defining its classes and inner html manually, or returning a full html script as a string with the parameters passed in where needed. An example of this is as shown below:

Option A:

```typescript
// Required private parameters of the control
private _context: ComponentFramework.Context<IInputs>;
private _container: HTMLDivElement;
private _main: HTMLDivElement;

public init(context: ComponentFramework.Context<IInputs>, notifyOutputChanged: () => void, state: ComponentFramework.Dictionary, container:HTMLDivElement)
{
    // assigning environment variables.
    this._context = context;
    this._container = container;
    this._main = document.createElement("div");
    this._creditCardErrorElement.setAttribute("class", "main-div");
    //And any other details
    
    this._container.appendChild(this._main)
}
```

Option B:

```typescript
// Required private parameters of the control
private _context: ComponentFramework.Context<IInputs>;
private _container: HTMLDivElement;
private _main: HTMLDivElement;

public init(context: ComponentFramework.Context<IInputs>, notifyOutputChanged: () => void, state: ComponentFramework.Dictionary, container:HTMLDivElement)
{
    // assigning environment variables.
    this._context = context;
    this._container = container;
    this._main = document.createElement("div");
    //And any other details
    var html =
       `<div class="main-div">
            <div class="full-name">
                <h1>${firstName} ${lastName}</h1>
            </div>
        </div>`
    this._main.innerHTML = html;
    
    this._container.appendChild(this._main)
}
```

This would not be an issue for a small component, but for a component as complex as the one for this project, the code becomes overly complex, or a long string of HTML which is very messy. Microsoft recently added framework compatibility for the Power App Component Framework, including React. This was a significant discovery during the project's development because it allows for better separation of concerns as well as cleaner and more reusable code. This means that we can now use the latest Microsoft Power Platform CLI for model-driven apps, canvas apps, and portals to create code components directly with the React JS and Fluent UI libraries without any custom project setup. 

Similar to the pac CLI command used for a default component, we type out the same details (name, namespace, and template), however with the addition of a framework parameter. The following is an example of this:

```powershell
pac pcf init --name SummaryComponent --namespace Fuseit.S4D.Dynamics --template field --framework React
```

In the `index.ts` file, we have `updateView()` method where we are initializing the React component where you can pass your own component as an entry point for the other React Component. This is shown below:

```typescript
//index.ts
import {LeadSummaryComponent, IStats} from "./LeadSummaryComponent"

public updateView(context: ComponentFramework.Context<IInputs>): React.ReactElement{
	const props: IStats = {
        name:"Daniel",
        age: 21
    }
    return React.createElement(
		LeadSummaryComponent, props
	);
}

//LeadSummaryComponent.tsx
export interface IStats{
    name: string,
    age: number
}

export class LeadSummaryComponent extends React.Component<IStats> {
    public render(): React.ReactNode {
        return (
            <div class="main-div">
            	<h1>{this.props.name}</h1>
            </div>
        )
    }
}
```

Below is a comparison between the a component using the with and without using the React framework:

Without React Framework:

```typescript
public init(context: ComponentFramework.Context<IInputs>, notifyOutputChanged: () => void, state: ComponentFramework.Dictionary, container:HTMLDivElement)
{
    this._container = container;
    this._main = document.createElement("div");
    
    var html =
           `
              <div className="container">
                <div className="row">
                  <div className="col-md-12">
                     ${ this.props.name }
                  </div>
                </div>
              </div>
            `
    this._main.innerHTML = html;
    
    this._container.appendChild(this._main)
}
```

With React Framework:

```React
export class SummaryComponent extends React.Component<ISummaryStats, ISummaryStats> {
  	constructor(props){
        super(props);
    }
    
    public render(): React.ReactNode {
    return (
      <div className="container">
        <div className="row">
          <div className="col-md-12">
             { this.props.name }
          </div>
        </div>
      </div>
    )
  }
}
```

Overall, it has been a pretty successful week with a lot learned and a significant amount of progress made towards the project.

------

# Journal 5 - 24/08/2022

As it was five weeks into the learning phase of the project, and most testing and integration testing was complete with good knowledge of custom controls and plugins, it was time to move onto the actual product that was desired. The custom control would remain pretty much the same with a few minor adjustments, whereas a lot of work would need to be put in to the development of the plugins that would work with the control.

Presented was a full outline of what was required for the project as well as the key requirements such as functionality and entity design. These would help with the design and implementation of the controls and plugins.

Firstly, the requirements of the product are as follows:

1. Custom entities for Sitecore XDB analytics data (the tables where the data will be stored)
2. Present data summary on Lead page (a custom control that queries data from a plugin and displays it on a standardized form)
3. Update analytics data for single Lead on-demand (a button click that would refresh the view)
4. View details of a single visit on-demand (the ability to select a visit and view its details)

### 1. Custom entities for Sitecore XDB analytics data

Create custom entities as per the ERD. 

**NOTE:** only item 3 (Update analytics data for single Lead on-demand) requires creating or updating data within  Dynamics 365; all other writes are performed externally by the Sitecore component of S4D

![image-20220901123803909](https://i.imgur.com/D6IXHWo.png)

### 2. Present data summary on Lead page

Create a Custom Control that presents Sitecore analytics data for a Lead, as referenced by `sitecorexdbcontact.leadid`. 

This should include the following: 

1. Simple overview statistics: 

   - Total Goal value

   - Total Visit count 

   - Average visit time 

   - Total page view count 

     Ideally these statistics should be retrieved via a single call to custom code, which includes the labels in the response. This may be modified in future to provide different statistics. 

2. A tabulated list of the Visits, including: 

   - Start date 

   - Channel 

   - Duration 

   - Total page view count 

   - Browser 

   - Total Goal value 

   - A button or link to perform item 4 (View details of a single visit on-demand) 

3. A chart of the page view count over time. 

4. A button or link to perform item 3 (Update analytics data for single Lead on-demand) with the text “Get latest visits”.

### 3. Update analytics data for a single lead on-demand

This functionality has little UI other than the button in (d) above, and a means of user feedback for successful or failed update. This may be in the form of popup messages or whatever mechanism is in keeping with standard Dynamics 365 interface elements.

This process should:

1. Use configured settings, likely an Environment Variable with JSON to define: 

   - The Sitecore endpoint base URL including protocol and optional port 

   - The maximum number of visits to retrieve 

   - A shared API key value (text e.g., a GUID/UUID)

2. POST a request to the configured endpoint (with a constant relative path TBD). This will be a REST endpoint  that receives and returns JSON.

   - For example:

     ```json
     //	Example request:
     {
          “apiKey”: “<value of shared api key setting>”,
          “contactAliasId”: “<value of sitecorexdbcontact.xdbaliasid>”,
          “maxVisits”: <value of maximum visits setting>
     }
     //	Example success response:
     {
          “status”: “success”,
          “contactAliasId”: “<should match the request>”,
          “visits”: [
             {
                 “interactionid”: “C6732455-439E-0000-0000-0639A894F065”,
                 “name”: “2021.255.10438”,
                 “startdate”: “2021-09-08 20:15:46.98Z”,
                 “enddate”: “2021-09-08 20:18:32.65Z”,
                 “browser”: “Chrome v102.0”,
                 “channel”: “direct”,
                 “duration”: “00:02:22.2023053”,
                 “views”: 5,
                 “value”: 10,
                 “campaign”: “”
                 “goals”: [
                     {
                     “goalid”: “637670953205500000_a24a09da-a5b3-0000-0000-
                     063a758176ad_b0c5a8de-8355-49be-ba71-37b770d3471a”,
                     “date”: “2021-09-13T02:02:00.000Z”,
                     “path”: “/products/request-quote”,
                     “value”: 10
                     },
         			{…}
         		]
         	},
         	{…}
         ]
     }
     //	Example failure response:
     {
          “status”: “error”,
          “error”: “Invalid API key”
     }
     ```

     

3. Parse the response. 

   - Errors should be relayed to the end user in a manner that integrates well with standard Dynamics  365 UI, and is fit for consumption by non-administrative users. 

   - Successful responses should immediately update the relevant entity records, followed by a refresh  of the on-screen data to show the latest information. 

   - Strongly-typed classes should be created to serialize and deserialize JSON. 

     **NOTE:** that Dynamics 365 does not allow third-party assemblies to be referenced, so this needs to be implemented using `System.Runtime.Serialization` with Data Contracts.

### 4. View details of a single visit on-demand

This functionality is triggered by button or link in the table of visits. Because detailed interaction data is not stored in  Dynamics 365 due to the large volume, this information is available only via on-demand API call to Sitecore.

This process works similarly in concept to the update analytics call. It should:

1. Use the same configuration settings for base URL and shared API key.

2. POST a request to the configured endpoint (with a constant relative path TBD, different from update  analytics). This will be a REST endpoint that receives and returns JSON.

   ```json
   //	Example request:
   {
        “apiKey”: “<value of shared api key setting>”,
        “interactionid”: “<value of sitecorevisit.interactionid>”
   }
   //	Example success response:
   {
        “status”: “success”,
        “interactionid”: “<should match request value>”,
        “events”: [
            {
                “type”: “PageViewEvent”,
                “timestamp”: “2021-09-08T20:15:46.98Z”,
                “url”: “https://sitename.com/path”
            },
            {..}
   	]
   }
   //	Example failure response:
   {
        “status”: “error”,
        “error”: “Invalid API key”
   }
   ```

   3. Parse the response.

      - Errors should be relayed to the end user in a manner that integrates well with standard Dynamics  365 UI, and is fit for consumption by non-administrative users.

      - Successful responses should immediately display the events on-screen in a manner consistent with  standard Dynamics 365 UI. This should be tabulated; it may be in a popup or on a new page. 

      - Strongly-typed classes should be created to serialize and deserialize JSON.

Overall, the main focus of this week was to understand the functional and non-functional requirements of the system. The goal hear was to focus more on the research and planning rather than starting the implementation of code. This was quite a productive week as a lot was learned and a decent start was made on the final report

------

# Journal 6 - 31/08/22

### Getting started

It was then time to start on the final product. Initially, a plugin had been developed to create the custom entities and their relationships, however this would not be necessary as the final product would be a solution that can be imported by the client. This solution would contain all of the required entities rather than creating them when the solution is imported. To do this, the Power Apps tool was used to create each of the entities shown in the relational diagram provided. This did not take very long as it is mostly just point and click with a few configurations (setting string limits).

As we can see below, all of the custom entities are displayed in the solutions tables list. You may notice that the lead and contacts table is also listed. This is because of the relationships with them (the lookup fields on the XDB entity).

![](https://i.imgur.com/EOhqvQq.png)

The next step was to start work on the plugin new plugin. 

A basic plugin is quite easy to make and only requires a few steps.

1. The first step Create a .NET Framework Class library project in Visual Studio. This plugin should use .NET Framework 4.6.2 as any features introduced after this version will throw an error.

   ![asd](https://imgur.com/wjYiJEp.jpg)

   ![](https://imgur.com/mYcm5nX.jpg)

   ![](https://imgur.com/ajd4ruF.jpg)

   ![](https://imgur.com/fZatnZy.jpg)

2. Next, add the `Microsoft.CrmSdk.CoreAssemblies` NuGet package to the project by running the following in the `Package Manager Console` located at the bottom of the screen:

   ```turtle
   Install-Package Microsoft.CrmSdk.CoreAssemblies
   ```

   Or go through the NuGet package manager.

3. Implement the [IPlugin](https://learn.microsoft.com/en-us/dotnet/api/microsoft.xrm.sdk.iplugin) interface on classes that will be registered as steps.

   ```c#
   using Microsoft.Xrm.Sdk;
   using System;
   
   namespace ExamplePlugin
   {
       public class Plugin : IPlugin
       {
       }
   }
   ```

4. Add your code to the Execute method required by the interface and add any references to services or business logic:

   ```c#
   using Microsoft.Xrm.Sdk;
   using Newtonsoft.Json;
   using System;
   using System.ServiceModel;
   using System.Text.Json.Serialization;
   
   namespace ExamplePlugin
   {
       public class Plugin : IPlugin
       {
           public void Execute(IServiceProvider serviceProvider)
           {
               // Obtain the tracing service
               ITracingService tracingService =
                   (ITracingService)serviceProvider.GetService(typeof(ITracingService));
   
               // Obtain the execution context from the service provider.  
               IPluginExecutionContext context = (IPluginExecutionContext)
                   serviceProvider.GetService(typeof(IPluginExecutionContext));
   
               // The InputParameters collection contains all the data passed in the message request.  
               if (context.InputParameters.Contains("Target") &&
                   context.InputParameters["Target"] is Entity)
               {
                   // Obtain the target entity from the input parameters.  
                   Entity entity = (Entity)context.InputParameters["Target"];
   
                   // Obtain the organization service reference which you will need for  
                   // web service calls.  
                   IOrganizationServiceFactory serviceFactory =
                       (IOrganizationServiceFactory)serviceProvider.GetService(typeof(IOrganizationServiceFactory));
                   IOrganizationService service = serviceFactory.CreateOrganizationService(context.UserId);
   
                   try
                   {
                       // Plug-in business logic goes here.  
                       context.OutputParameters["data"] = JsonConvert.SerializeObject(new
                       {
                           name = "Daniel",
                           age = 21,
                           gender = "Male"
                       });
                   }
   
                   catch (FaultException<OrganizationServiceFault> ex)
                   {
                       throw new InvalidPluginExecutionException("An error occurred in FollowUpPlugin.", ex);
                   }
   
                   catch (Exception ex)
                   {
                       tracingService.Trace("FollowUpPlugin: {0}", ex.ToString());
                       throw;
                   }
               }
           }
       }
   }
   ```

5. Sign the assembly

   1. Click the class library in the solution explorer
   2. Press `ctr + enter`
   3. In the navigation pane, select `Signing`
   4. Tick the textbox that says `Sign the assembly`
   5. In the dropdown, select new and create a new key with a password and a key name
   6. Click ok

6. Right click the solution and click `Build` or press `ctr + b`

7. The plugin is now ready to be registered

To register a plugin, you need to perform the following steps:

1. Firstly, you need to install the tools (ideally in your root directory):

   ```bash
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   $sourceNugetExe = "https://dist.nuget.org/win-x86-commandline/latest/nuget.exe"
   $targetNugetExe = ".\nuget.exe"
   Remove-Item .\Tools -Force -Recurse -ErrorAction Ignore
   Invoke-WebRequest $sourceNugetExe -OutFile $targetNugetExe
   Set-Alias nuget $targetNugetExe -Scope Global -Verbose
   
   if (-not (./nuget source | Where-Object { $_ -like "*https://api.nuget.org/v3/index.json*"})) {
     .\nuget sources Add -Name nuget.org.v3 -Source  https://api.nuget.org/v3/index.json
   }
   
   ##
   ##Download Plug-in Registration tool
   ##
   ./nuget install Microsoft.CrmSdk.XrmTooling.PluginRegistrationTool -O .\Tools
   mkdir .\Tools\PluginRegistration
   $prtFolder = (Get-ChildItem ./Tools | Where-Object {$_.Name -match 'Microsoft.CrmSdk.XrmTooling.PluginRegistrationTool.'}).Name
   Move-Item .\Tools\$prtFolder\tools\*.* .\Tools\PluginRegistration
   Remove-Item .\Tools\$prtFolder -Force -Recurse
   
   ##
   ##Remove NuGet.exe
   ##
   Remove-Item nuget.exe
   ```

2. Navigate to the folder ``[Your folder]\Tools\PluginRegistration`` and run the application called `PluginRegistration`

   ![](https://imgur.com/aK03ee8.jpg)

3. Create a new connection

   ![](https://imgur.com/5R8Pk3W.jpg)

4. Log into your Office 365 account

   ![](https://imgur.com/eoMoMDp.jpg)

5. This will display a list of all assemblies registered to the environment

   ![](https://imgur.com/GooVXe3.jpg)

6. Press `ctr + a` to register a new assembly (your plugin) or navigate through the navigation pane `Register > Register New Assembly`

7. Click the three dots to open the file explorer

   ![](https://imgur.com/NntMNFx.jpg)

8. Navigate to the `.dll` file located in `[Your solution]\Bin\Debug\[Plugin].dll`

   ![](https://imgur.com/zTvcsFQ.jpg)

9. Make sure the following are checked then click `Register Selected Plugins`

   ![](https://imgur.com/5RD0F9G.jpg)

10. If successful, the following message will display and we will be able to see the assembly in the list:

    ![](https://imgur.com/EBblotl.jpg)

    ![](https://imgur.com/ekPfjsq.jpg)

11. Now we need to register a step. This is what will trigger the plugin. Start by pressing `ctr + t`

    ![](https://imgur.com/2tHqIiX.jpg)

    Here some test details have been filled out:

    - Message: What will trigger the plugin (in this case a Create message)
    - Primary Entity: Used if it is specific to the table. Left blank if it applies to all call messages
    - Event Pipeline State of Execution: Stage in the event pipeline that best suites the purpose for the plug-in.

    Once all of the configurations have been made in relation to your plugin, click `Register New Step`

12. We will then see the new step in the list:

    ![](https://imgur.com/ZbqQCEc.jpg)

We have now successfully created and registered our plugin


This week has been quite a learning curve. Firstly, was learning how to make a plugin, then how to register it. This was a slow process, but fortunately there is a lot of documentation.

------

# Journal 7 - 07/09/2022

This week once again, work was done on the plugin. It was discovered the XRM Toolbox has a very useful tool called "Early Bound Generator." This tool is used for creating early bound entities/option sets, and actions. As one of the requirements is that quote "Strongly-typed classes should be created to serialize and deserialize JSON," it is necessary that their are classes for each of the entities in the plugin. However, the code for creating these classes is nowhere near as simple as that of a basic C# class. Coding each and every entity would have been very time consuming and a general waste of time given that it can be generated through this tool. One of the custom entities, when converted to code, is about 800 lines of code. Multiply this by the five custom entities of different complexities, and we will end up with thousands of lines of code across five different classes (not including option sets).

Below is the configuration of just one attribute of an entity:

```c#
[Microsoft.Xrm.Sdk.AttributeLogicalNameAttribute("createdonbehalfby")]
public Microsoft.Xrm.Sdk.EntityReference CreatedOnBehalfBy
{
    [System.Diagnostics.DebuggerNonUserCode()]
    get
    {
        return this.GetAttributeValue<Microsoft.Xrm.Sdk.EntityReference>("createdonbehalfby");
    }
    [System.Diagnostics.DebuggerNonUserCode()]
    set
    {
        this.OnPropertyChanging("CreatedOnBehalfBy");
        this.SetAttributeValue("createdonbehalfby", value);
        this.OnPropertyChanged("CreatedOnBehalfBy");
    }
}
```

Next was the CRM Service Context. This represents a source of entities bound to a CRM service. It tracks and manages changes made to the retrieved entities. Here we define the binding sets for each of the entities (reduces the code each time we want to make a Linq query):

```c#
	public partial class CrmServiceContext : Microsoft.Xrm.Sdk.Client.OrganizationServiceContext
	{
		[System.Diagnostics.DebuggerNonUserCode()]
		public CrmServiceContext(Microsoft.Xrm.Sdk.IOrganizationService service) : 
				base(service)
		{
		}
		
        // We define one of these bindings for each of the entites
		public System.Linq.IQueryable<CrmEarlyBound.dev_sitecorevisit> dev_sitecorevisitSet
		{
			[System.Diagnostics.DebuggerNonUserCode()]
			get
			{
				return this.CreateQuery<CrmEarlyBound.dev_sitecorevisit>();
			}
		}
	}
```

Finally was the main code for processing the data. The execution method would take the lead id that was passed from the custom action and using Linq methods to retrieve relevant data from the entity services. The services are useful as it reduces code repetition. Below is an example of a service:

```c#
/// <summary>
/// Gets a binding to the set of all <see cref="dev_sitecorevisit"/> entities.
/// </summary>
public System.Linq.IQueryable<dev_sitecorevisit> VisitSet
{
    [System.Diagnostics.DebuggerNonUserCode()]
    get
    {
        return this.CreateQuery<dev_sitecorevisit>();
    }
}
```

The original method used to retireve the data worked, however it was overly complicated and not the easiest to read. The complexity also made future development harder if I wanted to add more to the query. Below we can see the initial method:

```c#
var query =
	crmServiceContext.LeadSet
	    .Join(crmServiceContext.XdbcontactSet, x => x.LeadId,
		y => y.dev_dev_sitecorexdbcontact_leadid_lead.LeadId,
		(x, y) =>
		    new
		    {
			lead = x, 
			contact = y
		    }
	    )
	    .Join(crmServiceContext.VisitSet, a => a.contact.dev_sitecorexdbcontactId,
		b => b.dev_xdbcontactid.Id, (a, b) =>
		    new
		    {
			lead = a, 
			visits = b
		    }
	    )
	    .ToList().Where(z => z.lead.lead.LeadId == id)
	    .Select(data => new
	    {
		data.visits,
		data.lead.contact,
	    });
```

This method worked by joining the leadset created by the Early Bound Generator with the Xdbcontact set based on the matching lead ids, then to the visit set based on the matching xdbcontactids. In a sense, the method used is similar to that with a Common Table Expression (cte) in SQL; inner joins were created between the data sets, a where clause was then stated, followed by the details that we wanted to retrieve. At first glance, it looked overwhelming, but once you got an idea of it, it became a bit easier to understand. However, this method could be simplified much more.

it was better that we go step by step and gather the desired information rather than trying to achieve it in one query. Firstly, it was not necissery to go through the leadset as we were already passing the leadid to the plugin on initialization of the custom control. Thus, the initial `crmServiceContext.LeadSet` reference could be removed. This meant that the entry point would then become the `crmServiceContext.XdbcontactSet.` As the goal was to split the steps up, a comparison was not required between two entity sets, rather we find an entity in the `XdbcontactSet` that matches the Lead Id that was passed to the plugin through the `context.InputParameters.`

Below we can see how this is achieved:

```c#
dev_sitecorexdbcontact contact = crmServiceContext.XdbcontactSet
	.FirstOrDefault(x => x.dev_dev_sitecorexdbcontact_leadid_lead.Id == id);
```

As we can see, this is far more simplified than the first part of the previous method. The same was replicated for the `crmServiceContext.VisitSet,` however we compare the contact id of the visits that match that of the contact entity declared above, as shown below:

```c#
List<dev_sitecorevisit> visits = crmServiceContext.VisitSet
	.Where(x => x.dev_xdbcontactid.Id == contact.Id).ToList();
```

This reduced the way we retireve the related data by over half the amount of code from the original method. This also made the data retrieval method much easier to debug as well as understand. With a few modifications, we can see the simpler execution code for retrieving the data:

```c#
public void Execute(IServiceProvider serviceProvider)
        {
            // Obtain the tracing service
            ITracingService tracingService =
                (ITracingService)serviceProvider.GetService(typeof(ITracingService));

            // Obtain the execution context from the service provider.  
            IPluginExecutionContext context = (IPluginExecutionContext)
                serviceProvider.GetService(typeof(IPluginExecutionContext));

    		// Interface to allow plug-ins to obtain IOrganizational Service
            IOrganizationServiceFactory factory =
                (IOrganizationServiceFactory)serviceProvider.GetService(typeof(IOrganizationServiceFactory));
			
    		// Interface containing methods provided by Organization Service
            IOrganizationService service = factory.CreateOrganizationService(context.InitiatingUserId);

            try
            {
                // Represents a souce of entities bound to a CRM service. It tracks and manages changes to the retrieved entities
                EntityService crmServiceContext = new EntityService(service);
				
                // Context parameters from the caller
                string leadId = context.InputParameters["leadid"] as string;

                if (ValidateString(leadId, context))
                {
                    // Retrieves the contact from the XdbcontactSet binding set where the leadid mathches the parameter id
                    dev_sitecorexdbcontact contact = crmServiceContext.XdbcontactSet.FirstOrDefault(x =>
                        x.dev_dev_sitecorexdbcontact_leadid_lead.Id == new Guid(leadId));
                    
				  // Retrieves a list of sitecore visits that have the same contact id as the contact above
                    List<dev_sitecorevisit> visits = crmServiceContext.VisitSet
                        .Where(x => x.dev_xdbcontactid.Id == contact.Id).ToList();
					
                    // Sets the output parameters to the returned value of the ValidateVisits function and sends it back to the caller
                    context.OutputParameters["data"] = ValidateVisits(visits, contact);
                }
            }
            catch (FaultException<OrganizationServiceFault> ex)
            {
                context.OutputParameters["data"] = new
                {
                    status = "Failed",
                    code = 02,
                    error = "An error occurred when trying to perform this task. Please refresh the page and try again."
                };
            }

            catch (Exception ex)
            {
                tracingService.Trace("FollowUpPlugin: {0}", ex.ToString());
                throw;
            }
        }
```

In the method above, we can see that the returned value of `context.OutputParameters["data"]` is set to the returned value of the method `ValidateVisits(visits, contact).` There are several steps prior to returning the data, these include the calculations of each field as well as the validation of data.

There are four calculations that are required within the requirements of this project, these being:

**Total Visit Count**

```c#
/// <summary>Gets the count of visits from the list of visits</summary>
/// <param name="visits">The visits.</param>
/// <returns>Count if there are any visits, else zero</returns>
private static int SitecoreVisitCount(IReadOnlyCollection<dev_sitecorevisit> visits)
{
	return visits.Any() ? visits.Count() : 0;
}
```

**Average Duration**

```c#
/// <summary>Gets the average visit time from the list of visits</summary>
/// <param name="visits">The visits.</param>
/// <returns>Average if there are any visits, else zero</returns>
private static double SitecoreVisitAverageDuration(IReadOnlyCollection<dev_sitecorevisit> visits)
{
	return visits.Any() ? Math.Round(visits.Average(x => double.Parse(x.dev_duration)), 2) : 0;
}
```

**Total Page Views**

```c#
/// <summary>Gets the sum of page views from the list of visits</summary>
/// <param name="visits">The visits.</param>
/// <returns>Sum if there are any visits, else zero</returns>
private static int? SitecorePageViews(IReadOnlyCollection<dev_sitecorevisit> visits)
{
	return visits.Any() ? visits.Sum(x => x.dev_pageviews) : 0;
}
```

**Total Goal Value**

```c#
/// <summary>Gets the sum of visits values from the list of visits</summary>
/// <param name="visits">The visits.</param>
/// <returns>Sum if there are any visits, else zero</returns>
private static double SitecoreVisitValue(IReadOnlyCollection<dev_sitecorevisit> visits)
{
	return visits.Any() ? Math.Round((double)visits.Sum(x => x.dev_value), 2) : 0;
}
```

Each of these methods were original a set of if/else statements, however these were simplified to using conditional operators.

These were all called using the `ValidateVisits` method as seen below:

```c#
private static string ValidateVisits(List<dev_sitecorevisit> visits, dev_sitecorexdbcontact contact)
        {
            if (visits.Count > 0) // Firstly check to see if there actually is any visits
            {
                return JsonConvert.SerializeObject( // Return as a object converted into a JSON string on success
                    new
                    {
                        status = "success",
                        contactAliasId = contact.dev_xdbaliasid,
                        count = SitecoreVisitCount(visits),
                        duration = SitecoreVisitAverageDuration(visits),
                        views = SitecorePageViews(visits),
                        value = SitecoreVisitValue(visits),
                        visits
                    });
            }
            return JsonConvert.SerializeObject( // Return as a object converted into a JSON string when no visits present
                new
                {
                    status = "Failed",
                    code = 01,
                    error = "There are no visits related to this entity"
                });
        }
```

This week was a big leap in the development as the code was further refined and refactored to mitigate more bugs as well as overall complexity. By changing the data retrieval method from my previous method to using LINQ, the data retrieval method was both simplified and easier to read. A lot was learned this week in regards to C# which is great as it is a key aspect of this project. Not much was done in the way of the custom control as the plugin was the main focus of the week.

------

# Journal 8 - 14/09/2022

As we are running on trial instances of Dynamics 365 Sales, this week required the creation of a new environment. Prior to the trial ending, it was crucial that the previous solution was exported so that we didn't loose the configurations made to the system. This would allow the continuation in the new instance from where we had left off in the previous. 

There were initially a couple of issues, one being that when trying to retrieve the data via Postman (and the control), an authentication error was being thrown. This was due to their not being security roles provided to the current user that allows read and write privilege's for the new instance. This was an easy fix as all that was needed was to assign FuseIT a new application user of type "System Administrator." This is the easiest option as it gives full access rights to the user, however for deployment, it is likely that you would change this depending on who is using the system.

Testing was done to make sure that data retrieval and creation worked as it did with the previous environment. This was successful, which allowed for development to continue on other areas. The next step from the previous week, where we used entities and services rather than FetchXML, was to develop the plugin further to handle more functionality, as well as split code up into their purposes. At the stage that the project was at, all code (other than the validation) was executed within the main execution method. At the stage at it was at, this was not an issue, however if we wanted to reuse functionality or add more, we would have duplicate code or methods that have multiple purposes. 

Ideally, the methods and entities would be split up into their own separate files in folders relative to what they are. For example:

```spreadsheet
DataRetievalPlugin.
│   DataRetrievalPlugin.cs   
│   DataRetrievalPlugin.csproj
|   app.config
|   packages.config
|
└───Services
│   │   JsonService.cs
│   │   SitecoreService.cs
│   │   VisitService.cs
│   │   EntityService.cs
│
└───Entities
│   │   SitecoreXDBContactEntity.cs
│   │   SitecoreVisitEntity.cs
│   
└───Properties
    │   AssemblyInfo.cs
```

By splitting code up into different files, it becomes much easier to manage especially as the plugin becomes more complex. The execute method was then reduced to the following:

```c#
private readonly VisitService _visitService = new VisitService();

public void Execute(IServiceProvider serviceProvider)
{
    ITracingService tracingService = (ITracingService)serviceProvider.GetService(typeof(ITracingService));
    IPluginExecutionContext context = (IPluginExecutionContext) serviceProvider.GetService(typeof(IPluginExecutionContext));
    IOrganizationServiceFactory factory = (IOrganizationServiceFactory)serviceProvider.GetService(typeof(IOrganizationServiceFactory));
    IOrganizationService service = factory.CreateOrganizationService(context.InitiatingUserId);

    try
    {
    	string result = _visitService.GetVisits(context, service);
    	context.OutputParameters["data"] = result;
    }
    catch (Exception e)
    {
    	context.OutputParameters["data"] = e.Message;
    }
}	
```

A key difference that we can see is that a variable has been introduced called `_visitService.` This is used when we want to query data from the visit collection created using the Early Bound Generator. Within the `VisitService.cs` file, we have a method `GetVisits()` which passes the parameters `context` and `service.` These are used to create a new crm service and bind the returned values to the context output parameters.

Whenever we want to make a query, we use an `Entity Service`: 

```c#
EntityService crmServiceContext = new EntityService(service);
```

With this, we can make queries to any of the collections that have been generated. For the visits related to a given contact, we use the following code:

```c#
dev_sitecorexdbcontact contact = crmServiceContext.XdbcontactSet.FirstOrDefault(x => x.dev_leadid.Id == new Guid(leadId));
List<dev_sitecorevisit> visits = crmServiceContext.VisitSet.Where(x => x.dev_xdbcontactid.Id == contact.Id).ToList();
```

This gives us all the of the data needed to perform the calculations prior to returning the data back to the custom control when the following is run:

```c#
string result = _visitService.GetVisits(context, service);
context.OutputParameters["data"] = result;
```

Overall, this week a lot of progress was made as well as a lot of new learning. There is still much more to achieve and plenty more to learn throughout this project. The goal for next week is to continue work on developing the main component.

------

# Journal 9 - 21/09/2022

The focus of this week was to do more work on the custom control. There was an issue where the data was not passing to the main component. To find where the issue was occurring, debugger code was placed in each of the steps like follows:

```typescript
console.log(1);
console.log(result);
debugger;
```

This would pause the execution at the given point and tell me where it was with the first console log, and what the current state of the response looked like.

It was discovered that an error was being thrown in the following section:

```typescript
self._data.status = stats.data.status || "";
self._data.contactAliasId = stats.data.contactAliasId || "";
self._data.count = stats.data.count || 0;
self._data.duration = stats.data.duration || 0;
self._data.views = stats.data.views || 0;
self._data.value = stats.data.value || 0;
```

The reason for this is that `stats` is equal to `JSON.parse(response.data)` whereas the way it was written above would be if `stats` were to equal `JSON.parse(response).` This is a simple fix as the data part of each declaration needed to be removed.

The correct method is as follows:

```typescript
self._data.status = stats.status || "";
self._data.contactAliasId = stats.contactAliasId || "";
self._data.count = stats.count || 0;
self._data.duration = stats.duration || 0;
self._data.views = stats.views || 0;
self._data.value = stats.value || 0;
```

This fixed the issue and the data was then loading as desired. The next task was to work on getting the list of visits and mapping them to the ISitecoreVisit Interface. Getting the details was the easy part, however figuring out each of the attributes was a struggle. At first it was though that the following method would achieve the goal:

```typescript
stats.visits.map((visit: any) => {
    visits.push({
        id: visit.Id,
        interactionId: visit.dev_iteractionid,
        views: visit.dev_pageviews,
        browser: visit.dev_browser,
        value: visit.dev_value,
        channel: visit.channel
    })
});
```

However, after reviewing the returned object, the attributes of the visits are in a sub-array called `Attributes` where each item is an object with a key and value as shown below:

```json
[
    {
        "Key": "dev_interactionid",
        "Value": "7785a78a-552e-ed11-9db1-00224893bf8a"
    },
    {
        "Key": "dev_pageviews",
        "Value": 2
    },
    {
        "Key": "dev_browser",
        "Value": "chrome"
    },
    {
        "Key": "dev_value",
        "Value": 3
    },
    {
        "Key": "dev_channel",
        "Value": "MOBILE_WEB"
    }
]
```

To solve this, the following changes were made:

```typescript
stats.visits.map((visit: any) => {
    visits.push({
        id: visit.Id,
        interactionId: visit.Attributes[0].Value, // Instead of visit.dev_iteractionid
        views: visit.Attributes[1].Value, // Instead of visit.dev_pageviews
        browser: visit.Attributes[2].Value, // Instead of visit.dev_browser
        value: visit.Attributes[3].Value, // Instead of visit.dev_value
        channel: visit.Attributes[4].Value // Instead of visit.channel
    })
});
```

This may not be the best option as a change in the order returned would cause data issues.

Another key important part of the project that was focussed on this week was standardisation of components. This means to keep with the standards that Dynamics and Power Apps use for designing their controls. Power Apps uses a framework called FluentUI which is developed by Microsoft. This is similar to Bootstrap and allows for the use of custom components that are consitent with those that Power Apps and Dynamics uses.

The first and major change was the table of visits. FluentUI has a component called a detailslist. This is essentially a table that allows for sorting, grouping, and filtering of data. These are very customisable and allow for columns to include such things as links and buttons.

Below is the table after changing it to use FluentUI:

```typescript
<MarqueeSelection selection={this._selection}>
  <DetailsList
    items={value}
    columns={this._columns}
    setKey="set"
    layoutMode={DetailsListLayoutMode.justified}
    selectionPreservedOnEmptyClick={true}
    ariaLabelForSelectionColumn="Toggle selection"
    ariaLabelForSelectAllCheckbox="Toggle selection for all items"
    checkButtonAriaLabel="select row"
  />
</MarqueeSelection>
```

This passes the `this._value` as the data and `this._columns` (which are created previously) as the table columns.

Overall, this week was very productive and a lot of progress was made on developing the project.

------

# Journal 10 - 28/09/2022

This week some further work was done on standardisation. Last week the table was changed from a normal HTML/TSX table into a FluentUI table. However, the rest of the layout was still using normal `div's` to created the components layout. After doing some research, it was discovered that FluentUI has a component called a `Stack` (parent element) that has sub components called `Stack.Item` (child element). These work in a similar way to that of grid CSS.

You simply create a `Stack` like you would a `div` and add `Stack.Item`'s to them':

```typescript
<Stack>
	<Stack.Item></Stack.Item>
</Stack>
```

These are useful as you can define whether the stack goes horizontally or vertically. After a bit of time, the following was created:

```typescript
<Stack>
    {/* Main title */}
    <StackItem style={styles.title}>
        Sitecore Guest
    </StackItem>
    {/* Sitecore Summary Sectionn */}
    <StackItem style={styles.section}>
        <Stack horizontal horizontalAlign='center'>
            <StackItem>
                <Stack style={styles.secondary}>
                    <StackItem style={styles.header}>
                        {this.props.Statistics.value}
                    </StackItem>
                    <StackItem style={styles.subHeader}>
                        Goal Value
                    </StackItem>
                </Stack>
            </StackItem>
            <StackItem>
                <Stack style={styles.secondary}>
                    <StackItem style={styles.header}>
                        {this.props.Statistics.count}
                    </StackItem>
                    <StackItem style={styles.subHeader}>
                        Visit Count
                    </StackItem>
                </Stack>
            </StackItem>
            <StackItem>
                <Stack style={styles.secondary}>
                    <StackItem style={styles.header}>
                        {this.props.Statistics.duration}
                    </StackItem>
                    <StackItem style={styles.subHeader}>
                        Avg Visit Time
                    </StackItem>
                </Stack>
            </StackItem>
            <StackItem>
                <Stack style={styles.secondary}>
                    <StackItem style={styles.header}>
                        {this.props.Statistics.views}
                    </StackItem>
                    <StackItem style={styles.subHeader}>
                        Views
                    </StackItem>
                </Stack>
            </StackItem>
        </Stack>
        <Stack horizontal horizontalAlign='center'>
            {/* Refresh Analytics Button */}
            <StackItem style={styles.button}>
                <PrimaryButton text='Refresh Analytics' onClick={this.refresh} />
            </StackItem>
        </Stack>
        <StackItem align="auto">
            <Spinner size={SpinnerSize.large} label="Refreshing Analytics..." style={{display: display}} />
        </StackItem>
    </StackItem>

    {/* Secondary Title */}
    <StackItem style={styles.subTitle}>
        Sitecore Sessions
    </StackItem>
    {/* List of visits */}
    <StackItem style={styles.section}>
        <VisitsDetailsList data={this.props.Statistics.visits} />
    </StackItem>
</Stack>
```

As you can see, each of the components have their own style relative to what they are. Rather than doing inline styling, we can create style objects that have a set of defined styles. For example:

```typescript
const styles = {
    title: {
        "color": palette.midBlue,
        "fontSize": "2.25rem",
        "fontWeight": "bold",
        "margin": "0.5rem",
        "padding": "0.5rem",
    },
    subTitle: {
        "color": palette.midBlue,
        "fontSize": "1.5rem",
        "fontWeight": "bold",
        "margin": "0.5rem",
        "padding": "0.5rem",
    },
    header: {
        "color": palette.darkBlue,
        "fontSize": "1.75rem",
        "fontWeight": "bold"
    },
    subHeader: {
        "color": palette.darkBlue,
        "fontSize": "1.5rem",
        "fontWeight": "lighter"
    },
    section: {
        "border": "3px solid" + palette.lightBlue,
        "padding": "10px",
        "margin": "10px",
    },
    secondary: {
        "border": "3px solid" + palette.lightGrey,
        "padding": "10px",
        "margin": "10px",
    },
    button: {
        "margin": "1rem",
    }
}
```

Additionally, colour palettes can be created in the same way  and used throughout the style objects. The colour palette shown below is the same that is used in Microsoft Dynamics:

```typescript
const palette = {
    darkBlue: "#002050",
    midBlue: "#1160b7",
    lightBlue: "#b1d6f0",
    lightGrey: "#dfe2e8",
    red: "#d24726"
}
```

As this weeks focus was more on the standardising of the components, not a lot was done in other areas. This was mainly due to working from home and needing to spend some time tidying up the component before further work was done on the plugins. However, with what was achieved this week, there isn't much left to do component side, just plugin side.

------

# Week 11 - 05/10/2022

The focus of this week was to work on the outbound request to the Sitecore web service. At this stage, the web server was not ready for the system to connect too, so an Azure static app was created to host some test files that we could make http requests towards. This is a simple app that has two routes, one for testing the refresh of analytics and one for getting individual data of a visit. When a `HTTP GET` request is made, a JSON string is returned to the called. 

Below is an example of the data from the analytics route:

```json
{
    "status": "success",
    "contactAliasId": "123",
    "visits": [{
        "interactionid": "C6732455-439E-0000-0000-0639A894F065",
        "name": "2021.255.10438",
        "startdate": "2021-09-08 20:15:46.98Z",
        "enddate": "2021-09-09 21:22:22.65Z",
        "browser": "Chrome v102.0",
        "channel": "direct",
        "duration": "00:02:22.2023053",
        "views": 1,
        "value": 1,
        "campaign": "1234567890",
        "goals": [{
            "goalid": "637670953205500000_a24a09da-a5b3-0000-0000-063a758176ad_b0c5a8de-8355-49be-ba71-37b770d3471a",
            "date": "2021-09-13T02:02:00.000Z",
            "path": "/products/request-quote",
            "value": 10
        },
        {
            "goalid": "637670953205500000_a24a09da-a5b3-0000-0000-063a758176ad_b0c5a8de-8355-49be-ba71-37b770d3471a",
            "date": "2022-09-13T02:02:00.000Z",
            "path": "/products/request-quote2",
            "value": 22
        }]
    }, {
        "interactionid": "AS3AAA25-439E-0000-0000-0639A894F065",
        "name": "2021.255.10438",
        "startdate": "2021-09-08 20:15:46.98Z",
        "enddate": "2021-09-08 20:55:32.65Z",
        "browser": "Chrome v102.0",
        "channel": "direct",
        "duration": "00:02:22.2023053",
        "views": 9,
        "value": 33,
        "campaign": "1234567890",
        "goals": [{
            "goalid": "637670953205500000_a24a09da-a5b3-0000-0000-063a758176ad_b0c5a8de-8355-49be-ba71-37b770d3471a",
            "date": "2021-09-13T02:02:00.000Z",
            "path": "/products/request-quote",
            "value": 10
        }]
    }]
}
```

And here is an example of the data from the events route:

```json
{
    "interactionid": "C6732455-439E-0000-0000-0639A894F065",
    "type": "PageViewEvent",
    "timestamp" : "2022-09-01T20:10:46.98Z",
    "url" : "https://sitename.com/path/to/page1"
}
```

Developing the methods required to make these outbound requests was rather straight forward. There were no issues with this due to prior knowledge of making HTTP requests in .NET. The only area that was new was adding custom headers to the request to pass the Sitecore API key to the web server. After a quick Google search, the following method was implemented:

```c#
client.DefaultRequestHeaders.Add("x-api-key", config.SitecoreApiKey);
```

This adds the specified header and its value to the `System.Net.Http.Headers.HttpHeaders` collection. The http method is as follows (stripped additional code):

```c#
public static async Task<string> PostAsync(SitecoreConfig config, string id, PluginContext context)
        {
            try
            {
                using (HttpClient client = context.HttpClient)
                {
                    client.DefaultRequestHeaders.Add("x-api-key", config.SitecoreApiKey);
                    
                    Dictionary<string, string> parameters = new Dictionary<string, string>()
                    {
                        {
                            "maxVisits", config.MaxVisits.ToString()
                        }
                    };

                    HttpResponseMessage response = await client.PostAsync(config.SitecoreApiUrl + "GetData",
                        new FormUrlEncodedContent(parameters));
                    
                    string contents = await response.Content.ReadAsStringAsync();
                    //Other Business Logic
                }
            }
            catch (Exception e)
            {
                //Exception Handling
            }
        }
```

This method uses the HTTP client generated in the `PluginContext` file, adding the Sitecore API key as a custom header and the config max visits in the body. As the config URL is not specific to a route, we add the route name to the end in the `PostAsync()` method.

This same method is consistent across the `UpdateAnalytics` plugin and the `IndividualDetails` plugin. However, the section marked above as `Other Business Logic` differs between the two plugins.

Implementation with the custom control to make the requests was the same as which the method is implemented for retrieving the data:

```typescript
private refresh = () => {
        let request = {
            leadid: this.props.Statistics.id,
            getMetadata: function () {
                return {
                    boundParameter: null,
                    operationType: 0,
                    operationName: 'dev_RefreshAnalytics',
                    parameterTypes: {
                        'leadid': {
                            typeName: 'Edm.String',
                            structuralProperty: 1,
                        },
                    },
                };
            },
        };

        Xrm.WebApi.online.execute(request).then(
            (result: any) => {
                // Other Business Logic
            },
            (error: any) => {
                // Other Business Logic
            }
        );
    };
```

1. We define the request passing the `leadid` as a string parameter, and defining the `operationType` as unbound (0), and the  operation name as `dev_RefreshAnalytics`
2. We execute the request using the `Xrm.WebApi.online.execute()` method and wait for a response.
3. If the response is successful then we perform the business logic for a successful response, else we do something with the error

Overall, this week was very successful. A lot of progress was made on the project and it is almost at the stage of being a minimal proof-of-concept product. The component is working as it should in the "best case scenario," but this is what is required of me from the work placement. However, some additionally error handling should be implemented in the next couple of weeks. All that is left to do is some code refactoring, code commenting, and a write up on the system to provide to FuseIT.

------

# Week 12 - 12/10/2022

3262 - 3762

The focus of this week was to refactor the code and start the documentation process. I was asked that the code was commented and formatted in a way that would be easy to be picked up by someone else and be understandable. This includes such things as file structure, naming conventions, consistency across the plugins, and code comments.

One of the problems when developing this project was that each requirement was a learning process. As one feature was implemented, better ways of doing so were learned throughout the work placement. As there were three plugins developed, this meant that the first plugin was the earliest implementation, while the others had improvements along the way. This is because with the given timeframe, it was necessary to move on to the next plugin once the previous was working. The result of this was that each plugin had their own way of doing similar tasks. This mean that if you were to compare each plugin, there were differences in the tasks that do the same thing.

Looking at the high level layout of the plugins, they could all be designed to use a similar file structure. Below is the updated design for the plugin that retrieves analytics:

```
📦Plugin_Retrieve_Analytics
| 📜PluginBase.cs
|
└───📁Context
|	| 📜PluginContext.cs
|
└───📁Entities
|	└───📁BaseEntities
|	|	| 📜SitecoreSummary.cs
|	|
|	└───📁SitecoreEntities
|	|	| 📜SitecoreVisitEntity.cs
|	|	| 📜SitecoreXDBContactEntity.cs
|	|
└───📁Services
|	| 📜DynamicsService.cs
|	| 📜EntityService.cs
|	| 📜OptionSets.cs
|
└───📁Utilities
|	| 📜JsonUtil.cs
|	| 📜StringUtil.cs
```

Comparing it to the plugin for individual events, there are only minor differences:

```
📦Plugin_Individual_Events
| 📜PluginBase.cs
|
└───📁Context
|	| 📜PluginContext.cs
|
└───📁Entities
|	└───📁BaseEntities
|	|	| 📜EventEntityBase.cs
|	|	| 📜ResponseMessage.cs
|	|	| 📜SitecoreConfig.cs 
|	|
|	└───📁SitecoreEntities
|	|	| 📜EnvironmentVariableDefinitionEntity.cs
|	|	| 📜EnvironmentVariableValueEntity.cs
|	|	| 📜SitecoreVisitEntity.cs
|	|	| 📜SitecoreXDBContactEntity.cs
|
└───📁Services
|	| 📜DynamicsService.cs
|	| 📜EntityService.cs
|	| 📜OptionSets.cs
|	| 📜SitecoreService.cs
|
└───📁Utilities
|	| 📜JsonUtil.cs
|	| 📜StringUtil.cs
```

1. The base entities has extra classes in the plugin for individual events.
2. The Sitecore Entities has strongly types entities for the Dynamics Environment Variable in the plugin for individual events.
3. The plugin that retrieves analytics does not have the `SitecoreService.cs` file in the `Services` folder as it does not make any requests to Sitecore.

The design for the analytics refresh plugin is almost the same as the plugin for individual events, however it has a couple of additional `BaseEntities`

One major change from the original design was the extraction of the initialization of all the contexts and services. Originally, these were all placed within the `Execute()` method in the `PluginBase` file. It was decided that these would be moved into their own class that could be used across multiple files.

Below is the original design:

```c#
public void Execute(IServiceProvider serviceProvider)
{
    // Obtain the tracing service
    ITracingService tracingService =
        (ITracingService)serviceProvider.GetService(typeof(ITracingService));

    // Obtain the execution context from the service provider.  
    IPluginExecutionContext context = (IPluginExecutionContext)
        serviceProvider.GetService(typeof(IPluginExecutionContext));

    IOrganizationServiceFactory factory =
        (IOrganizationServiceFactory)serviceProvider.GetService(typeof(IOrganizationServiceFactory));

    IOrganizationService service = factory.CreateOrganizationService(context.InitiatingUserId);
	
    // Represents a source of entities bound to a CRM service. It tracks and manages changes made to the retrieved entities.
    EntityService entityService = new EntityService();
    
    //Other Business Logic
}
```

And here is the class:

```c#
using System;
using System.Net.Http;
using Microsoft.Xrm.Sdk;
using Plugin_Refresh_Analytics.Entities.SitecoreEntities;

namespace Plugin_Refresh_Analytics.Context
{
    /// <summary>
    ///     Contains all methods for Dynamics/Microsoft Dataverse Actions
    /// </summary>
    public class PluginContext
    {
        // Represents a source of entities bound to a CRM service. It tracks and manages changes made to the retrieved entities.
        private EntityService _entityService;

        // Interface for plug-ins to provide trace information. Used for debugging and/or troubleshooting plug-in issues or behaviors are complicated
        // without rich and insightful logging or tracing.
        private ITracingService _tracingService;

        // Defines the contextual information passed to a plug-in at run-time.
        // Contains information that describes the run-time environment that the plug-in is executing in,
        // information related to the execution pipeline, and entity business information.
        private IPluginExecutionContext _context;

        // Represents a factory for creating IOrganizationService instances
        private IOrganizationServiceFactory _factory;

        // Provides programmatic access to the metadata and data for an organization.
        // Contains methods provided by Organization Service. Used in locations where user is granted access
        private IOrganizationService _userOrganizationService;

        // Provides programmatic access to the metadata and data for an organization.
        // Contains methods provided by Organization Service. Used in locations where only system can access
        private IOrganizationService _systemOrganizationService;

        // Provides a class for sending HTTP requests and receiving HTTP responses from a resource identified by a URI.
        private HttpClient _httpClient;

        // Constructor for PluginContext class containing IServiceProvider parameter.
        // IServiceProvider defines a mechanism for retrieving a service object; that is, an object that provides custom support to other objects.
        public PluginContext(IServiceProvider serviceProvider)
        {
            try
            {
                // If the service provider is null, throw an error.
                if (serviceProvider == null)
                {
                    throw new InvalidPluginExecutionException("serviceProvider");
                }

                // Obtain the tracing service from the service provider
                _tracingService = (ITracingService)serviceProvider.GetService(typeof(ITracingService));

                // Obtain the execution context from the service provider. 
                _context = (IPluginExecutionContext)
                    serviceProvider.GetService(typeof(IPluginExecutionContext));

                // Get the OrganizationServiceFactory from the service provider
                _factory =
                    (IOrganizationServiceFactory)serviceProvider.GetService(typeof(IOrganizationServiceFactory));

                //Create the organization service using the OrganizationServiceFactory passing the id of the user that initiated the web service
                _userOrganizationService = _factory.CreateOrganizationService(_context.InitiatingUserId);

                //Create the organization service using the OrganizationServiceFactory passing a null value indicating that its a system user
                _systemOrganizationService = _factory.CreateOrganizationService(null);

                // Creating a new instance of the entity service passing the system organization service
                _entityService = new EntityService(_systemOrganizationService);

                HttpClientHandler handler = new HttpClientHandler();

                // Initializes a new instance of the HttpClient class
                _httpClient = new HttpClient(handler, false);
            }
            catch (Exception e)
            {
                // Trace the error message
                _tracingService?.Trace(e.Message);
                throw;
            }
        }

        /// <summary>
        /// Defining the public PluginContext attributes
        /// </summary>
        public EntityService Service
        {
            get => _entityService;
            set => _entityService = value;
        }

        public ITracingService TracingService
        {
            get => _tracingService;
            set => _tracingService = value;
        }

        public IPluginExecutionContext Context
        {
            get => _context;
            set => _context = value;
        }

        public IOrganizationServiceFactory Factory
        {
            get => _factory;
            set => _factory = value;
        }

        public IOrganizationService UserOrganizationService
        {
            get => _userOrganizationService;
            set => _userOrganizationService = value;
        }

        public IOrganizationService SystemOrganizationService
        {
            get => _systemOrganizationService;
            set => _systemOrganizationService = value;
        }

        public HttpClient HttpClient
        {
            get => _httpClient;
            set => _httpClient = value;
        }
    }
}
```

And here is the creation of the new instnace:

```c#
public void Execute(IServiceProvider serviceProvider)
{
    if (serviceProvider == null) throw new InvalidPluginExecutionException("serviceProvider");
	PluginContext pluginContext = new PluginContext(serviceProvider);
    // Other Business Logic
}
```

Additionally, two additional attributes were added:

1. The Http Client, so that we weren't creating a new one each time the request was called to make an outbound request.
2. `_systemOrganizationService = _factory.CreateOrganizationService(null);` was added for when operations needed to be executed that were not accessible by someone without the correct privileges.

Overall, this week a lot of progress was made for the project. Each of the aspects of the project (plugins and custom component) we refactored and made to be consistent with each other, redundant code was removed, and all of the code was commented. Not much was learned this week as the focus was to tidy up what was already working, rather than adding additional functionality.

------

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

## Dynamics Solution

### File Structure:

![img](https://imgur.com/dUVf5v5.png)

### Custom Entities

- **Sitecore Behavior** 
- **Sitecore Behavior Profile Key**
- **Sitecore Goal**
- **Sitecore Visit**
- **Sitecore XDB Contact**

### Custom Processes (Actions)

- **GetData**
  - Action used when request is made to get analytics details of a given lead. Triggers plugin for individual visits. Expects a `leadid` input and optionally has a `data` output.
- **GetEvents**
  - Action used when request is made to get details of an individual visit. Triggers plugin for individual visits. Expects a `visitid` input and optionally has a `data` output.
- **RefreshAnalytics**
  - Action used when request is made to refresh the analytics. Triggers plugin for analytics refresh. Expects a `leadid` input and optionally has a `data` output.

### Custom Control (PCF Component)

- **SitecoreSummaryComponent**
  - Display Name: dev_Fuseit.S4D.Dynamics.SitecoreSummaryComponent
  - Does not require a set value when added to the form. Selecting a value will not do anything in the way of functionality. Selecting `lead` may cause the component to get stuck on the form as it is a required field (do not try). 

### Environment Variables

- **Sitecore Configuration**
  - Required to make outbound requests to the web server.
  - Set in the "current value" textbox.
  - Attribute naming must be exactly the same as: `{"SitecoreApiUrl" : "<URL>","SitecoreApiKey" : "<API Key>","maxVisits" : <Number of visits> }`
    - The API URL and the API Key must be a string format (between quotations " ")
    - The maxVisits must be an integer type (no decimals and no quotation marks " ")

### Other

- Contact and Lead are added when you create lookup fields on an entity. These should not be deleted or modified.

------

## GitHub Folder

### General File Structure

```
📦FuseInfoTech/S4D-Dynamics:BRANCH:dev/proof-of-concept 
│   📄README.md  
└───📁Plugins
│   └───📦Plugin-GetAnalytics
│   └───📦Plugin-IndividualEvents
│   └───📦Plugin-RefreshAnalytics 
└───📁Components
│   └───📦SitecoreLeadSummaryComponent 
└───Tools
│   └───📁ConfigurationMigration 
│   └───📁CoreTools 
│   └───📁PackageDeployment 
│   └───📁PluginRegistration 
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
📦Plugin_Retrieve_Analytics
| 📜PluginBase.cs
|
└───📁Context
|	| 📜PluginContext.cs
|
└───📁Entities
|	└───📁BaseEntities
|	|	| 📜SitecoreSummary.cs
|	|
|	└───📁SitecoreEntities
|	|	| 📜SitecoreVisitEntity.cs
|	|	| 📜SitecoreXDBContactEntity.cs
|	|
└───📁Services
|	| 📜DynamicsService.cs
|	| 📜EntityService.cs
|	| 📜OptionSets.cs
|
└───📁Utilities
|	| 📜JsonUtil.cs
|	| 📜StringUtil.cs
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

------

### Plugin_Individual_Events

#### Purpose

The purpose of this plugin is to make an on-demand outbound request to the Sitecore Web Server to retrieve events relative to a given visit. The Web server returns an array of event objects containing the URL, visit type, and timestamp of the event. This data is then passed back to the sender.

*Source:* *https://github.com/FuseInfoTech/S4D-Dynamics/tree/dev/proof-of-concept/Plugins/Plugin-IndividualEvents/Plugin_Individual_Events/Plugin_Individual_Events*

*File Path:* [*C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Plugins\Plugin-IndividualEvents\Plugin_Individual_Events/Plugin_Individual_Events.sln*](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Plugins\Plugin-IndividualEvents\Plugin_Individual_Events/Plugin_Individual_Events.sln)

#### File Structure

**NOTE**: I have only listed the important files for this solution. These are ones that I have worked with and will likely only need to me added to/modified.

```
📦Plugin_Individual_Events
| 📜PluginBase.cs
|
└───📁Context
|	| 📜PluginContext.cs
|
└───📁Entities
|	└───📁BaseEntities
|	|	| 📜EventEntityBase.cs
|	|	| 📜ResponseMessage.cs
|	|	| 📜SitecoreConfig.cs 
|	|
|	└───📁SitecoreEntities
|	|	| 📜EnvironmentVariableDefinitionEntity.cs
|	|	| 📜EnvironmentVariableValueEntity.cs
|	|	| 📜SitecoreVisitEntity.cs
|	|	| 📜SitecoreXDBContactEntity.cs
|
└───📁Services
|	| 📜DynamicsService.cs
|	| 📜EntityService.cs
|	| 📜OptionSets.cs
|	| 📜SitecoreService.cs
|
└───📁Utilities
|	| 📜JsonUtil.cs
|	| 📜StringUtil.cs
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

------

### Plugin_Refresh_Analytics

#### Purpose

The purpose of this plugin is to make an outbound request to the Sitecore web service, return a set of visits, determine whether they exist, and update or insert the visits into Dynamics. This plugin does not return a set of data, just a response message (success/error).

*Source:* *https://github.com/FuseInfoTech/S4D-Dynamics/tree/dev/proof-of-concept/Plugins/Plugin-RefreshSitecoreAnalytics/Plugin_Refresh_Analytics/Plugin_Refresh_Analytics*

*File Path:* [*C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Plugins\Plugin-RefreshSitecoreAnalytics\Plugin_Refresh_Analytics/Plugin_Refresh_Analytics.sln*](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Plugins\Plugin-RefreshSitecoreAnalytics\Plugin_Refresh_Analytics/Plugin_Refresh_Analytics.sln)

#### File Structure

**NOTE**: I have only listed the important files for this solution. These are ones that I have worked with and will likely only need to me added to/modified.

```
📦Plugin_Refresh_Analytics
| 📜PluginBase.cs
|
└───📁Context
|	| 📜PluginContext.cs
|
└───📁Entities
|	└───📁BaseEntities
|	|	| 📜GoalEntityBase.cs
|	|	| 📜ResponseEntity
|	|	| 📜ResponseMessage.cs
|	|	| 📜SitecoreConfig.cs
|	|	| 📜VisitEntityBase.cs
|	|
|	└───📁SitecoreEntities
|	|	| 📜EnvironmentVariableDefinitionEntity.cs
|	|	| 📜EnvironmentVariableValueEntity.cs
|	|	| 📜SitecoreVisitEntity.cs
|	|	| 📜SitecoreXDBContactEntity.cs
|
└───📁Services
|	| 📜DynamicsService.cs
|	| 📜EntityService.cs
|	| 📜OptionSets.cs
|	| 📜SitecoreService.cs
|
└───📁Utilities
|	| 📜JsonUtil.cs
|	| 📜StringUtil.cs
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
📦SitecoreLeadSummaryComponent
└───📁SitecoreSummaryComponent
|	📄ControlManifest.Input.xml
|	📜index.ts
|	└───📁Components
|	|	| 📜ComponentBase.tsx
|	|	| 📜ComponentBase.tsx
|	|
|	└───📁Solutions
|	|	└───📁src
|	|	| └───📁Other
|	|	|	| 📄Solution.xml
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
| ███████████                                                  | █████████████                                                | █████     | █████████████████████████████████████████████████████████████████████████ |
| Modify detailslists                                          | General                                                      | N/A       | C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx |
| Modify columns on the individual visits details detailslist  | `VisitsDetailsList()`                                        | 41        | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| Modify the column on the table that displays all visits      | `VisitsDetailsList()`                                        | 80        | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| The method for viewing details of a single visit on-demand   | `VisitsDetailsList()`                                        | 94        | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| Modify the html for the details list component               | `VisitsDetailsList()`                                        | 235       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| Modify the html for the details list for all visits          | `VisitsDetailsList()`                                        | 238       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| Modify the html for the modal popup                          | `VisitsDetailsList()`                                        | 251       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| Modify the html for the details list for individual visit details | `VisitsDetailsList()`                                        | 271       | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| Change the styling for the details list component            | `contentStyles`, `iconButtonStyles`, `stackTokens`, `stackStyles` | 296+      | [C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx](C:\Repos\FuseInfoTech\S4D-Dynamics\S4D-Dynamics\Components\SitecoreSummaryComponent\SitecoreLeadSummaryComponent\Components\DetailsList.tsx) |
| ███████████                                                  | █████████████                                                | █████     | █████████████████████████████████████████████████████████████████████████ |
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

------

# Week 14 - 26/10/2022
