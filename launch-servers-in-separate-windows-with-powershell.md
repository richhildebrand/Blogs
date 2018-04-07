## Introduction

Recently I have been spending my free time building an Angular5, node.js web application. While I have enjoyed working with these tools, spinning up the node.js server and the angular cli server in seperate windows usually involves a few mintues of copy pasting. I wanted a quicker, way to get started and as with most things, automation is the easy answer!

My goal was to create a powershell script `runlocally.ps1` that would spin up a powershell window for the angular cli server, and a seperate cmd window for the node.js server. I wanted to use the two different tools to make them easier to identify when debugging and developing.

My folder structure inclinde a base folder with items such as my `.gitignore` and automated deploy scripts `stagingDeploy.ps1` and `productionDeploy.ps1`. I then have two seperate folders `client` for the angular5 code, and `server` for the node.js code.

## The Code

Here are the two commands that open new windows and run our servers!

[code language="powershell"]

    Start-Process powershell -ArgumentList "-noexit", "npm run start --prefix client"

    Start-Process cmd -ArgumentList "/k", "cd server & SET configEnv=develop && node --inspect app.js"
    	

[/code]

