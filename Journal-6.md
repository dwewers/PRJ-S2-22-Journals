# Journal 6 - 07/09/2022

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
