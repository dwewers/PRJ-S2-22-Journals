


## Week 7

### Day 1:

- Fixed the plugin
- Working on component

### Day 2:

- Debugging

I was having issues where the data was not passing to the main component. I decided that I would place some debugger code in each of the steps like follows:

```typescript
console.log(1);
console.log(result);
debugger;
```

This would pause the execution at the given point and tell me where it was with the first console log, and what the current state of the response looked like.

I discovered that an error was being thrown in the following section:

```typescript
self._data.status = stats.data.status || "";
self._data.contactAliasId = stats.data.contactAliasId || "";
self._data.count = stats.data.count || 0;
self._data.duration = stats.data.duration || 0;
self._data.views = stats.data.views || 0;
self._data.value = stats.data.value || 0;
```

The reason for this is that `stats` is equal to `JSON.parse(response.data)` whereas the way I have written it above would be if `stats` were to equal `JSON.parse(response).` This is a simple fix as I just need to remove the data part of each declaration.

The correct method is as follows:

```typescript
self._data.status = stats.status || "";
self._data.contactAliasId = stats.contactAliasId || "";
self._data.count = stats.count || 0;
self._data.duration = stats.duration || 0;
self._data.views = stats.views || 0;
self._data.value = stats.value || 0;
```

Data is now loading

I then needed to work on getting the list of visits and mapping them to the ISitecoreVisit Interface. Getting the details was the easy part, however figuring out each of the attributes was a struggle. At first I though I would be able to do the following:

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

However, after reviewing the returned object, I found that the attributes of the visits are in a sub-array called `Attributes` where each item is an object with a key and value as shown below:

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

The way I found that works is as follows:

```typescript
stats.visits.map((visit: any) => {
    visits.push({
        id: visit.Id,
        interactionId: visit.Attributes[0].Value,
        views: visit.Attributes[1].Value,
        browser: visit.Attributes[2].Value,
        value: visit.Attributes[3].Value,
        channel: visit.Attributes[4].Value
    })
});
```

This may not be the best option as a change in the order returned would cause data issues.

Used Fluent  UI to create the table, rather than normal TSX/HTML

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

This passes the `value` as the data and `columns` (which are created previously) as the table columns

# Journal 9 - 21/09/2022

## Week 8

### Day 1: Outward Bound Requests

- Data structure
- Data contracts
- Config

- URL
- Shared API Key
- Visit Count
- XDB Contact ID
- JSON

- Environment Variable - Power Apps Web Resource + Table

### Day 2:

Creating temp database:

1. Username: admin
2. Password: YFqGoXrru8Mz1PGn

Environment Variable

```typescript
//ERROR: Error occured during initialization of control: dev_FuseIT.CDP4D.Controls.S4Ddynamics1;Message: Option Parameter should begin with "?"

// CAUSE: Missing ? at the start of "$filter=(schemaname eq 'dev_sitecoreConfig')"
Xrm.WebApi.online.retrieveMultipleRecords("environmentvariabledefinition", "$filter=(schemaname eq 'dev_sitecoreConfig')").then(
            function success(result: any) {
                if (result.ok) {
                    result.json().then((response: any) => {
                        return response.value;
                    })
                }
            },
            function (error: any) {
                return error.message;
            }
        );
 // SOLUTION:
 // Changed "$filter=(schemaname eq 'dev_sitecoreConfig')" to "?$filter=(schemaname eq 'dev_sitecoreConfig')"
```

