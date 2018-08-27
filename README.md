# go-module-knobs

## List of go module knobs for controlling CI, vendoring, and when go commands access the network

By default, a command like `go build` ignores the vendor directory and will reach out to the network as needed to satisfy imports.

Some teams will not want the go tooling to touch the network in CI, whereas other teams might want greater control during day-to-day work regarding when the go tooling updates `go.mod`, how dependencies are obtained, and how vendoring is used.

This list is an attempt to build a more consolidated enumeration of some of the primary knobs and options that people might use to adapt the default Go behavior to better suite their particular use cases. Most of this information is currently spread throughout different sections of the official [documentation](https://tip.golang.org/cmd/go/#hdr-Modules__module_versions__and_more) or [modules wiki](https://github.com/golang/go/wiki/Modules).

The intent of this list is to help socialize the existence of these knobs, without giving all the details for each. Because different teams will likely want to weave these together in different ways, this document does not prescribe any particular workflow, but it might serve help while the community is starting to develop workflows, write more introductory blogs, etc.

* `GOFLAGS` environment variable
     * Allows you to set a particular go command flag by default.
     * Can be useful in CI or testing workflows, or if you have an opinion on what the default flags/behavior should be for the go tool for your day-to-day development workflows (e.g., perhaps by opting in to vendoring by setting `GOFLAGS=-mod-vendor` in your .bashrc or to control).
     * More details: [CL](https://go-review.googlesource.com/c/go/+/126656), [tip documentation](https://tip.golang.org/cmd/go/#hdr-Environment_variables)
     
* `-mod=readonly` flag (e.g., `go build -mod=readonly`, `go test -mod=readonly`)
     * Prohibits most go commands (except `go get` and `go mod`) from modifying go.mod, causing the command to fail if it would have otherwise implicitly wanted to update go.mod.
     * Can be useful to check that go.mod does not need updates, such as in continuous integration or testing.
     * More details: [tip documentation](https://tip.golang.org/cmd/go/#hdr-Maintaining_module_requirements) and comment [here](https://github.com/golang/go/issues/26850#issuecomment-411903910).

* `go mod vendor` command
     * Resets the main module's vendor directory to include all packages needed to build and test all of the main module's packages based on the state of the `go.mod` files and Go source code.
     * Different teams have different philosophies on vendoring, but some teams prefer to check dependencies into source control using vendoring. Vendoring also provides resiliency in the event of external sources going down, disappearing, or moving.
     * Can also be used to have the same set of dependencies for users that are using older versions of Go (because older versions of Go will find and use the vendor directory)
     * Also supports testing with older versions of Go (e.g., Go 1.9 and 1.10) during CI
     * More details: tip documentation [here](https://tip.golang.org/cmd/go/#hdr-Make_vendored_copy_of_dependencies) and [here](https://tip.golang.org/cmd/go/#hdr-Modules_and_vendoring)

* `-mod=vendor` flag (e.g., `go build -mod=vendor`, `go test -mod=vendor)
     * Uses the main module's top-level vendor directory to satisfy dependencies (disabling use of the usual network sources and local caches). Ignores the dependency descriptions in go.mod and assumes that the vendor directory holds the correct copies of dependencies. Note that only the main module's top-level vendor directory is used; vendor directories in other locations are still ignored.
     * More details: tip documentation [here](https://tip.golang.org/cmd/go/#hdr-Modules_and_vendoring) and [here](https://tip.golang.org/cmd/go/#hdr-Maintaining_module_requirements)
   
* `GO111MODULE=off` environment variable
     * go commands never uses the new module support. Instead they look in vendor directories and GOPATH to find dependencies (following pre-1.11 behavior).

* `GOPROXY=off` environment variable
     * go commands in module mode are disallowed from using the network for dependencies.
     * More details: [CL](https://go-review.googlesource.com/c/go/+/126696)

* `GOPROXY=file:///filesystem/path` environment variable
     * go commands will use a filesystem (local or remote) to resolve dependencies, without any need to have an actual running proxy process.
     * Note that the go command stores downloaded dependencies in its local cache ($GOPATH/pkg/mod) and its cache layout is the same as the requirements for a proxy, so that cache can be used as the content for a filesystem-based GOPROXY or simple webserver used as a GOPROXY. 
     * In addition, `go mod download` populates $GOPATH/pkg/mod/cache/download, which means that command can be used to pre-populate or update the contents for a GOPROXY.
     * More details: [tip documentation](https://tip.golang.org/cmd/go/#hdr-Module_proxy_protocol)

* Open source distributed module repositories such as Project Athens
     * One goal is to support a globally hosted "always on" module repository for public code.
     * A separate goal is a stand-alone proxy server that can be deployed on-premise to cache and control available Go modules for an organization.
     * See for example: https://github.com/gomods/athens

* `go mod download` command
     * Most day-to-day use cases do not require this (because normally the go command will automatically download modules as needed).
     * Primarily targeted at pre-warming caches for docker builds or in some cases CI.
     * Also likely will be used by proxies (e.g., Project Athens or perhaps a simple internally developed proxy) as a way to obtain module files on cache miss.
     * More details: [tip documentation](https://tip.golang.org/cmd/go/#hdr-Download_modules_to_local_cache)
      
* `replace` directives in go.mod
     * Provides additional control in the top-level `go.mod` for what is actually used to satisfy a dependency found in the Go source or go.mod files.
     * The `replace` directive is flexible, and supports multiple use cases. 
     * One sample use case is if you need to fix something in a dependency, you can have a local fork and use a `replace example.com/original/import/path => your/forked/import/path` in your top-level `go.mod` (without needing to update the import paths in the actual source code). The `replace` directive allows you to supply another import path that might be another module located in VCS (GitHub or elsewhere), or on your local filesystem.
     * `replace` also allows the top-level module control over the exact version used for a dependency (e.g., `replace example.com/some/dependency => example.com/some/dependency@v1.2.3`).
     * More details: [tip documentation](https://tip.golang.org/cmd/go/#hdr-Edit_go_mod_from_tools_or_scripts)
      
