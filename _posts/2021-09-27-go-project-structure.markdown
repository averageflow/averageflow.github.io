---
layout: post
title: Go Project Structure
date: 2021-09-27
---

In this post I will try to give an explanation to the project structure I follow commonly while making Go projects.

This is a basic layout for Go application projects. It's not an official standard defined by the core Go dev team; however, it is a set of common historical and emerging project layout patterns in the Go ecosystem. Some of these patterns are more popular than others. It also has a number of small enhancements along with several supporting directories common to any large enough real world application.

If you are trying to learn Go or if you are building a PoC or a toy project for yourself this project layout is an overkill. Start with something really simple (a single main.go file is more than enough). As your project grows keep in mind that it'll be important to make sure your code is well structured otherwise you'll end up with a messy code with lots of hidden dependencies and global state. When you have more people working on the project you'll need even more structure. That's when it's important to introduce a common way to manage packages/libraries. 

When you have an open source project or when you know other projects import the code from your project repository that's when it's important to have private (aka internal) packages and code. Clone the repository, keep what you need and delete everything else! Just because it's there it doesn't mean you have to use it all. None of these patterns are used in every single project. Even the vendor pattern is not universal.

With Go 1.14 Go Modules are finally ready for production. Use Go Modules unless you have a specific reason not to use them and if you do then you don’t need to worry about $GOPATH and where you put your project. The basic go.mod file in the repo assumes your project is hosted on GitHub, but it's not a requirement. The module path can be anything though the first module path component should have a dot in its name (the current version of Go doesn't enforce it anymore, but if you are using slightly older versions don't be surprised if your builds fail without it). See Issues 37554 and 32819 if you want to know more about it.

This project layout is intentionally generic and it doesn't try to impose a specific Go package structure.

This is a community effort. Open an issue if you see a new pattern or if you think one of the existing patterns needs to be updated.

If you need help with naming, formatting and style start by running gofmt and golint. Also make sure to read these Go code style guidelines and recommendations:

