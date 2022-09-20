# Journal 7 - 14/09/2022

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
