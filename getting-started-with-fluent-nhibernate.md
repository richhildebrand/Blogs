#Getting started with Fluent NHibernate

##Installation

Install Fluent NHibernate from NuGet

![](../images/Adding-Fluent-Nhibernate-from-nuget.PNG)

##Database Context

Now add a new class to use as your NHibernate database context.

	namespace Demo.Infrastructure.Nhibernate
	{
	    public static class DatabaseContext
	    {

	    }
	}

###Connection String

Add you connection string to your app.config or web.config.

	<?xml version="1.0" encoding="utf-8"?>
	<configuration>
	...
	  <connectionStrings>
	    <add name="DatabaseConnectionString" connectionString="server=localhost\sqlexpress2012;database=Demo;Integrated Security=true;" providerName="System.Data.SqlClient" />
	  </connectionStrings>
	...
	</configuration>




Inside this class add a method to retrieve your connection string.

    private static MsSqlConfiguration GetSqlConfiguration(string databaseConnectionStringKey)
    {
        return MsSqlConfiguration.MsSql2012
            .ConnectionString(c => c.FromConnectionStringWithKey(databaseConnectionStringKey));
    }

###Configuration

Build your configuration.

    private static Configuration BuildConfiguration()
    {
        return Fluently.Configure()
            .Database(GetSqlConfiguration("DatabaseConnectionString"))
            .Mappings(m => m.AutoMappings
                    .Add(AutoMap.AssemblyOf<Person>())
            )
            .BuildConfiguration();
    }

###Session Factory

Create your session factory.

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

###The Complete Database Context

The completed database context should now look as follows.

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

##Your first Class and Table

Add a new class to use with Fluent NHibernate. The virtual key word is required by NHibernate on properties.

	namespace Demo.Core.Models
	{
	    public class Person
	    {
	        public virtual int Id { get; set; }
	        public virtual string FirstName { get; set; }
	        public virtual string LastName { get; set; }
	    }
	}

User your migration script manager of choice to create the corresponding table in your database.

	CREATE TABLE Person
	(
		Id INT IDENTITY(1,1) PRIMARY KEY,
		LastName VARCHAR(255),
		FirstName VARCHAR(255)
	)


##Using the Database Context

The completed database context is now ready to be used. Lets add a repository to save a new Person.

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
	                session.Save(person);
	            }
	        }
	    }
	}

And some code to call the repository.

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

