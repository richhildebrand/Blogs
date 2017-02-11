## Introduction

Often razor views are slow to load the first time you navigate to a page. This is because they are compiled the first time the page is loaded. MVC does allow you to precompile these views by setting `<MvcBuildViews>true</MvcBuildViews>` in your `.csproj` file. Unfortunately, setting this flag will not aleviate the slow initial page loads from our razor views and significantly increase the amount of time your project will take to build.

## The Solution

Start by installing the NuGet packages` RazorGenerator.Mvc` and `RazorGenerator.MsBuild` into your MVC project.

Now add the following line to your `.csproj` file.

	//Find this line
	<Import Project="$(SolutionDir)\packages\RazorGenerator.MsBuild.2.4.7\build\RazorGenerator.MsBuild.targets"/>
    
    //Add this line directly below the above line
    <Import Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets" Condition="false" />

Finally, directly before the closing `</Project>` tag, add the following code.

    <Target Name="BeforeBuild">
    	<ItemGroup>
      		<Content Remove="Views\**\*.cshtml" />
      		<None Include="Views\**\*.cshtml" />
    	</ItemGroup>
    </Target>