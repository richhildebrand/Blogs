## Introduction

This article will show you how to configure a primary key convention for automappings in Fluent NHibernate.

If you would like to setup automapping, [this previous article](https://richardhildebrand.wordpress.com/2015/05/23/advanced-fluent-nhibernate-automapping-configuration/) may be of help. 

## Add a class and a table

First lets add a class with the primary key convention class name _Id.

[code language="csharp"]

	using System;
	using Demo.Core.Interfaces;
	
	namespace Demo.Core.Models
	{
	    public class Pet : IPersistentModel
	    {
	        public virtual Guid Pet_Id { get; set; }
	        public virtual string Name { get; set; }
	    }
	}
		
[/code]

Then, in our database, lets add a table matching this primary key convention.

[code language="sql"]

	CREATE TABLE Pets
	(
		Pet_Id UNIQUEIDENTIFIER PRIMARY KEY,
		PetName VARCHAR(255),
	);
		
[/code]

## The DataContext

You should already have an automapping configuration file which may look something like this.

[code language="csharp"]

	using Demo.Core.Models;
	using Demo.Infrastructure.Nhibernate.Configurations;
	using Demo.Infrastructure.Nhibernate.Mappings.Overrides;
	using FluentNHibernate.Automapping;
	using FluentNHibernate.Cfg;
	using FluentNHibernate.Cfg.Db;
	using NHibernate;
	using NHibernate.Cfg;

	namespace Demo.Infrastructure.Nhibernate
	{
	    public static class DatabaseContext
	    {
	        private static ISessionFactory _sessionFactory;
	        public static ISessionFactory SessionFactory
	        {
	            get { return _sessionFactory; }
	        }
	
	        static DatabaseContext()
	        {
	            InitializeSessionFactory();
	        }
	
	        private static void InitializeSessionFactory()
	        {
	            Configuration configuration = BuildConfiguration();
	            _sessionFactory = configuration.BuildSessionFactory();
	        }
	
	        private static Configuration BuildConfiguration()
	        {
	            return Fluently.Configure()
	                .Database(GetSqlConfiguration("DatabaseConnectionString"))
	                .Mappings(m => m.AutoMappings
	                        .Add(AutoMap.AssemblyOf<Pet>(new AutomappingConfiguration())
	                        .UseOverridesFromAssemblyOf<PetMap>())
	                )
	                .BuildConfiguration();
	        }
	
	        public static MsSqlConfiguration GetSqlConfiguration(string databaseConnectionStringKey)
	        {
	            return MsSqlConfiguration.MsSql2012
	                .ConnectionString(c => c.FromConnectionStringWithKey(databaseConnectionStringKey));
	        }
	    }
	}

[/code]

Possibly with an automapping Configuration similar to the following.

[code language="csharp"]

	using System;
	using Demo.Core.Interfaces;
	using FluentNHibernate.Automapping;
	
	namespace Demo.Infrastructure.Nhibernate.Configurations
	{
	    public class AutomappingConfiguration : DefaultAutomappingConfiguration
	    {
	        public override bool ShouldMap(Type type)
	        {
	            var persitentType = typeof(IPersistentModel);
	            return persitentType.IsAssignableFrom(type);
	        }
	    }
	}

[/code]

## Configuring the primary key convention

Lets configure our primary key convention to set the Id Column. Since above we configured Fluent Nhibernate to only automap classes implementing the interface IPersistentModel we will only configure the Id on classes that implement IPersistentModel.

[code language="csharp"]
			
	using System;
	using Demo.Core.Interfaces;
	using FluentNHibernate.Conventions;
	using FluentNHibernate.Conventions.Instances;
	
	namespace Demo.Infrastructure.Nhibernate.Configurations
	{
	    public class PrimaryKeyConvention : IIdConvention
	    {
	        public void Apply(IIdentityInstance instance)
	        {
	            Type type = instance.EntityType;
	            var persitentType = typeof(IPersistentModel);
	            var isMappableModel = persitentType.IsAssignableFrom(type);
	
	            if (isMappableModel)
	            {
	                instance.Column(type.Name + "_Id");
	                instance.GeneratedBy.Assigned();
	            }
	        }
	    }
	}

[/code]

Now lets add the primary key configuration to our automapping configuration.

[code language="csharp" highlight="8"]

    private static Configuration BuildConfiguration()
    {
        return Fluently.Configure()
            .Database(GetSqlConfiguration("DatabaseConnectionString"))
            .Mappings(m => m.AutoMappings
                    .Add(AutoMap.AssemblyOf<Pet>(new AutomappingConfiguration())
                    .UseOverridesFromAssemblyOf<PetMap>()
                    .Conventions.Add(new PrimaryKeyConvention())
                )
            )
            .BuildConfiguration();
    }

[/code]

## Summary

That is all there is to it. Though it is always good to test and confirm everything is working.

[code language="csharp"]

	using System;
	using Demo.Core.Models;
	using Demo.Infrastructure.Nhibernate;
	using Demo.Infrastructure.Nhibernate.Repositories;
	using NUnit.Framework;
	using Should;
	
	namespace Demo.IntegrationTests.PetRepositoryTests
	{
	    [TestFixture]
	    public class SavePetShould
	    {
	        [Test]
	        public void Save()
	        {
	            var petId = Guid.NewGuid();
	            var petRepository = new PetRepository(DatabaseContext.SessionFactory);
	
	            var pet = new Pet
	            {
	                Pet_Id = petId,
	                Name = "Dawg Pound"
	            };
	
	            petRepository.Save(pet);
	
	            pet.Pet_Id.ShouldEqual(petId);
	        }
	    }
	}


[/code]