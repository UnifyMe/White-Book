# Code routines

## Organization

A build matrix will show status info for each build platform, target and options
for all the projects and publicly available at all times.

There will be no testing or release stage, but instead all commits will be
tested, and merged on `master` if tests passes, having it always a clean,
production-ready status. Releases will be published automatically using
[semantic-release](https://github.com/semantic-release/semantic-release). Nobody
will have direct access to `master` branch except the CI servers, and will only
accept fast-forward merges.

Having `master` branch protected, it doesn't matter if pull-requests come from
branches or forks, so we'll allow both. In the future, maybe we restrict main
repo branches to oficial ones and dictate all pull-requests come from forks to
simplify management.

When `master` branch gets updated, all pull-requests and branches will be
updated and re-checked. This will give to the developer the responsability that
his code pass the tests, and also would help to maintain backwards compatibility.
In case a backwards compatibility breakage happens, it could be moved to a
`next-major` branch and merge changes there. Change of a major version needs to
be discussed, but probably would happens when there's no open other non-breaking
pull-requests changes.

## Architecture

Architecture will be designed based on the usage of
[Fat clients](https://en.wikipedia.org/wiki/Fat_client), logic will be hosted by
the client apps, being used the servers mostly to store data for billing and
machine learning algorythms. Comunications with diferent platforms will be done
directly by the clients, in case some server is needed for proxying or data
conversion it will be powered by serverless services, Node.js PaaS or
[NodeOS](http://node-os.com) instances.

Interchange and storage format will be based on JSON, and communications will be
done using a REST and JSON-RPC APIs, both on HTTP, SSE or WebSockets.

App should try to integrate and use available host system services and data
instead of replicate them (specially on mobile platforms), doing the conversion
in real time. Being able to replace and integrate all the system communications,
it should be able to be configured as the default app for diferent intents and
URL schemes handlers, but also check and update the system info in case in some
moment the application is not being used. It could also be able in the future to
do other tasks like launch system apps (UnifyOS, like Facebook mobile OS layer).

## Code style

Code will follow BSD style (i.e. opening brackets on new line, where aplicable),
without semi-colons (where aplicable), 80 columns, tabbed to two spaces, one
whiteline between methods and functions and two between sections. Otherwise
that, code must follow best practices and widely accepted community standards,
and in doubts use Python PEP-8 guidelines as reference (when aplicable). This is
done this way to improve legibility. Also when possible advanced languages
features will be used (for example `map()`, `filter()` or `reduce()` instead of
plain `for` loops if it doesn't affect performance or can allow them to be put
on external clossures). Another good style reference is
[Google Python style](https://google.github.io/styleguide/pyguide.html).

All files newlines will by in UNIX format (`\n`).

## Design clean APIs

APIs should be minimalist and expose only public methods and attributes, both on
the constructor and `prototype` chain, making inaccessible any internal and/or
private state. Tests must be done ONLY against the public API, if some internal
state it's needed to be accesible for testing or by some child class, it will be
done accesible by using an underscore-leaded non-writable and non-enumerable
property, to prevent it of being used by accident on regular production code. In
more advances cases where they need to be set (for example, doing dependencies
injection to mock internal objects during testing), consider doing so by using
optional arguments or an *options bag* argument object, so they don't need to be
explicitly defined for the most basic cases. Don't break previous API usage of
the simple and default cases except if some of their required arguments can be
considered optional in some other simpler use cases (so they are not the most
simple and basic usage anymore).

## Modularize everything

- Think big, do small
- [Don't Repeat Yourself](https://en.wikipedia.org/wiki/Don't_repeat_yourself)
- [KISS principle](https://en.wikipedia.org/wiki/KISS_principle)
- UNIX philosofy

Use and create reusable modules and components. Before writting your own one,
investigate first if there's some actual library or module that does what you
want to achieve with no changes or with minimal improvements, specially if it
does a complex task or has good community reputation (high number of stargazers
on Github or downloads on npm) or a good code quality.

Clearly identify and isolate what's bussiness model and what can be written and
freely distributed and used by others as a generic and standalone project. Don't
overdesign on a first take, but if you see something on the source code that can
be extracted and refactored in a more generic, reusable and abstract way, also
in simple cases like duplicated code, don't hesitate and go for it.

If some internal component needs to be accesed just for debugging purposses,
consider about how to publish it on the public API in a more generic way in case
it could be useful in some other legit use cases, or better than that, move it
to its own module or package making it a lower level public API (and doing so,
with its own tests too).

## Document everything

`README.md` file should have a minimal purposse and usage info of the project
itself with some examples, like suggested at
[Feed me readmes](https://github.com/LappleApple/feedmereadmes). This must have
enought info so a project newcomer can be able to install all the project
dependencies, build it from scratch and fully testing it without needing to ask
for advice. In case someone has doubts on this step, it must be updated to fix
and clarify them.

Code should have [JsDoc](http://usejsdoc.org)
[DocStrings](https://en.wikipedia.org/wiki/Docstring) for all modules, classes
and methods, and code inside a function/method with some own identity should be
fenced with whitelines and a comment before it explaining in one or two lines
what it's going to do (think of it as the function name or description in case
it was written or converted as a standalone function).

In-depth documentation will be written in [GitBook](https://www.gitbook.com)
format under the `/docs` folder of each project (both manually written and/or
autogenerated) and published on [Github Pages](https://pages.github.com) by
using the Github projects auto-publish feature.

## Log everything

Logging needs to be done in a smart way, not just adding traces to know where
and how the code is running though and what function is executing at any exact
moment that decrease performance and add noise to log files. Instead, log all
events and data that comes in and out of the system, so later errors and
executions can be reproducible.

For normal execution, `stdout` must be clean and `stderr` show only warnings and
error messages, and events and log entries stored on `.jsonl`
[JSON Lines files](http://jsonlines.org) having their timestamp, full
module/class/method name, module version, arguments, event name and payload.

Errors should clearly identify the problem without ambiguities, and ideally have
included debug info for the developer. To don't leak info to final users, give
them just only a log file identifier, an error unique UUID, and the hash of the
error log entry so it's possible to univoquely identify the logged error entry
with the full undisclosed error info.

Javascript errors must be serialized and included with their stacktrace on the
error log entry, probably as JSON format. This can't be done by default, so
Error objects must be augmented to allow JSON serialization.

## Test everything

Don't do TDD except if you are really sure of how some user stories and/or API
parts will look like (and related to this, when you don't get your code to fully
work as you desired), but instead do some (basic) regression tests after your
code is working correctly and before pushing it to the repo. It's acceptable
that tests (and fixes) are on their own commits after the ones holding the code
of your implementation. In the long term, you should try to achieve 100% branch
coverage whenever you can, and be sure percentage never drops, so you can be
able to acomplise refactoring without fear. Automated code quality metrics will
be added in the short future to check this pitfalls for you.

## Update everything

Code must always be updated, with nightly checks for dependencies new versions
and ensure they are working correctly. Useful tools for that task can be
[GreenKeeper](https://greenkeeper.io) and [buho](https://github.com/piranna/buho).
Other checks for style and/or new features would need to be done by hand from
time to time, or develop style checkers for them if not available. Once an
update makes a nightly to fail, it must be fixed inmediatly, and maybe also
notified to the dependency author.

Code should not have regressions, once one is found having a test for it or not,
it will have the highest priority and must be fixed before adding new
functionality, except if that new functionality would remove it because it's
related or would deprecate it anyway.
