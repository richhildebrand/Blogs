## Introduction

This article assumes that Fluent NHibernate is already installed and running. If you are trying to set Fluent NHibernate up for the first time, this article may be of help [Getting started with Fluent NHibernate](../getting-started-with-fluent-nhibernate.md).

## Example Automapping configuration

To tell Fluent NHibernate to scan all classes in an assembly, we start by creating a class.

[code language="csharp"]

	namespace Demo.Core.Models
	{
	    public class Person
	    {
	        public virtual int Id { get; set; }
	        public virtual string FirstName { get; set; }
	        public virtual string LastName { get; set; }
	    }
	}
	

[/code]

We then tell Fluent NHibernate to scan the assembly (typically a project in your solution) for classes to map.

[code language="csharp"]

	Fluently.Configure()
	    .Mappings(m => m.AutoMappings
	            .Add(AutoMap.AssemblyOf<Person>())
	    )
	
[/code]

## Adding other classes to your project

### The issue - FluentNHibernate.Visitors.ValidationException

Most of the time we will not have a project exclusively dedicated to Database Models.

However, if we now add another class to our core project.

[code language="csharp"]

	using Demo.Core.Models;
	
	namespace Demo.Core.Helper
	{
	    public class PersonHelper
	    {
	        private readonly Person _person;
	
	        public PersonHelper(Person person)
	        {
	            _person = person;
	        }
	
	        public string GetFullName()
	        {
	            return _person.FirstName + " " + _person.LastName;
	        }
	    }
	}
	
[/code]

We will begin to throw a FluentNHibernate.Visitors.ValidationException.

[code language="csharp"]

	FluentNHibernate.Visitors.ValidationException : The entity 'PersonHelper' doesn't have an Id mapped. Use the Id method to map your identity property. For example: Id(x => x.Id).
	
[/code]

### The solution - DefaultAutomappingConfiguration

To resolve this exception we need to tell Fluent Nhibernate exactly which classes it should automap.

To do this, lets add an interface.

[code language="csharp"]

	namespace Demo.Core.Interfaces
	{
	    public interface IPersistentModel
	    {
	    }
	}
	
[/code]

And include it on our model.

[code language="csharp"]

	using Demo.Core.Interfaces;

	namespace Demo.Core.Models
	{
	    public class Person : IPersistentModel
	    {
	        public virtual int Id { get; set; }
	        public virtual string FirstName { get; set; }
	        public virtual string LastName { get; set; }
	    }
	}
	
[/code]

Now we can create DefaultAutomappingConfiguration and tell it to only map classes that implement our new interface.

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

Finally, this configuration must be added to our Fluent Configuration.

[code language="csharp" highlight="6"]

    private static Configuration BuildConfiguration()
    {
        return Fluently.Configure()
            .Database(GetSqlConfiguration("DatabaseConnectionString"))
            .Mappings(m => m.AutoMappings
                    .Add(AutoMap.AssemblyOf<Person>(new AutomappingConfiguration()))
            )
            .BuildConfiguration();
    }
	
[/code]

At this point our FluentNHibernate.Visitors.ValidationException is resolved and we can once again use our mapped models which implement our interface.

[code language="csharp"]

	using Demo.Core.Models;
	using Demo.Infrastructure.Nhibernate;
	using Demo.Infrastructure.Nhibernate.Repositories;
	using NUnit.Framework;
	using Should;
	
	namespace Demo.IntegrationTests.PersonRepositoryTests
	{
	    [TestFixture]
	    public class SavePersonShould
	    {
	        [Test]
	        public void Save()
	        {
	            var personRepository = new PersonRepository(DatabaseContext.SessionFactory);
	
	            const int idNotSet = 0;
	            var person = new Person
	            {
	                Id = idNotSet,
	                FirstName = "Bernie",
	                LastName = "Kosar",
	            };
	
	            personRepository.Save(person);
	
	            person.Id.ShouldNotEqual(idNotSet);
	        }
	    }
	} 
	
[/code]