* [https://talks.golang.org/2014/names.slide](https://talks.golang.org/2014/names.slide)

* [https://golang.org/doc/effective_go.html#names](https://golang.org/doc/effective_go.html#names)

* [https://blog.golang.org/package-names](https://blog.golang.org/package-names)

* [https://github.com/golang/go/wiki/CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments)


More about naming and organizing packages as well as other code structure recommendations:

* [GopherCon EU 2018: Peter Bourgon - Best Practices for Industrial Programming](https://www.youtube.com/watch?v=PTE4VJIdHPg)

* [GopherCon Russia 2018: Ashley McNamara + Brian Ketelsen - Go best practices](https://www.youtube.com/watch?v=MzTcsI6tn-0)

* [GopherCon 2017: Edward Muller - Go Anti-Patterns](https://www.youtube.com/watch?v=ltqV6pDKZD8)

* [GopherCon 2018: Kat Zien - How Do You Structure Your Go Apps](https://www.youtube.com/watch?v=oL6JBUk6tj0)

 
## Go Directories

`/cmd`

Main applications for this project.

The directory name for each application should match the name of the executable you want to have (e.g., `/cmd/myapp`).

Don't put a lot of code in the application directory. If you think the code can be imported and used in other projects, then it should live in the `/pkg` directory. If the code is not reusable or if you don't want others to reuse it, put that code in the /internal directory. You'll be surprised what others will do, so be explicit about your intentions!

It's common to have a small main function that imports and invokes the code from the `/internal` and `/pkg` directories and nothing else.

`/internal`

Private application and library code. This is the code you don't want others importing in their applications or libraries. Note that this layout pattern is enforced by the Go compiler itself. See the Go 1.4 release notes for more details. Note that you are not limited to the top level internal directory. You can have more than one internal directory at any level of your project tree.

You can optionally add a bit of extra structure to your internal packages to separate your shared and non-shared internal code. It's not required (especially for smaller projects), but it's nice to have visual clues showing the intended package use. Your actual application code can go in the `/internal/app` directory (e.g., `/internal/app/myapp`) and the code shared by those apps in the `/internal/pkg` directory (e.g., `/internal/pkg/myprivlib`). I do very often have `/internal/domain`, `/internal/infrastructure` and `/internal/usecases` by using clean architecture.

`/pkg`

Library code that's ok to use by external applications (e.g., `/pkg/mypubliclib`). Other projects will import these libraries expecting them to work, so think twice before you put something here :-) Note that the internal directory is a better way to ensure your private packages are not importable because it's enforced by Go. The `/pkg` directory is still a good way to explicitly communicate that the code in that directory is safe for use by others. The I'll take pkg over internal blog post by Travis Jeffery provides a good overview of the pkg and internal directories and when it might make sense to use them.

It's also a way to group Go code in one place when your root directory contains lots of non-Go components and directories making it easier to run various Go tools (as mentioned in these talks: Best Practices for Industrial Programming from GopherCon EU 2018, GopherCon 2018: Kat Zien - How Do You Structure Your Go Apps and GoLab 2018 - Massimiliano Pippi - Project layout patterns in Go).

See the `/pkg` directory if you want to see which popular Go repos use this project layout pattern. This is a common layout pattern, but it's not universally accepted and some in the Go community don't recommend it.

It's ok not to use it if your app project is really small and where an extra level of nesting doesn't add much value (unless you really want to :-)). Think about it when it's getting big enough and your root directory gets pretty busy (especially if you have a lot of non-Go app components).

`/api`

OpenAPI/Swagger specs, JSON schema files, protocol definition files.

`/web`

Web application specific components: static web assets, server side templates and SPAs.

`/configs`

Configuration file templates or default configs. Put your confd or consul-template template files here.

`/init`

System init (systemd, upstart, sysv) and process manager/supervisor (runit, supervisord) configs.

`/scripts`

Scripts to perform various build, install, analysis, etc operations. These scripts keep the root level Makefile small and simple (e.g., https://github.com/hashicorp/terraform/blob/master/Makefile).

`/build`

Packaging and Continuous Integration. Put your cloud (AMI), container (Docker), OS (deb, rpm, pkg) package configurations and scripts in the /build/package directory.

Put your CI (travis, circle, drone) configurations and scripts in the /build/ci directory. Note that some of the CI tools (e.g., Travis CI) are very picky about the location of their config files. Try putting the config files in the `/build/ci` directory linking them to the location where the CI tools expect them (when possible).

`/deployments`

IaaS, PaaS, system and container orchestration deployment configurations and templates (docker-compose, kubernetes/helm, mesos, terraform, bosh). Note that in some repos (especially apps deployed with kubernetes) this directory is called `/deploy`.

`/test`

Additional external test apps and test data. Feel free to structure the /test directory anyway you want. For bigger projects it makes sense to have a data subdirectory. For example, you can have `/test/data` or `/test/testdata` if you need Go to ignore what's in that directory. Note that Go will also ignore directories or files that begin with "." or "_", so you have more flexibility in terms of how you name your test data directory.

`/docs`

Design and user documents (in addition to your godoc generated documentation).

`/tools`

Supporting tools for this project. Note that these tools can import code from the `/pkg` and `/internal` directories.

`/examples`

Examples for your applications and/or public libraries.

`/third_party`

External helper tools, forked code and other 3rd party utilities (e.g., Swagger UI).

`/githooks`

Git hooks.

`/assets`

Other assets to go along with your repository (images, logos, etc).

`/website`

This is the place to put your project's website data if you are not using GitHub pages.

## Directories You Shouldn't Have
/src

Some Go projects do have a src folder, but it usually happens when the devs came from the Java world where it's a common pattern. If you can help yourself try not to adopt this Java pattern. You really don't want your Go code or Go projects to look like Java :-)

Don't confuse the project level /src directory with the /src directory Go uses for its workspaces as described in How to Write Go Code. The $GOPATH environment variable points to your (current) workspace (by default it points to $HOME/go on non-windows systems). This workspace includes the top level /pkg, /bin and /srcdirectories. Your actual project ends up being a sub-directory under /src, so if you have the /src directory in your project the project path will look like this: /some/path/to/workspace/src/your_project/src/your_code.go. Note that with Go 1.11 it's possible to have your project outside of your GOPATH, but it still doesn't mean it's a good idea to use this layout pattern.