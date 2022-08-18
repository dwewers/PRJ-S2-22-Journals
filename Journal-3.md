## Journal 3 - 18/08/2022

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
