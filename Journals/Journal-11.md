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
