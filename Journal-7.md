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

Below is the full execution method:

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

And Here is the validation:

```c#
private static string ValidateVisits(List<dev_sitecorevisit> visits, dev_sitecorexdbcontact contact)
        {
            if (visits.Count > 0)
            {
                return JsonConvert.SerializeObject(
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
            return JsonConvert.SerializeObject(
                new
                {
                    status = "Failed",
                    code = 01,
                    error = "There are no visits related to this entity"
                });
        }

        /// <summary>Gets the count of visits from the list of visits</summary>
        /// <param name="visits">The visits.</param>
        /// <returns>Count if there are any visits, else zero</returns>
        private static int SitecoreVisitCount(IReadOnlyCollection<dev_sitecorevisit> visits)
        {
            return visits.Any() ? visits.Count() : 0;
        }

        /// <summary>Gets the average visit time from the list of visits</summary>
        /// <param name="visits">The visits.</param>
        /// <returns>Average if there are any visits, else zero</returns>
        private static double SitecoreVisitAverageDuration(IReadOnlyCollection<dev_sitecorevisit> visits)
        {
            return visits.Any() ? Math.Round(visits.Average(x => double.Parse(x.dev_duration)), 2) : 0;
        }

        /// <summary>Gets the sum of page views from the list of visits</summary>
        /// <param name="visits">The visits.</param>
        /// <returns>Sum if there are any visits, else zero</returns>
        private static int? SitecorePageViews(IReadOnlyCollection<dev_sitecorevisit> visits)
        {
            return visits.Any() ? visits.Sum(x => x.dev_pageviews) : 0;
        }

        /// <summary>Gets the sum of visits values from the list of visits</summary>
        /// <param name="visits">The visits.</param>
        /// <returns>Sum if there are any visits, else zero</returns>
        private static double SitecoreVisitValue(IReadOnlyCollection<dev_sitecorevisit> visits)
        {
            return visits.Any() ? Math.Round((double)visits.Sum(x => x.dev_value), 2) : 0;
        }


        /// <summary>Validates the string to see if the user has designated a Lead Id.</summary>
        /// <param name="leadid">The leadid.</param>
        /// <param name="context">The context.</param>
        /// <returns>True if there is a value, false if empty or null</returns>
        private static bool ValidateString(string leadid, IPluginExecutionContext context)
        {
            if (!string.IsNullOrWhiteSpace(leadid)) return true;

            context.OutputParameters["data"] = new
            {
                status = "Failed",
                error = "Lead Id is required to perform this action."
            };
            return false;
        }
```
