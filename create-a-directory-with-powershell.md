## Introduction

This article will show you two different ways to create a directory using PowerShell.

## Using Test-Path

The first method is to check for the directory and then to add the folder if it is missing. 

[code language="powershell"]

	param(
		[string]$PathAndFolderToCreate
	) 
	
	$isAlreadyCreated = Test-Path -Path $PathAndFolderToCreate
	if (!$isAlreadyCreated) {
		New-Item -ItemType directory -Path $PathAndFolderToCreate
	}
	
[/code]

## Using Force

A more succinct way of achieving the same goal is to use the `-Force` flag.

[code language="powershell"]

	param(
		[string]$PathAndFolderToCreate
	) 
	
	# Force will preserve the current folder contents
	New-Item -Force -ItemType directory -Path $PathAndFolderToCreate
	
[/code]

## An Example using our script

	C:\> .\OurScript.ps1 Path\FolderName