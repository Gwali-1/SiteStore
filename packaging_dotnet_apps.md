---
title: "Packaging .NET Applications"
slug: "Packaging-dotnet-applications"
date: "2025-12-02"
tags: ["C#", "dotnet"]
---

# Packaging and preparing .Net applications into deployable assets

It’s common for programming platforms to provide tools that help transform
your source code into an executable during development or into a deployable,
production-ready asset you can confidently distribute or deploy on a target platform.

On most platforms this will mean compiling your source code into a native binary.

The .NET platform works a bit differently. It runs applications inside a virtual environment called
the **Common Language Runtime (CLR)**, which offers features like memory management and type safety.
The build process is a little different here. When you build a .NET application, you're not producing a
native binary your CPU can interpret right away. Instead, your code is compiled
into an intermediate language called **Common Intermediate Language (CIL)**, which is a low-level,
platform-agnostic format that sits somewhere between your high-level C# code and native machine code.

This universal standardized platform-agnostic language in the .NET world is what the **CLR** executes.
The CLR **just-in-time (JIT)** compiles this CIL into native code at runtime.
This is the cross-platform compatibility of .NET.

> Note that starting from .NET 7+, you can skip this process entirely using Ahead-of-Time (AOT) compilation, which
> produces native
> code you can execute directly removing the need for JIT and the CLR at runtime.

Because of dynamics such as these, the .NET platform provides us with several interesting ways to bundle your apps
into deployable, production-ready formats.

In this post, we'll look at these interesting options, their cons and pros, and
a few transformations you can apply to optimize the final outputs for performance, size and portability.

## The dotnet publish command

On the .NET development platform, the dotnet CLI is the
entry point for tasks like building, running and packaging applications.

The command of interest here is the `dotnet publish` command.

Publishing is the process of turning your application into a deployable artifact. It compiles your code, resolves
dependencies, and outputs a directory of file(s) ready for deployment.
Depending on the options you provide, dotnet publish can produce very different types of output, ranging from simple
DLLs(Dynamically linked files)to native executables.

But before we explore those formats, let’s walk through an example.

## Example Application (a one endpoint web API)

To demonstrate the various deployment formats, let's consider a very simple web API with a single
endpoint.

Optionally if you would want to follow along and type out the commands locally
you can create the application by first creating a dotnet console app with command

```shell
dotnet new console -o OneEndpoint
```

Navigate into the `OneEndpoint` directory and replace the contents of Program.cs with:

```csharp
var builder = WebApplication.CreateBuilder();

var app = builder.Build();

app.MapGet("/", () => "Hello!!");

app.Run();
```

Then run the application:

```shell
dotnet run
```

Visit <http://localhost:5000> in your browser. You should see **Hello!!** displayed.

Everything good? Great! Let's move on.

## Packaging .Net applications into deployable assets

There are two main ways you might take to
distribute an application in the .NET world.

* Framework-dependent deployment approach.
* self-contained deployment approach.

What do these mean?

### Framework dependent deployment

To take the framework dependent route means you shall use the `dotnet publish`
command to produce a deployable asset that comprises your app's compiled DLLs, it's
dependencies and some metadata files.

The core runime binaries and assemblies needed to run your application are excluded,
requiring that where you intend to deploy or run your application must have
a compatible target runtime already installed.

Obviously you can see how excluding the runtime assemblies and other platform assemblies
in this approach will result in a relatively smaller size of deployable assets
than an approach that includes them.

Also with this packaging format, an advantage is target runtime
updates is easier since if you deploy a couple of applications this way on a
target platform, they all share the same runtime so updating runtime will affect
all of them.

This can also be a source of headache because since the applications are
tied to the runtime installed on the target host , there is always a risk of the
application breaking if the runtime version is not compartible with the one the
application was developed and packaged with.

Now let's look at how we shall package our `OneEndpoint` application into a
framework dependent distributable format.

Make sure you are in the `OneEndpoint` directory and run the command.

```shell
dotnet publish -c Release -o framework-dependent

```

*This command will package the application building it with a release configuration(turns
on compiler optimizations and removes debug symbols ) using the `-c Release`
flag as well specifying the output files should be put in a directory called
`framework-dependent`*

*Navigate into the directory, and you shall find DLL files of the application
and .json metadata files that contain information about application dependencies and
runtime information.*

We shall not go into detail about the contents of the `self-contained`
directory. What you need to understand is you have packaged the application
into a production ready deployable format.

All you need to do now is transfer the directory to the target host and start with
the runtime like this:

```shell
dotnet OneEndpoint.dll
```

Great! Now that we have seen how to package an application into a
deployable/distributable format that will require a .NET runtime already
installed on the target host we wish to deploy on and why we might even consider
using this option, let's now see how we might package it into a format
where we don't really require a .NET runtime available where we want to
deploy/distribute it.

A format where the deployable asset contains everything
your application needs to run.

### Self contained deployment

When you package a .NET application into a self-contained format, what you are
basically doing is producing a deployable asset that has the runtime, platform
assemblies and binaries included making it independent of a host runtime.

Meaning there is no need for the target host to already have .NET runtime
available.

Also note the deployable asset here is operating system(OS) specific
as there are runtime specific binaries that are included in the
packaging process.

Now let's look at how we shall package our `OneEndpoint` application into a
self-contained distributable format.

