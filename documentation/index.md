# Installing Packages

To install a seed package, just use the `install` command:

	seed install markdown
	
This will install both the package itself as well as any dependencies in a shared location on your computer (usually ~/.seeds/packages).  

# Requiring Packages

Seed extends the require() function defined for your module so that it can 
find seed packages as well as regular node modules.  This means to use seed
you need to make sure it is loaded before you use require.  Normally the
best way to do this is to run your JavaScript using seed.  For example,
to run some JavaScript you could use seed like so:

	seed ./path/to/my/javascript.js

This will preload seed then run your JavaScript with node.js as normal.  You 
can also invoke your code using seed as a pound-bash:

	#!/usr/bin/env seed
	
	var markdown = require('markdown');
	markdown.makeHtml('# Hello World!');

Once you are running with seed, you can load package code simply with require:

	require('markdown');
	
This will look for a module in your local package (if you have one) named 
'markdown'.  If it is not found, seed will look for a package named 
'markdown'.  If still not found, it will look for a node library by the same
name.  

Generally this means that code written for node will continue to work with
seed unchanged.

# Requiring Modules Within Packages

Often times you will only want to work with a package by requiring its 
public API.  You get this API by calling `require('packageName')`.  If you 
instead need to require an individual module within a package, you and get the
module by using namespacing.  For example:

	require('markdown:parser');
	
Would look for a module named 'parser' inside the 'markdown' package.  

# Creating Your Own Packages

Seed is useful if you need to hack out a single JavaScript file to run on the
command line.  But if you plan to build a larger app, you are probably going 
to want to split your code into multiple files.  You may even want to split
your code into multiple packages that you can share with others.

In both of these cases, the best way to manage your files is to create 
a package for yourself.

Building a package is very easy.  Just create a folder, add a package.json 
file and put your source code into a lib folder like so:

	hello-world/
		package.json  <-- contains your package definition
		lib/
			index.js  <-- default module
			another-module.js 

In Seed, the default module is called 'index.js'.  This module usually exports
your public API. It is loaded when someone requires your package without 
naming a specific module (for example `require('markdown')` would look in the
'markdown' package for a module named 'index').  

Alternatively, you can omit the index.js file and use a file with the same 
name as the package.  In the example above, `index.js` could be renamed to 
`hello-world.js` and it would still load as the default.

## Defining Your Package.json

The most important part of a package is the package.json file.  This file 
describes the settings for your package to seed and other CommonJS-compliant
package managers.

At a minimum is must contain the following two properties:

	{
		"name": "hello-world",
		"version": "0.1.0"
	}
	
The name must be the name of the package.  It should match the name of the 
directory your package is in ('hello-world' in this example).  The version
number must increment anytime you plan to publish new changes.  It must 
follow the [semantic versioning spec](http://semver.org).

In addition to the above two properties, you will often want your package.json
to also list any dependencies like so:

	{
		"name": "hello-world",
		"version": "0.1.0",
		
		"dependencies": {
			"markdown": "1.0.0"
		}
	}
	
The dependencies hash describes all packages that your package requires in 
order to run.  When someone installs your package via seed, all of these
dependencies will be automatically fetched an installed as well.

Each item in the hash defines a single package.  The value is a required 
version number.  Since seed uses semantic versioning, naming a version number
here usually means that any version of the package can be used that is 
semantically compatible.  

In general this means seed will use any version of the package that is equal 
to or greater than the version named but less then the next major version 
number.  For example, requiring version '1.2.0' will get any version >= 1.2.0
but < 2.0.0.

If you do not want to use semantic versioning, you can force seed to use an 
exact version instead by prefixing the version with an '=' sign.  For example 
'=1.2.0' will _only_ load version 1.2.0 of the package - nothing later.  

You can also use '~' to simply load the latest installed version of the 
package, but that this is not recommended since it can make your code 
unstable.

# Loading Your Own Package

If you are writing an app as a package, you will often want to invoke the 
code from the command line.  To do this, you should create a bootstrap file
in a 'bin' directory within your package.  It should look like this:

	#!/usr/bin/env seed
	
	require.loader.register(__dirname, '..');
	require('hello-world').main();
	
The first line of code in this example tells seed that you want to register the top-level directory as the current package.  This will make the package 
visible to seed even though it is not installed in the system.

After you have registered your package you can require the package and invoke
any methods (like main()) in the example here.

## Using Nested Packages

One added benefit of working inside your own packages is that you can include
frozen or nested packages that will override any installed versions.  To add 
a nested package, just create a 'packages' directory and place your packages
inside of it.  Requiring those package names will cause them to be used 
instead.

Alternatively, if you have a package already installed and you just want to
get a copy into your own package to ensure it is always used, you can use the
`freeze` command:

	seed freeze markdown
	
This will copy the package file into your own packages directory.

If you want to edit the package, instead you can use the `fork` command,
which will look for a repository definition in the package.json and use it:

	seed fork markdown
	
# Pushing Packages

To push a package, you will first need to signup for an account or login in 
a machine.  If you have never pushed before, signup for an account with:

	seed remote signup
	
__NOTE:__ Due to limitations in Node, seed will echo your password to the 
screen as you type.

If you already have an account, you should login to a new machine:

	seed remote login
	
You only need to login or signup once per machine.  Once you've done so, you
can push a package with the `push` command:

	seed push ./hello-world
	
Note that you must pass the path to the package you want to push, not the 
package name.

Seed will only allow you to push a particular package version once.  If you 
need to make a change and push again, just bump the patch number and push 
again.

That's all the docs we have for now.  More is coming later.

	
	