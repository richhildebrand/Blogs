## How to access a network drive from the command line ##

When navigating the folder direction with the command line you would typically use the change directory command.

For example:

	cd windows

Will move you to the windows directory.

### The problem

Unfortunately mapped network drive paths start with `\\` with causes the command line to throw the error `CMD does not support UNC paths as current directories.`

### The solution

To work around this issue you can use `pushd` to map a temporary virtual drive.

	pushd \\my\shared\folder

### Cleanup

To remove the the temporary directory you would then use 
`popd` from the command line while inside your mapped virtual drive.

	U:\my\shared\folder> popd

 
