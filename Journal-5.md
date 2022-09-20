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
