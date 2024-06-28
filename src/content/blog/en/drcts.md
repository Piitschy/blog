--- 
title: 'drcts - Directus Migration CLI'
description: 'drcts is a Go-CLI tool to migrate directus databases. The ease of use and the ability to migrate databases in one step make drcts a useful tool for developers working with directus.'
pubDate: 'Jul 08 2024'
heroImage: '/picography-laptop-notepad-desk-1.jpg'
tags:
    - directus
    - migration
    - cli 
    - go 
    - golang
    - tool 
---

<!--toc:start-->
- Introduction](#introduction)
- Installation](#installation)
- Usage](#usage)
<!--toc:end-->

# Directus Migration CLI

## Introduction

[`drcts`](https://github.com/Piitschy/drcts) is a Go-CLI tool to migrate [directus](https://directus.io/) databases. 
The ease of use and the ability to migrate databases in one step make `drcts` a useful tool for developers working with directus.
For example, if you want to migrate a production database to a test database without transferring user data, `drcts` is exactly what you are looking for!

## Installation

```bash
go install github.com/Piitschy/drcts@latest
```

## Usage

To use `drcts` you need the access data with the necessary rights for the databases that are to be migrated.
There are two ways to pass the access data:
- Environment variables
- flags

[Here](https://github.com/Piitschy/drcts?tab=readme-ov-file#environment-variables) you will find a detailed list of the necessary environment variables

Under the hood, `drcts` uses the [directus API](https://docs.directus.io/api/reference.html) to migrate the databases, simplifying the handling considerably and guiding the user through the process. 
A simple command is sufficient (after installation) to migrate the databases:

```bash
drcts migrate
```

Now `drcts` guides you through the process and asks you for the necessary information to migrate the databases.

In addition to the migration, `drcts` also offers the possibility to save schemas, compare them with each other and apply differences between two schemas directly.
Make sure that you have the necessary rights to migrate the databases, as `drcts` does not perform a rights check.
Also keep in mind that this tool is only intended for migrating schemas and does not migrate data!

### Save schema

```bash
drcts save -o schema.json
```
This command saves the database schema in the file ``schema.json``.
This file can later be used to create a difference file, for example.
This allows you to compare schemas from the same database at different times.

If you want to use `drcts` within a script, you can also specify credentials and environment variables directly via flags:

```bash
drcts --bu <base-url> --bt <base-token> save -o <output-file>
```

### Apply schema 

If you want to apply a saved schema to a database, use the ``apply`` command:

```bash
drcts apply -i schema.json
```

Now the schema from the file `schema.json` is applied to the database and the schema is adjusted accordingly.

### Compare schema

To compare the saved schema with the current state of the Dirctus instance, use the command `save-diff`:

```bash
drcts save-diff -i schema.json -o diff.json
```

Now you can view the differences in the file `diff.json` to decide if and which differences you want to apply.

### Apply differences 

To apply the differences from the file `diff.json`, use the command `apply-diff`:

```bash
drcts apply-diff -i diff.json
```

The differences are now applied to the database and the schema is adjusted accordingly.

## Help 

For detailed help on the commands and options, use the command `help`:

```bash
drcts help
drcts <command> help
```
or look in the `readme` of the [GitHub repository](https://github.com/Piitschy/drcts).

Translated with DeepL.com (free version)
