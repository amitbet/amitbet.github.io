---
layout: post
title: "Typescript: sharing typings between modules"
date: 2016-04-24 21:25:06
tags : [NodeJS, typescript]
category : typescript
description: Explaining about typescript modules and typings, and how to properly use them to get a fully typed application
toc: true
---

# What's this 'sharing typings' that you speak of?
Typescript is a totally cool language, which has been gathering a sizable following lately. It comes as no surprise, Given the help it provides when scaling large JS projects into enterprise products.

The only problem is that it is still a bit rough around the edges in a couple of places, one of these sore spots is extending the typing system to cover external modules. Such modules may come from two main sources - you either download some npm module from the net, or create a shared module by yourself. The first type would most likely be written in JS and require hand coded definitions, the second one would almost surely be in TS, given that you are reading this post...

Just to complicate matters there are two types of modules (ambient and external), old (and deprecated) three slash reference declarations (///reference)  and if we want to get picky, two types of installer utils: tsd (deprecated) and typings (current), but you should only use the latter. The biggest problem with these tools and typings in general, is the lack of coherent online documentation about the way they should be used, the specific quirks of this system, and how to troubleshoot the problems when they arise (since they really tend to pop up). This is a problem I will try to correct right here in this post.

# What are typings?
Typings are typescript definition files (*.d.ts) which help with code completion and enforcing the type system in compilation time, similar to header files in cpp, Typescript requires them if a type is used and has no definition in the scope of the current module, in contrast to cpp header files, TS will always accept JS definitions, so you can always skip using them completely by using __var x = require__, but that comes with the price of not having the type system extend to your common & 3rd party components. (which is not really a healthy solution).

We have two types of modules in TS, internal and external, internal modules are defined within the same file, and enclosed by an explicit module declaration   (__module YYY{}__), external modules are defined by your *.ts file, and include all that is marked with '__export__'

When using the '__typings__' utility to install type files you may get two forms of module definitions (both are external modules):

## Ambient definitions
these definition style will usually be seen in JS modules coded by hand, since it's main purpose is making it easier to cram several module definitions in the same file, it uses the 'declare module' keywords to define several separate module declarations:

```javascript
declare module "url" {
    export interface Url {
       protocol?: string;
       hostname?: string;
       pathname?: string;
   }
  export function parse(urlStr: string, parseQueryString?, slashesDenoteHost?): Url;
}

declare module "path" {
   export function normalize(p: string): string;
   export function join(...paths: any[]): string
   export var sep: string;
}
```

this way large definition bodies like 'node.d.ts' can be written without fragmenting it into many smaller definition files.

## Regular external definitions 
usually TS auto generated module definitions look like the below example, they include only one module per file, similar to the way the original external module was written:   

```javascript
import {Worker} from './worker';

export declare class Manager {
  constructor(name: string, id:number);
  privage _workers;
  addWorker(w: Worker):void;
}

```  

# Using the typings utility:
Using typings is very similar to npm, except for the extra need to provide whether or not the definition you want is ambient. 

For JS npm modules you write:
```
typings install the-module-name --save --ambient
```
For modules with built in regular style TS definitions from the global npm repo (these are few and far between):
```
typings install the-module-name --save
```
For TS defined modules, which may be your own npm-linked modules, Sinopia contained modules, or other TS modules from GitHub that contain regular external module definitions, you need to explicitly state the location of their definition files (this is taken from 'typings install -h'):

```
typings install [<name>=]<location>

  <name>      Module name of the installed definition
  <location>  The location to read from (described below)

Valid Locations:
  [<source>!]<pkg>[@<version>][#<tag>]
  file:<path>
  github:<org>/<repo>[/<path>][#<commitish>]
  bitbucket:<org>/<repo>[/<path>][#<commitish>]
  npm:<pkg>[/<path>]
  bower:<pkg>[/<path>]
  http(s)://<host>/<path>

  <source>    The registry mirror: "npm", "bower", "env", "global", "lib" or "dt"
              When not specified, `defaultSource` or `defaultAmbientSource` in
              `.typingsrc` will be used.
  <path>      Path to a `.d.ts` file or `typings.json`
  <host>      A domain name (with optional port)
  <version>   A semver range (E.g. ">=4.0")
  <tag>       The specific tag of a registry entry
  <commitish> A git commit, tag or branch
```

## TroubleShooting & Special situations
There are a couple of edge cases to lookout for:

### Diamond dependencies:
typescript has an issue with classes that have private members & functions, it will compare them by structure when validating types, verses types without private members, that would be type-validated by type name only,  as to why do private members should even matter to this subject, i don't really understand but a bug is a bug. This issue cases a strange behaviour so that when importing definitions from a common module A, the definition would be re-generated in a module B that imports A, so that if you had another module C, which imports both A and B, the type C gets directly from A (let's call it C.a), is different from the same type it obtained through a function in B (B.a).

So if: A-B-C and  A-C, a type 'a' might be different from another 'a' coming from b (b.a)...
This can be solved by creating an interface in A, and exporting it instead of the class. this will eliminate the problem, since the interface has no private types and will be validated according to it's type name (so no new definitions will be created in module B for the types coming from A).

### 