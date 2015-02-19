PEP: 003  
Title: SEED fix  
Author: Pierre Thirouin <pierre.thirouin@ext.mpsa.com>  
Priority: High  

## Abstract

Create a tool in Go that will be used by SeedStack users for
common project tasks. This tool will be a command line executable
providing the following sub command:

 - fix: Allow to upgrade a project to a new version of SeedStack.

## Motivation

SeedStack is currently a PSA inner project. As we are planning to open
source it, we have to do a huge refactoring in order to make it
independent from our IT organization. This refactoring will require a
lot of changes in our internal applications, but direct benefits
will be invisible. So to encourage applications to migrate to the OSS
version of SeedStack, we are planning to build an upgrader tool. On
another hand, this tool should help fixing deprecated API without
inciting them.

Building this tool, we will keep in mind scalablity and modularity, in order to
not be a one shot project but a future-proof tool for API refactoring.
However, we don't plan for now to provide behavior refactoring, but
only API renaming.

## Specification

### Goal

We will refer to "seed" tool's "fix" subcommand as "seedfix".

The goal of seedfix is to apply idenpotent transformations on a source
directory.

### Transformations description format

Transformations will be described in a YAML file with the following
structure:

```yaml
---
- 
  pre:
    - precondition1
    - precondition2
  proc:
    -
      transformation-name:
        - "parameter1"
        - "parameter2"
-
  ...
```

These transformations are ordered and represented by:

 - a set of preconditions filtering the group of files to update.
 - a set of procedures linking to function to apply on the files

Sample:

```yaml
- 
  pre: 
    - "file:*-root/pom.xml"
    - isJarType
  proc: 
    - 
      update-groupdId: 
        - "com.mycompany:org.mycompany"
    - 
      update-artifactId: 
        - "my-arifact2:my-artifact2"
        - "my-arifact1:my-artifact1"
        - "my-arifact3:my-artifact3"
    - remove-unused-boms
    - upgrade-seedstack-bom-version
- 
  pre: 
    - "file:*.props"
  proc: 
    - replace
    - flatten-props
```

### List of transformations

For **pom files**:
 - Update dependencies (groupId and artifactId)
 - Check and clean maven boms and explicit dependency versions
 - Update component versions

For **props files**:
 - flatten sections
 - Rename/Replace keys
 - Search & replace values

For **java files**:
 - Search & replace (mostly for imported packages)
