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

Overall, this was a big week in the development of the project.

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

# Journal 9 - 21/09/2022 (TODO)
