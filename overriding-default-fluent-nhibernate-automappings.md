## Introduction

This article will show you how to override the default Fluent NHibernate automapping settings for a particular model.

If you would like to setup automapping, [this previous article](../advanced-fluent-nhibernate-automapping-configuration.md) may be of help. 

## The Class and the mismatched Table

Our class.

[code language="csharp"]

	using System;
	using Demo.Core.Interfaces;
	
	namespace Demo.Core.Models
	{
	    public class Pet : IPersistentModel
	    {
	        public virtual Guid Id { get; set; }
	        public virtual string Name { get; set; }
	    }
	}
		
[/code]

Our mismatched table.

[code language="sql"]

	CREATE TABLE Pets
	(
		Id UNIQUEIDENTIFIER PRIMARY KEY,
		PetName VARCHAR(255),
	);
		
[/code]

## The mapping override class
	
Lets create our new mapping class.

[code language="csharp"]

	namespace Demo.Infrastructure.Nhibernate.Mappings.Overrides
	{
	    public class PetMap : IAutoMappingOverride<Pet>
	    {
	        public void Override(AutoMapping<Pet> mapping)
	        {
	        }
	    }
	}
		
[/code]


Adjust the the table name.

[code language="csharp"]

	mapping.Table("Pets");
		
[/code]

Notify Fluent NHibernate that the primary key will be set from c# instead of by the database.

[code language="csharp"]

	mapping.Id(x => x.Id).GeneratedBy.Assigned();
		
[/code]

Correct the column name.

[code language="csharp"]

	mapping.Map(x => x.Name).Column("PetName");
		
[/code]


The completed mapping override class will looks as follows.

[code language="csharp"]

	using Demo.Core.Models;
	using FluentNHibernate.Automapping;
	using FluentNHibernate.Automapping.Alterations;
	
	namespace Demo.Infrastructure.Nhibernate.Mappings.Overrides
	{
	    public class PetMap : IAutoMappingOverride<Pet>
	    {
	        public void Override(AutoMapping<Pet> mapping)
	        {
	            mapping.Table("Pets");
				mapping.Id(x => x.Id).GeneratedBy.Assigned();
	            mapping.Map(x => x.Name).Column("PetName");
	        }
	    }
	}	
		
[/code]

## Adding the overload to the Fluent NHibernate Configuration

Using UseOverridesFromAssemblyOf will automatically find and add any further overrides placed in the same project.

[code language="csharp" highlight="7"]

    private static Configuration BuildConfiguration()
    {
        return Fluently.Configure()
            .Database(GetSqlConfiguration("DatabaseConnectionString"))
            .Mappings(m => m.AutoMappings
                    .Add(AutoMap.AssemblyOf<Pet>(new AutomappingConfiguration())
                    	.UseOverridesFromAssemblyOf<PetMap>()
					)
            )
            .BuildConfiguration();
    }
		
[/code]

## The repository


[code language="csharp"]

Lets add some code to save our pet.

	using Demo.Core.Models;
	using NHibernate;

	namespace Demo.Infrastructure.Nhibernate.Repositories
	{
	    public class PetRepository
	    {
	        private readonly ISessionFactory _sessionFactory;
	
	        public PetRepository(ISessionFactory sessionFactory)
	        {
	            _sessionFactory = sessionFactory;
	        }
	
        public void Save(Pet pet)
        {
            using (ISession session = _sessionFactory.OpenSession())
            {
                using (var transaction = session.BeginTransaction())
                {
                    session.Save(pet);
                    transaction.Commit();
                }
            }
	    }
	}
	
[/code]

## Saving

Finally, lets save a pet.


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
	                Id = petId,
	                Name = "Dawg Pound"
	            };
	
	            petRepository.Save(pet);
	
	            pet.Id.ShouldEqual(petId);
	        }
	    }
	}

[/code]