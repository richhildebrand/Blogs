## Introduction

Recently I was having trouble getting the primary key convention in NHibernate to work. Here is the trap that I fell into.

## Add a class and a table

I created the a table.

[code language="sql"]

	CREATE TABLE House (
		House_Id INT IDENTITY(1,1) PRIMARY KEY,
		Address NVARCHAR(255),
		City NVARCHAR(255),
		State NVARCHAR(255),
		ZipCode NVARCHAR(255),
	);
		
[/code]

And a corresponding class.

[code language="csharp"]

using Demo.Core.Interfaces;

	namespace Demo.Core.Models
	{
	    public class House : IPersistentModel
	    {
	        public virtual int House_Id { get; set; }
	        public virtual string Address { get; set; }
	        public virtual string City { get; set; }
	        public virtual string State { get; set; }
	        public virtual string ZipCode { get; set; }
	    }
	}
		
[/code]

## The Primary Key convention

To allow for the primary key House_Id I used the following primary key convention.

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
	                instance.GeneratedBy.Native();
	            }
	        }
	    }
	}

[/code]

## My mistake

Unfortunately this configuration gave me the following error ```{"The entity 'House' doesn't have an Id mapped. Use the Id method to map your identity property. For example: Id(x => x.Id)."}```.

This was because I had set my class Id property as ```public virtual int House_Id { get; set; }``` instead of correctly using ```public virtual int Id { get; set; }```.

For the Primary Key convention to be triggered the class must contain an Id property.

## Using another property name for a class Id.

Using ```Id``` was an acceptable solution for me. However, if I did not want to change my property name to ```Id```, I could have written an automapping override to continue using ```House_Id```.

To create the automapping override we would add the following code.

[code language="csharp"]
			
	using Demo.Core.Models;
	using FluentNHibernate.Automapping;
	using FluentNHibernate.Automapping.Alterations;
	
	namespace Demo.Infrastructure.Nhibernate.Mappings.Overrides
	{
	    public class HouseMap : IAutoMappingOverride<House>
	    {
	        public void Override(AutoMapping<House> mapping)
	        {
	            mapping.Table("House");
	            mapping.Id(x => x.House_Id);
	        }
	    }
	}

[/code]

Then to apply the automapping override we need to add it to the automapping configuration.

[code language="csharp" highlight="8"]

        private static Configuration BuildConfiguration()
        {
            return Fluently.Configure()
                .Database(GetSqlConfiguration("DatabaseConnectionString"))
                .Mappings(m => m.AutoMappings
                        .Add(AutoMap.AssemblyOf<House>(new AutomappingConfiguration())
                        .UseOverridesFromAssemblyOf<HouseMap>()
                        .Conventions.Add(new PrimaryKeyConvention())
                    )
                )
                .BuildConfiguration();
        }
	

[/code]

# Proof

[code language="csharp"]

Of course it is always good to confirm the new configuration works.

	using Demo.Core.Models;
	using Demo.Infrastructure.Nhibernate;
	using Demo.Infrastructure.Nhibernate.Repositories;
	using Demo.IntegrationTests.TestHelpers;
	using NUnit.Framework;
	using Should;
	
	namespace Demo.IntegrationTests.RepositoryTests
	{
	    [TestFixture]
	    public class SaveShould
	    {
	        private Repository _repository;
	
	        [SetUp]
	        public void SetUp()
	        {
	            _repository = new Repository(DatabaseContext.SessionFactory);
	        }
	
	        [Test]
	        public void SaveHouses()
	        {
	            var house = ModelCreator.CreateHouse(ModelCreator.IdNotSet);
	            _repository.Save<House>(house);
	            house.House_Id.ShouldNotEqual(ModelCreator.IdNotSet);
	        }
	    }
	}
	
[/code]