```typescript
/* ERROR:

Error Details: TypeError: Cannot read properties of undefined (reading 'then')
    at S4DLeadSummaryComponent1.init (webpack://pcf_tools_652ac3f36e1e4bca82eb3c1dc44e6fad/./S4DLeadSummaryComponent1/index.ts?:28:35)
    at https://orgb717200a.crm6.dynamics.com/uclient/scripts/9.js?v=1.4.4731-2208.5:1:252140
    at t._measureLifecycleMethod (https://orgb717200a.crm6.dynamics.com/uclient/scripts/9.js?v=1.4.4731-2208.5:1:249801)
    at t._initializeControl (https://orgb717200a.crm6.dynamics.com/uclient/scripts/9.js?v=1.4.4731-2208.5:1:252071)
    at https://orgb717200a.crm6.dynamics.com/uclient/scripts/9.js?v=1.4.4731-2208.5:1:239345
    at https://orgb717200a.crm6.dynamics.com/uclient/scripts/app.js?v=1.4.4731-2208.5:3983:801
    at Array.forEach (<anonymous>)
    at https://orgb717200a.crm6.dynamics.com/uclient/scripts/app.js?v=1.4.4731-2208.5:3983:790
    at https://orgb717200a.crm6.dynamics.com/uclient/scripts/app.js?v=1.4.4731-2208.5:3983:806
    at Object.hu [as unstable_batchedUpdates] (https://orgb717200a.crm6.dynamics.com/uclient/scripts/vendor.js?v=1.4.4731-2208.5:9:95393)

Activity Id: 519aecea-45bf-4a7b-ba4e-2d67b8b08b84
*/

//CAUSE:
 Xrm.WebApi.online.retrieveMultipleRecords("environmentvariabledefinition", "?$filter=(schemaname eq 'dev_sitecoreConfig')").then(
            function success(result: any) {
                if (result.ok) {
                    result.json().then((response: any) => {
                        return response.value; // I believe that value does not exist
                    })
                }
            },
            function (error: any) {
                return error.message;
            }
        );
//SOLUTION:
 Xrm.WebApi.online.retrieveMultipleRecords("environmentvariabledefinition", "?$filter=(schemaname eq 'dev_sitecoreConfig')").then(
            function success(result: any) {
                if (result.ok) {
                    result.json().then((response: any) => {
                        return response; // Just returning the response
                    })
                }
            },
            function (error: any) {
                return error.message;
            }
        )
```

```typescript
/* ERROR
Error Details: Error: Minified React error #185; visit https://reactjs.org/docs/error-decoder.html?invariant=185 for the full message or use the non-minified dev environment for full errors and additional helpful warnings.
    at uu (https://orgb717200a.crm6.dynamics.com/uclient/scripts/vendor.js?v=1.4.4731-2208.5:9:92321)
    at Object.enqueueSetState (https://orgb717200a.crm6.dynamics.com/uclient/scripts/vendor.js?v=1.4.4731-2208.5:9:48626)
    at y.setState (https://orgb717200a.crm6.dynamics.com/uclient/scripts/vendor.js?v=1.4.4731-2208.5:25:1308)
    at SummaryComponent.render (webpack://pcf_tools_652ac3f36e1e4bca82eb3c1dc44e6fad/./S4DLeadSummaryComponent2/component.tsx?:155:10)
    at Ua (https://orgb717200a.crm6.dynamics.com/uclient/scripts/vendor.js?v=1.4.4731-2208.5:9:71428)
    at Ha (https://orgb717200a.crm6.dynamics.com/uclient/scripts/vendor.js?v=1.4.4731-2208.5:9:71227)
    at Ul (https://orgb717200a.crm6.dynamics.com/uclient/scripts/vendor.js?v=1.4.4731-2208.5:9:112369)
    at Eu (https://orgb717200a.crm6.dynamics.com/uclient/scripts/vendor.js?v=1.4.4731-2208.5:9:98325)
    at ku (https://orgb717200a.crm6.dynamics.com/uclient/scripts/vendor.js?v=1.4.4731-2208.5:9:98253)
    at xu (https://orgb717200a.crm6.dynamics.com/uclient/scripts/vendor.js?v=1.4.4731-2208.5:9:98116)

Activity Id: c1dd3205-e99a-426e-a7de-2db4b03ea82d
*/

// CAUSE: 
//Using Xrm.WebApi.online.retrieveMultipleRecords
 Xrm.WebApi.online.retrieveMultipleRecords("environmentvariabledefinition", "?$filter=(schemaname eq 'dev_sitecoreConfig')");

//SOLUTION: Remove online
 Xrm.WebApi.retrieveMultipleRecords("environmentvariabledefinition", "?$filter=(schemaname eq 'dev_sitecoreConfig')");
```

