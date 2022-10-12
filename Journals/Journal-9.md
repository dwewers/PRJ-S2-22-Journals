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