*This example assumes the target host is Mac operating system on x64
architecture and uses `osx-x64` as value for the runtime identifier `-r`.*

You can specify for other platforms like

* `linux-x64`  for Linux operating system on x64 architecture
* `win-x64`  for Windows operating system on x64 architecture

> Find more info on [runtime identifiers here](https://learn.microsoft.com/en-us/dotnet/core/rid-catalog)

Make sure you are in the `OneEndpoint` directory and run the command.

```
dotnet publish -c Release -r osx-x64 --self-contained true -o self-contained

```

We have packaged the `OneEndpoint` web API into a self contained deployable
format. You now have a self contained deployable asset as a directory with name`self-contained`.

When you navigate into this directory, one instant noticeable difference is
there is a lot more files here than there was when we packaged the application
as framework dependent. And that is because we now have to accommodate runtime
assemblies and dependencies needed to run our application.

You can see how this will affect the asset size. It will be much larger than
a framework dependent approach.

This format is however more portable as we can just copy the `self-contained`
directory and run the application anywhere without worrying if there is a .NET
runtime already installed.

You can run the application like this :

```shell
dotnet OneEndpoint.dll
```

Or use the outputted application binary as the entry point.

```shell
./OneEndpoint
```

So far we have looked at the two most common ways to bundle up a
.NET application for the purpose of distribution or deployment.
Before we explore the optimization transformations you can apply to these approaches, let’s consider one final packaging
option. One that works a little differently.

This next method stands out because it completely removes the need for the CLR at runtime.
Unlike framework-dependent and self-contained deployments which both compile to Intermediate Language (IL) and rely on
the CLR and Just-In-Time (JIT) compilation at runtime, this approach compiles your application directly to native
machine code.

Imagine skipping IL and JIT altogether, and ending up with a single, native binary that doesn’t need the .NET runtime to
be present on the host. That will be super cool? I'm glad you agree. Next up.

## Ahead-of-Time (AOT) Compilation

Starting .NET version 7, the platform introduced true Ahead-of-Time (AOT) compilation, allowing you to compile source
code directly into a native, OS-specific machine code binary. No Intermediate Language (IL), no Just-In-Time (JIT)
compilation, and no need for the .NET runtime at all.
You can execute your app directly as a native binary!

This makes deployment is super simple as all you need to do is distribute a single executable file.
You also get great performance benefits, especially in startup time, since there's no JIT compilation

However, note that there tradeoffs to be aware of when using this approach. You get relatively larger
distributable asset(binary) size as it will include pre-compiled native code, some runtime features and metadata.
Also, there are some limitations on refection and some dynamic features.

Let’s now compile our `OneEndpoint` application using AOT:

```shell
dotnet publish -c release -r osx-x64 /p:PublishAot=true /p:StripSymbols=true -o aot_compiled
```

*We specify two MSBuild properties to first enable AOT compilation and also to
instruct the AOT compiler to remove debug symbols from the native binary to getting
a leaner binary without debug information*

When you navigate into the `aot_compiled` directory you shall notice it's very
lean. You will find the native compiled binary for your application here.
You can copy this binary file to the target host and run your application with

```shell
./OneEndpoint
```

That’s it, your app is now running as a native binary with zero runtime dependencies!

Now let's quickly go through some transformations you might apply to the final outputs
using the above approaches.

## Packaging optimization transformations

### Trimming

By default, .NET includes all referenced assemblies in your application even though
you don't use all the classes and types they provide.This can result in unnecessarily large deployment assets.

To addres this, we can apply trimming transformation to remove all unused IL from your app and it's dependencies.

.NET uses a tool called `ILLinker` for this.It determines all unused code
that is safe to remove using static analysis.
Because the codebase is statically analyzed to determine what is safe to remove, this
process doesn't work well with reflection-heavy applications, since it's difficult for the tool
to detect usage of types or members that are only accessed via reflection.

Still, trimming can significantly reduce the size of your final output and is worth considering especially
for small web services and Command line utility programs.

To enable trimming, add the `/p:PublishTrimmed=true` property when publishing:

```shell
dotnet publish -c Release  /p:PublishTrimmed=true -o framework-dependent
```

### PublishSingleFile

You might have noticed that when we packaged our application as a self-contained deployment, the output directory still
contained several .dll files. If it's self-contained, you might wonder, why isn't it just a single file we can
distribute?

Well, even in self-contained deployments, the application is still based on the IL. The main
.dll or binary file essentially acts as a launcher, while the runtime and dependencies remain separate.

However, you can instruct the .NET SDK to bundle everything into a single executable file using
the `/p:PublishSingleFile=true` MSBuild property.

```shell
dotnet publish -c Release -r osx-x64 --self-contained  /p:PublishSingleFile=true
/p:PublishTrimmed=true \ true  -o self-contained
```

This command creates a self-contained, and trimmed single binary that you can easily distribute and run on the
target OS without needing to manage multiple files.

## Concluding

Alright, time to wrap things up.

In this post, we explored the different ways you can package a .NET application for distribution. From
framework-dependent and self-contained deployments to ahead-of-time (AOT) compilation.
We also looked at optimization techniques like trimming and publishing a single-file executable to help reduce size
and simplify distribution.
Hopefully, this gave you a clearer picture of the options available when it comes to getting your .NET application
ready to share with the world. Until next time!

