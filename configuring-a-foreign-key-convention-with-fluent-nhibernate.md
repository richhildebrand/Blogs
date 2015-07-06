## Introduction

This article will show you how to configure a foreign key convention for automappings in Fluent NHibernate.

If you would like to setup automapping, [this previous article](https://richardhildebrand.wordpress.com/2015/05/23/advanced-fluent-nhibernate-automapping-configuration/) may be of help. 

Additionally if setting up a primary key convention is of interest to you, [this article](https://richardhildebrand.wordpress.com/2015/06/18/configuring-a-primary-key-convention-with-fluent-nhibernate/) may be of use.

## Adding the tables and classes

In the database, lets create tables to represent people who live in a house. They will be connected by the foreign key FK_House_Id in the person table and House_Id in the House table.

[code language="sql"]

	CREATE TABLE Person (
		Person_Id INT IDENTITY(1,1) PRIMARY KEY,
		FirstName NVARCHAR(255),
		LastName NVARCHAR(255),
		FK_House_Id INT,
	);
	
	CREATE TABLE House (
		House_Id INT IDENTITY(1,1) PRIMARY KEY,
		Address NVARCHAR(255),
		City NVARCHAR(255),
		State NVARCHAR(255),
		ZipCode NVARCHAR(255),
	);
	
	ALTER TABLE Person
	ADD FOREIGN KEY (FK_House_Id)
	REFERENCES House(House_Id)
	

[/code]

Now lets add the House class.

[code language="csharp"]

	using Demo.Core.Interfaces;
	
	namespace Demo.Core.Models
	{
	    public class House : IPersistentModel
	    {
	        public int Id { get; set; }
	        public string Address { get; set; }
	        public string City { get; set; }
	        public string State { get; set; }
	        public string ZipCode { get; set; }
	    }
	}
	
	
[/code]

And the Person class, who will have an instance of the House class.

[code language="csharp"]
	
	using Demo.Core.Interfaces;
	
	namespace Demo.Core.Models
	{
	    public class Person : IPersistentModel
	    {
	        public virtual int Id { get; set; }
	        public virtual string FirstName { get; set; }
	        public virtual string LastName { get; set; }
	        public virtual House House { get; set; }
	    }
	}
	
		
[/code]

## The Foreign Key convention

Now we need to create the Foreign Key convention.

[code language="csharp"]

	using System;
	using FluentNHibernate;
	using FluentNHibernate.Conventions;
	
	namespace Demo.Infrastructure.Nhibernate.Configurations
	{
	    public class CustomForeignKeyConvention : ForeignKeyConvention
	    {
	        protected override string GetKeyName(Member property, Type type)
	        {
	            if (property == null)
	            {
	                return "";
	            }
	
	            return "FK_" + property.Name + "_Id";
	        }
	    }
	}
	

[/code]

And add the new convention to the automapping configuration.

[code language="csharp" highlight="8"]

    private static Configuration BuildConfiguration()
    {
        return Fluently.Configure()
            .Database(GetSqlConfiguration("DatabaseConnectionString"))
            .Mappings(m => m.AutoMappings
                    .Add(AutoMap.AssemblyOf<House>(new AutomappingConfiguration())
                    .Conventions.Add(new PrimaryKeyConvention())
                    .Conventions.Add(new CustomForeignKeyConvention())
                )
            )
            .BuildConfiguration();
    }
	

[/code]

## Proof

Now lets confirm that we can save our objects.

[code language="csharp"]
			
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
	            house.Id.ShouldNotEqual(ModelCreator.IdNotSet);
	        }
	
	        [Test]
	        public void SavePerson()
	        {
	            var house = ModelCreator.CreateHouse(ModelCreator.IdNotSet);
	            _repository.Save<House>(house);
	
	            var person = ModelCreator.CreatePerson(ModelCreator.IdNotSet, house.Id);
	            _repository.Save<Person>(person);
	
	            person.Id.ShouldNotEqual(ModelCreator.IdNotSet);
	            person.House.Id.ShouldEqual(house.Id);
	        }
	    }
	}
	

[/code]

And that we can pull back saved data.

[code language="csharp"]

	using Demo.Core.Models;
	using Demo.Infrastructure.Nhibernate;
	using Demo.Infrastructure.Nhibernate.Repositories;
	using Demo.IntegrationTests.TestHelpers;
	using NHibernate.Event;
	using NUnit.Framework;
	using Should;
	
	namespace Demo.IntegrationTests.RepositoryTests
	{
	    [TestFixture]
	    public class GetShould
	    {
	        private Repository _repository;
	
	        [SetUp]
	        public void SetUp()
	        {
	            _repository = new Repository(DatabaseContext.SessionFactory);
	        }
	
	        [Test]
	        public void GetPersonAndHouse()
	        {
	            var house = ModelCreator.CreateHouse(ModelCreator.IdNotSet);
	            _repository.Save<House>(house);
	
	            var person = ModelCreator.CreatePerson(ModelCreator.IdNotSet, house.Id);
	            _repository.Save<Person>(person);
	
	            var personFromDb = _repository.Get<Person>(person.Id);
	            personFromDb.Id.ShouldEqual(person.Id);
	            personFromDb.House.Id.ShouldEqual(house.Id);
	        }
	
	    }
	}
	

[/code]