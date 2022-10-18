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
