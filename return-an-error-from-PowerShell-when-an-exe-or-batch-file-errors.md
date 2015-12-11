Often when using a build server, such as TeamCity or Jenkins, you may wish to run an exe or batch file through PowerShell.

The problem is that when these external files fail, the PowerShell script still reports a success. This in turn causes your build server to report a success, giving you a false positive.

To prevent this, you can use `$LastExitCode` to see if the last command exited with an error. If the last command did throw an error, you can then throw from inside your PowerShell script, which will in turn cause your build server to report the error.

[code language="powershell" highlight="22, 23,24,25"]

	param(
	    [string]$roundHouseLocation,
	    [string]$db_server,
	    [string]$db_name,
	    [string]$db_scripts_directory
	)
	
	$output_directory = $db_scripts_directory + "/output"
	$roundhouse_script_location = $roundHouseLocation + 'rh.exe'
	Write-Host "Starting Database Migrations with $roundhouse_script_location"
	
	& $roundhouse_script_location `
	    -s $db_server `
	    -d $db_name `
	    -f $db_scripts_directory `
	    -o $output_directory `
	    --commandtimeout=300 `
	    --silent `
	
	Write-Host "The exit code from roundhouse was " $LastExitCode
	
	$success = 0
	if ($LastExitCode -ne $success) {
	    throw "Error when running roundhouse scripts"
	}
	

[/code]