## Introduction

Both automated testing and continues integrations servers are essential to establishing a rapid software development pipeline. When developing in .net it is very easy to configure a continues integration server to execute `Web.config` transformations. These transformations allow you to quickly and consistently move code between development environments. However, it can be a bit tricky to set these same transformations up for our integration tests.


## Create your new App.config files

First we need to creature our configuration files.

Go into the root folder of your test project where you will find the `App.config` file. Create a copy of it to mirror each for your web.cong files.

For example, we have use a `Web.Debug.config`, a `Web.QA.config`, and a `Web.Release.config` so we will create three files, `App.Debug.config`, `App.QA.config`, and `App.Release.config`.

Don't worry about the content of these files for now. We will address that later on.


## Include your new App.config files in your .csproj

Now we need to prepare our test project.

Using a text editor, open your .csproj file that contains your tests, in our case it is ` Tests.Integration.csproj`.

Below the last `PropertyGroup` add the following lines of code:

[code language="xml"]

    <PropertyGroup>
      <ProjectConfigFileName>App.config</ProjectConfigFileName>
    </PropertyGroup>

[/code]


Now find the `ItemGroup` that relates to your App.config files. It should look something like this:

[code language="xml"]

      <ItemGroup>
        <None Include="App.config">
          <SubType>Designer</SubType>
        </None>
        <None Include="packages.config" />
      </ItemGroup>

[/code]


## Enable transformations

After the last `Import` tag, add the following:

[code language="xml"]

      <Import Project="$(SolutionDir)\.nuget\NuGet.targets" Condition="Exists('$(SolutionDir)\.nuget\NuGet.targets')" />
      <Import Project="$(MSBuildExtensionsPath)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets" />
      <Import Project="$(MSBuildExtensionsPath)\Microsoft\VisualStudio\v10.0\Web\Microsoft.Web.Publishing.targets" />

[/code]


Directly above the closing `Project` tag, add the following:

[code language="xml"]

    <Target Name="AfterBuild">
      <TransformXml Source="@(AppConfigWithTargetPath)" Transform="$(ProjectConfigTransformFileName)" Destination="@(AppConfigWithTargetPath->'$(OutDir)%(TargetPath)')" />
    </Target>

[/code]

## Setup your config file transformations

Now create your transformations in the relevant `App.config` files. For example, our `App.Debug.config` looks similar to:

[code language="xml"]

    <?xml version="1.0" encoding="utf-8"?>
    <configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">

          <connectionStrings>
            <add name="DatabaseConnectName"
                 connectionString="DatabaseConnectionString"
                 xdt:Transform="SetAttributes"
                 xdt:Locator="Match(name)"/>
          </connectionStrings>

    </configuration>

[/code]
