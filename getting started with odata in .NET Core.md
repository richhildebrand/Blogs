# Getting Started with OData

## Why OData?

Using OData will quickly provide us with filtering, sorting, and selecting options from any IQueryable. We will see a few examples of this once we setup our project (about 5 minutes).

## Setup our Project

Create a new .net core project

![New Project](https://richardhildebrand.files.wordpress.com/2019/06/newcoreproject.jpg)
![Pick Angular2](https://richardhildebrand.files.wordpress.com/2019/06/newangular2project.jpg)
![Run Project ](https://richardhildebrand.files.wordpress.com/2019/06/runproject.jpg)

Import our DB by going to **_Tools –&gt; NuGet Package Manager –&gt; Package Manager Console_** and running `Scaffold-DbContext "Server=localhost;Database=YOUR_DATABASE_NAME;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -DataAnnotations`

## Add OData from NuGet

Add OData from NuGet: `Microsoft.AspNetCore.OData`

![New Project](https://richardhildebrand.files.wordpress.com/2019/06/addodatafromnuget.jpg)

## Add our OData Controller

Create an ODataController `YourTableController.cs`

[code language="csharp"]
    public class YourTableController : ODataController
    {
        private readonly DBContext _dbContext;
    
        public YourTableController()
        {
            _dbContext = new DBContext();
        }
    
        [EnableQuery]
        public IActionResult Get()
        {
            return Ok(_proDbContext.YourTable);
        }
    }
[/code]

## Configure OData in Startup.cs

Build our Entity Data Model by adding the `GetEdmModel()` method to the **_Startup_** class.

[code language="csharp"]
    public class Startup
    {
        private static IEdmModel GetEdmModel()
        {
            ODataConventionModelBuilder builder = new ODataConventionModelBuilder();

            # If your keys have been defined in your Database Context instead of as class attributes, you will need to redeclare them here.
            builder.EntitySet<YourTable>("YourTable").EntityType.HasKey(x => new { x.YourTableKey });
            return builder.GetEdmModel();
        }
    
        //...
    }
[/code]

Register our OData service by calling `services.AddOData();` in **_ConfigureServices_**.

[code language="csharp"]
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddOData();
    
        //...
    }
[/code]

Add OData to our routes with `routes.Select().Expand().Filter().OrderBy().MaxTop(1000).Count();` and `routes.MapODataServiceRoute("odata", "odata", GetEdmModel());`

[code language="csharp"]
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        //...
    
        app.UseMvc(routes =>
        {
            routes.Select().Expand().Filter().OrderBy().MaxTop(1000).Count();
            routes.MapODataServiceRoute("odata", "odata", GetEdmModel());
        });
    
        //...
    }
[/code]

## Test OData

Now we can build dynamic queries without adding further backend code. `https://localhost:PORTNUMBER/odata/YourTable?$filter=YourTable/YourColumn eq 2018&amp;$select=YourColumn&amp;$count=true&amp;$orderby=YourColumn desc`