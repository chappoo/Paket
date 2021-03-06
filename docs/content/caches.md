# Additional Caches

Paket works well with central repositories like [nuget.org](https://www.nuget.org/) or [myget.org](http://myget.org/). Under normal circumstances these repositories are always available and allow you to retrieve all packages for the rest of all time.
Unfortunately there are times when the central repository is not reachable or even worse: packages can be removed permanently from the feed.

This happened in other ecosystems, but also on [nuget.org](https://www.nuget.org/) and can lead to breaking builds and a lot of trouble for the users.

There are different solutions to this problem and one is using "additional caches" with Paket.

## Caching on a network share

By using a network share as an additional package cache Paket will store all used packages automatically on that network share.
Every restore process tries to retrieve the package from the cache before hitting the central repository.
As long as nobody deletes packages from the network share, package restore will continue to work.

Configuration of additional network share as caches can be done in the  [`paket.dependencies` file](dependencies-file.html):

    [lang=paket]
    source https://nuget.org/api/v2
    cache //hive/dependencies

    nuget Newtonsoft.Json
    nuget UnionArgParser
    nuget FSharp.Core

## Caching locally inside the repository

Many projects decide to commit all dependencies into the version control system.
Some people argue that this bloats the version control system, but it also ensures that dependencies are always available after checkout.

With a local dependencies cache you can instruct Paket to copy dependencies to a local folder.
This folder will then contain the `*.nupkgs files` of all dependencies and can be committed to source control.

<blockquote>In contrast to the default <i>packages</i> folder this new folder will only contain the zipped dependencies. This way the <i>packages</i> folder which contains also the unzipped versions can still be gitignored.</blockquote>

The configuration can be done in the  [`paket.dependencies` file](dependencies-file.html):

    [lang=paket]
    source https://nuget.org/api/v2
	cache ./nupkgs versions: current

    nuget Newtonsoft.Json
    nuget UnionArgParser
    nuget FSharp.Core

### Caching options

Paket allows you two caching options:

* `versions:all` - stores all currently used version of the dependencies.
* `versions:current` - stores only currently used version of the dependencies and deletes all other version. This option should not be used with network shares since it might affect other projects.

## Caches as package feeds

All configured caches are automatically used as additional package feeds. So if a package gets removed from the central repository even "paket update" will work.
The fact that it's now only found in the cache will be written to the [`paket.lock` file](lock-file.html).

All packages in the cache are treated as "unlisted". This means Paket's resolver will only use these packages in a new resolution if the central feed has no unlisted packages.

## NuGet machine cache

In addition to the above, Paket will also cache packages to a local machine cache, which defaults to `%LocalAppData%\NuGet\Cache`.  

This location can be overridden to support cases where parallelised package installs (or updates) are prone to file locks, such as when running multiple build agent processes on a single server.

To override at the process level, define a process environment variable called `"NuGetCachePath"` with the custom location, such as

    $env:NuGetCachePath = "C:\NuGetCaches\BuildAgentName"