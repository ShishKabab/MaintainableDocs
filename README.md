Maintainable docs
=================

This project wants to help teams manage their documentation processes in a more structured way by providing automated libraries tools to detect which documentation needs to be written and updated. The problem with current practices is that the information what needs to be documented is scattered all over the place (in people's heads, PR request, notes, etc.) So, the theory behind this project is:

* Have a way to automatically detect what consitutes your public API (a system component, a library, or a collection of libraries) and track changes in that API.
* Have a way to talk about use-cases of your public API, and which parts of the API they touch.
* Have a way to link your documentation to this API

With these components, you could detect which parts of the API are undocumented, need use-case descriptions (how-to guides for example), and are out-of-date. This project should therefore become a re-usable library (not a [framework](https://www.programcreek.com/2011/09/what-is-the-difference-between-a-java-library-and-a-framework/)) that can be pieced together for different kind of languages/sources (TypeScript code, GraphQL library, etc.) using that information in different kinds of ways (CI/CD pipeline, tools for doc contributors to see what needs to be done, etc.)


Status
======

So far, this is just a repo to brainstorm and lay out architecture and use cases. It shouldn't be to hard to get a PoC in here soon, though.


Desired developer experience
============================

Let's say we're creating a library with one entry point. This may be something like this in TypeScript:

```typescript
import ExpenseSource from './expense-sources'
import Reporter from './reporters'
import { ExpenseData } from './types'

export * from './expense-sources'
export * from './reportes'

... some high-level classes and functions here, like a class that can load expenses from sources and send them to reporters ...
```

We should be able to translate that with a command or library to a language-independent data structure describing the API (theoretical example expressed in YAML for readibility):

```yaml
classes:
    ExpenseReporter:
        version: 'hash generated from hash of methods, properties, etc.' # Should be something more intelligent, to distiguish backward-compatible changes from non-
        methods:
            report:
                version: 'hash generated from function signature'
                arguments:
                    - name: data
                      type: ExpenseData
            
interfaces:
    ExpenseData:
        version: 'hash'
        properties:
            amount:
                version: 'hash'
                type: float
                description: This may be extracted from TSDoc
functions:
    reportExpenses:
        version: 'hash'
        
```

Then we'd maintain either by hand, or with a tool, how these APIs relate to use-cases:

```yaml
set-up:
    description: Setting up the library
    flow:
        - instantiate-subclass-of: ExpenseSource
        - instantiate-subclass-of: ExpenseReporter
    variants:
        - typescript
        - javascript
reporting-expenses:
    description: Reporting expenses
    flow:
        - use-case: set-up
        - call-function: reportExpenses
```

Now, imagine this tool:

```sh
$ tool init
Wrote a doc state file to `./.docstate`. You should check this into version control.
$ tool check
Error: There's no API docs for
    these classes:
        - ExpenseReporter
    these interfaces
        - ExpenseData
    these functions:
        - reportExpenses
    these use-cases
        - set-up
        - reporting-expenses
$ # I write docs for the use-case set-up
$ tool mark-done use-case:set-up.typescript
$ tool report
Error: There's no API docs for
    these classes:
        - ExpenseReporter
    these interfaces
        - ExpenseData
    these functions:
        - reportExpenses
    these use-cases
        - reporting-expenses
        
Also, the use-case 'set-up' is missing the following variants:
- javascript
$ # I now make a backwards incompatible change to how reportExpense is called

$ tool report
Error: There's no API docs for
    these classes:
        - ExpenseReporter
    these interfaces
        - ExpenseData
    these functions:
        - reportExpenses
    these use-cases
        - reporting-expenses
        
Also, the use-case 'set-up'
    is missing the following variants:
        - javascript
    and was written for a an incompatible version of the function `reportExpenses`
```

And so on. It would detect:
* Incompatible API changes (changed signatures, deleted methods, etc.)
* Compatible changes like extra arguments that could have a lower priority
* Use-case docs that are out-of-date
* Added units (classes, functions, etc.) without docs
* Units without any linked use cases

Ideally, one could imagine a custom mark-down tag for example to automatically link documentation to these structures without calling the `mark-as-done` command. The end-result should be minimal overhead documenting stuff, and coming up with use-cases that should be documented.

Open questions
==============

* How does this relate to Readme Driven Development?
* How to enable easy feedback and contributions?
* What data model enables us to have a generic representation of relationships that can be used for different use cases? (e.g. an library has classes with methods, but a REST API contains endpoints related to certain types, and C has functions that interact with types and kind of serve as class members.)
