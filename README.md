# Ash

Ash is a modular Bash framework written with ease of use + reusability in mind.

> **Note:** This project is in early development, and versioning is a little different. [Read this](http://markup.im/#q4_cRZ1Q) for more details.

# Why should you care?

Building command line tools in Bash is an extremely tedious and somewhat enigmatic task.  There's quite a bit of boilerplate code you're going to have to write if you want your script to do more than just one thing, which will only clutter your script.  In addition, your scripts will likely never be able to reference good code you've written from old scripts.

Ash helps you get rid of all of your boilerplate by letting you call functions directly from the command line, while also providing a modular approach to scripting which will allow you to share code between scripts.

You are able to build a module independently that functions as a CLI or as a library (or any combination of the two), and easily share your module with the world.

# Installation

Run this line right here, and you should be good to go:

```bash
curl https://raw.githubusercontent.com/ash-shell/ash/master/install.sh | sh
```

> [This script](/install.sh) simply clones down this repo to `/usr/local` and links the [ash file](/ash) to `/usr/local/bin`, which is usually always a `$PATH` directory.  In the event that this one liner doesn't work (or you don't want to run a script downloaded over the network), you can simply recursively clone this repo, and add the [ash file](/ash) to somewhere in your $PATH.

# Modules

Modules are the fundamental building blocks of Ash.  They allow you to build out custom CLI's and libraries that can be used in any other Ash module.

> In this section I will be building a hypothetical module called `Wrecker` to illustrate usage

## Module Structure

A module looks something like this:

```
├── ash_config.yaml
├── callable.sh
├── classes
│   └── WreckerClass.sh
|   └── ...
└── lib
    └── wrecker_library_file.sh
    └── ...
```

> Feel free to add any folders or files you want to this -- this is simply the base structure that Ash uses.  You can't break anything by adding new folders or files.  You'll see I've even done this myself in [ash-make](https://github.com/ash-shell/ash-make).

#### ash_config.yaml

This is the only required file in an Ash module.  This file tells Ash that your project is an Ash module.

Here is an example `ash_config.yaml` file:

```yaml
name: Wrecker
package: github.com/ash-shell/wrecker
default_alias: wrecker
callable_prefix: Wrecker
```

**`name`**: This is the human readable name of your module.  This value is used by other modules who might want to output the name of the current module.  This field is **required**.

**`package`**: This is the unique identifier to your project.  I strongly suggest keeping this the same as your git project url, or at minimum scoped under a domain you own to prevent any collisions with other ash modules.  This field is **required**.

**`default_alias`**: This is the default alias of your project.  When you install a module, it has to be aliased so it can reasonably be called by ash from the command line.  This value should be something short and sweet, and you don't need to worry about collisions with other packages (the user will be asked to come up with a new alias in the event there is a collision).  This field is **required**.

**`callable_prefix`**: This field specifies the function prefix that Ash looks for in the callable file when calling upon a module (more detail in [the section below](#callable-modules)).  This field is only required for modules that provide a callable file.

#### callable.sh

This file is only required if you want your library to be callable.  The contents of this file are explained in [this section](#callable-modules).

#### classes/

This directory is where you would place your modules classes.  Only class files at the root of this directory will be usable.  This is an optional folder.

The README of [ash-shell/obj](https://github.com/ash-shell/obj) goes into full detail of how this all works.

> TLDR: Ash has native Bash object support

#### lib/

This is another optional directory.  If you want your module to provide a functional based library, this is where you would place those files.

Other packages will be able to import all of the root files in the `lib` directory via:

```bash
Ash__import "your/modules/package/name"
```

You can nest folders inside of `lib` for structure, but you'll need to manage importing those files yourself as `Ash__import` won't import them.

Files in the `lib` directory are auto loaded for you in the callable portions of your module, so you don't have to import your own module.

## Callable Modules

You can build your module to be directly callable from the command line.

The first thing you will need to do in your module is add `callable_prefix` to your [ash_config.yaml](#ash_configyaml) file.

```yaml
callable_prefix: Wrecker
```

Now you can create a `callable.sh` file and add it to the root of your module.

> One of the most important things to understand about callable files is that they are just bash files.  They provide immediate access all variables set in the [ash file](/ash), and import all of the [core module libraries](/core_modules).

Your newly created callable file will look something like this:

```bash
#!/bin/bash

Wrecker__callable_echo(){
    echo "Hello Echo"
}

Wrecker__callable_main(){
    echo "Hello"
    normal_bash_function
}

normal_bash_function(){
    echo "World"
}
```

#### Callable Functions

Callable functions are functions that can be called directly from the command line.

If your module is installed and aliased as `wrecker`, you can call the callable functions from the command line.

To call the `Wrecker__callable_echo`, simply call this in the command line:

```bash
ash wrecker:echo
```

To call `Wrecker__callable_main`, simply call this in the command line:

```bash
ash wrecker:main
```

`main` is actually a magical name for a callable, as you can simply call the main callable like this:

```bash
ash wrecker
```

## Library Modules

Library modules are modules who provide either a `lib` and/or `classes` directory.  Modules can be both callable and library modules at the same time.

#### Ash__import

`Ash__import` is how modules load in other modules `lib/` files.  You pass this function the package name of another module to import it.

Example usage:

```bash
Ash__import "github.com/ash-shell/slugify"
```

#### Obj__import

> This is technically an [ash-shell/obj](https://github.com/ash-shell/obj) function, so check out that projects README to get a deeper understanding.

`Obj__import` is how modules load in other modules `class/` files.  You pass this function the package name of another module, and also an alias for that module.

```bash
Obj__import "github.com/ash-shell/wrecker" "wrecker"
```

# The .ashrc File

The `.ashrc` file is a file loaded in by the Ash core which you can use in configuring your modules.  The `.ashrc` file is located in your home directory `~/.ashrc` (you're going to have to create this file yourself).

This file is optional in terms of the Ash core, but may be required for some modules that require an `.ashrc` configuration.

It's worth noting that this is the *first* thing that is loaded in Ash, so module writers don't have to worry about a users `.ashrc` causing any variable/function collisions with their modules, as everything in your module is loaded after and will take priority.

# License

[MIT](LICENSE.md)
