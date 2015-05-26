## Introduction

This article is intended to help setup Fluent NHibernate for the first time.

## Installation

Install Fluent NHibernate from NuGet

![](../images/Adding-Fluent-Nhibernate-from-nuget.PNG)

## Database Context

Now add a new class to use as your NHibernate database context.

[code language="csharp"]

	namespace Demo.Infrastructure.Nhibernate
	{
	    public static class DatabaseContext
	    {

	    }
	}
	
[/code]
	

### Connection String

Add your connection string to your app.config or web.config.

[code language="xml"]

	<?xml version="1.0" encoding="utf-8"?>
	<configuration>
	...
	  <connectionStrings>
	    <add name="DatabaseConnectionString" connectionString="server=localhost\sqlexpress2012;database=Demo;Integrated Security=true;" providerName="System.Data.SqlClient" />
	  </connectionStrings>
	...
	</configuration>
	
[/code]

Inside this class add a method to retrieve your connection string.

[code language="csharp"]

    private static MsSqlConfiguration GetSqlConfiguration(string databaseConnectionStringKey)
    {
        return MsSqlConfiguration.MsSql2012
            .ConnectionString(c => c.FromConnectionStringWithKey(databaseConnectionStringKey));
    }
	
[/code]

### Configuration

Build your configuration.

If you are setting up Fluent Nhibernate on a existing project you may need to add a [DefaultAutomappingConfiguration](../advanced-fluent-nhibernate-automapping-configuration.md) to avoid a FluentNHibernate.Visitors.ValidationException.

[code language="csharp"]

    private static Configuration BuildConfiguration()
    {
        return Fluently.Configure()
            .Database(GetSqlConfiguration("DatabaseConnectionString"))
            .Mappings(m => m.AutoMappings
                    .Add(AutoMap.AssemblyOf<Person>())
            )
            .BuildConfiguration();
    }
	
[/code]

### Session Factory

Create your session factory.

[code language="csharp"]

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
	
[/code]

### The Complete Database Context

The completed database context should now look as follows.

[code language="csharp"]

	using Demo.Core.Models;
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
	                        .Add(AutoMap.AssemblyOf<Person>())
	                )
	                .BuildConfiguration();
	        }
	
	        private static MsSqlConfiguration GetSqlConfiguration(string databaseConnectionStringKey)
	        {
	            return MsSqlConfiguration.MsSql2012
	                .ConnectionString(c => c.FromConnectionStringWithKey(databaseConnectionStringKey));
	        }
	    }
	}
	
[/code]

## Your first Class and Table

In a new empty project, add a new class to use with Fluent NHibernate. The virtual key word is required by NHibernate on properties.

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

User your migration script manager of choice to create the corresponding table in your database.

[code language="sql"]

	CREATE TABLE Person
	(
		Id INT IDENTITY(1,1) PRIMARY KEY,
		LastName VARCHAR(255),
		FirstName VARCHAR(255)
	);
	
[/code]


## Using the Database Context

The completed database context is now ready to be used. Lets add a repository to save a new Person.

[code language="csharp"]

	using Demo.Core.Models;
	using NHibernate;
	
	namespace Demo.Infrastructure.Nhibernate.Repositories
	{
	    public class PersonRepository
	    {
	        private readonly ISessionFactory _sessionFactory;
	
	        public PersonRepository(ISessionFactory sessionFactory)
	        {
	            _sessionFactory = sessionFactory;
	        }
	
	        public void Save(Person person)
	        {
	            using (ISession session = _sessionFactory.OpenSession())
	            {
	                using (var transaction = session.BeginTransaction())
	                {
	                    session.Save(person);
	                    transaction.Commit();
	                }
	            }
	        }
	    }
	}
	
[/code]

And some code to call the repository.

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

## Closing

This article is intended to get you up and running as quickly as possible. 

If you wish to add classes that you do not want to automap with Fluent Nhibernate to the new project containing the Person class, you will need to add a [DefaultAutomappingConfiguration](../advanced-fluent-nhibernate-automapping-configuration.md) to avoid a FluentNHibernate.Visitors.ValidationException.


