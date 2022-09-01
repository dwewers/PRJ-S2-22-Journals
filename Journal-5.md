## Journal 5 - 31/08/2022

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

   -  Total Visit count 

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

### Getting started

It was then time to start on the final product. Initially, a plugin had been developed to create the custom entities and their relationships, however this would not be necessary as the final product would be a solution that can be imported by the client. This solution would contain all of the required entities rather than creating them when the solution is imported. To do this, the Power Apps tool was used to create each of the entities shown in the relational diagram provided. This did not take very long as it is mostly just point and click with a few configurations (setting string limits).

As we can see, all of the custom entities are displayed in the solutions tables list. You may notice that the lead and contacts table is also listed. This is because of the relationships with them (the lookup fields on the XDB entity).

![](https://i.imgur.com/EOhqvQq.png)

The next step was to start work on the plugin. It was discovered the XRM Toolbox has a very useful tool called "Early Bound Generator." This tool is used for creating early bound entities/option sets, and actions. As one of the requirements is that quote "Strongly-typed classes should be created to serialize and deserialize JSON," it is necessary that their are classes for each of the entities in the plugin. However, the code for creating these classes is nowhere near as simple as that of a basic C# class. Coding each and every entity would have been very time consuming and a general waste of time given that it can be generated through this tool. One of the custom entities, when converted to code, is about 800 lines of code. Multiply this by the five custom entities of different complexities, and we will end up with thousands of lines of code across five different classes (not including option sets).

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

