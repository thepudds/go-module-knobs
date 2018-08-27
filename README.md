# go-module-knobs

## List of primary Go module knobs for controlling CI, vendoring, and when go commands access the network

By default, a command like `go build` ignores the vendor directory and will reach out to the network as needed to satisfy imports.

Some teams will not want the go tooling to touch the network in CI, whereas other teams might want greater control over when the go tooling touches the network during day-to-day development activities and how dependencies are obtained.

This is a consolidated list providing some of the knobs and options that people might use in support of their workflows related to those types of use cases. Most of this information is currently spread throughout in different sections in the official documentation.

The current intent of this list is to help socialize the existence of these knobs, without giving all the details for each. This list does not prescribe any particular workflow given different teams will likely want to weave these together in different ways, but this list might be useful as input as the community starts developing module workflows, writing blogs, etc.

* `GOFLAGS` environment variable
     * Allows you to set a particular go command flag by default.
     * Can be useful in CI or testing workflows, or if you have an opinion on what the default flags/behavior should be for the go tool for your day-to-day development workflows (e.g., perhaps by opting in to vendoring by setting `GOFLAGS=-mod-vendor` in your .bashrc or to control).
     * More details: [CL](https://go-review.googlesource.com/c/go/+/126656), [tip documentation](https://tip.golang.org/cmd/go/#hdr-Environment_variables)