```
https://orgb717200a.crm6.dynamics.com/api/data/v9.2/environmentvariabledefinitions?$filter=(schemaname eq 'dev_sitecoreConfig')
```

```typescript
// TESTING NEW FILTERS
        Xrm.WebApi.retrieveMultipleRecords("environmentvariabledefinition", "?$expand=environmentvariabledefinition_environmentvariablevalue($select=value)&$filter=(schemaname eq 'dev_sitecoreConfig') and (environmentvariabledefinition_environmentvariablevalue/any(o1:(o1/environmentvariablevalueid ne null)))").then(
            function success(result: any) {
                if (result.ok) {
                    result.json().then((response: any) => {
                        return response;
                    })
                }
            },
            function (error: any) {
                return error.message;
            }
        );
```

```c#
/* ERROR:
{
    "@odata.context": "https://orgb717200a.crm6.dynamics.com/api/data/v9.0/$metadata#Microsoft.Dynamics.CRM.dev_GetDataResponse",
    "data": "Exception has been thrown by the target of an invocation."
}
*/

//REASON: 
//For some reason the refrence was changed from dev_dev_sitecorexdbcontact_leadid_lead.Id to dev_leadid.Id

```

```typescript
/* Uncaught (in promise) TypeError: Cannot read properties of undefined (reading 'map')
    at eval (index.ts:95:26) */

//REASON: 
//For some reason the refrence was changed from visits to VisitsList
```

### Day 3:

```typescript
/*
Error Details: TypeError: Cannot read properties of undefined (reading 'SitecoreUrl')
    at SitecoreLeadSummaryComponent2.updateView (webpack://pcf_tools_652ac3f36e1e4bca82eb3c1dc44e6fad/./SitecoreLeadSummaryComponent2/index.ts?:41:37)
    at https://orgb717200a.crm6.dynamics.com/uclient/scripts/9.js?v=1.4.4731-2208.5:1:273107
    at t._measureLifecycleMethod (https://orgb717200a.crm6.dynamics.com/uclient/scripts/9.js?v=1.4.4731-2208.5:1:249801)
    at t._renderMainControlComponent (https://orgb717200a.crm6.dynamics.com/uclient/scripts/9.js?v=1.4.4731-2208.5:1:273047)
    at t.renderWrappedMainElement (https://orgb717200a.crm6.dynamics.com/uclient/scripts/9.js?v=1.4.4731-2208.5:1:275731)
    at t._renderInternal (https://orgb717200a.crm6.dynamics.com/uclient/scripts/9.js?v=1.4.4731-2208.5:1:279538)
    at t.render (https://orgb717200a.crm6.dynamics.com/uclient/scripts/9.js?v=1.4.4731-2208.5:1:280656)
    at Ua (https://orgb717200a.crm6.dynamics.com/uclient/scripts/vendor.js?v=1.4.4731-2208.5:9:71428)
    at Ha (https://orgb717200a.crm6.dynamics.com/uclient/scripts/vendor.js?v=1.4.4731-2208.5:9:71227)
    at Ul (https://orgb717200a.crm6.dynamics.com/uclient/scripts/vendor.js?v=1.4.4731-2208.5:9:112369)

Activity Id: 79ae908d-5c83-4100-bfc3-94f607640ad6 */

// CAUSE: Object attributes didnt match the attributes of the incomming JSON

// I was dumb and also removed the call of the function that retrieves the data :(
```

- This is being a pain :(
