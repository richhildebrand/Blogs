It is a common practice among .net developers to leave their nuget packages out of source control.

These missing packages can then cause build errors when deploying from a build server, such as TeamCity or Jenkins.

To ensure that msbuild will pull down your nuget packages you must add the following configuration to your csproj files.

[code language="xml"]

	<Project>
	...

		<Import Project="$(SolutionDir)\.nuget\NuGet.targets" Condition="Exists('$(SolutionDir)\.nuget\NuGet.targets')" />
		<Target Name="EnsureNuGetPackageBuildImports" BeforeTargets="PrepareForBuild">
		    <PropertyGroup>
		      <ErrorText>This project references NuGet package(s) that are missing on this computer. Enable NuGet Package Restore to download them.  For more information, see http://go.microsoft.com/fwlink/?LinkID=322105. The missing file is {0}.</ErrorText>
		    </PropertyGroup>
	    	<Error Condition="!Exists('$(SolutionDir)\.nuget\NuGet.targets')" Text="$([System.String]::Format('$(ErrorText)', '$(SolutionDir)\.nuget\NuGet.targets'))" />
		</Target>

	...
	</Project>

[/code]