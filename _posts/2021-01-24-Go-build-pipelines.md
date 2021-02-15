---
title: 1 minute Go build pipelines
published: true
---

One of the things I'm enjoying about working in Go is how fast the compile times are. Not only does that make the local dev/test loop fast but it means we can create more involved automated build pipelines and have them still run in under a minute.

Given I use hosted build agents a lot of tasks have an install step then an execute step. The following lists the pre-test-and-build tasks I currently have in a Go pipeline in Azure:

# GoImports Formatter

I'm a big fan of opinionated formatting tools, they removed the need for the old "standards documents" we used to know and hate, and get rid of the arguments about what's "right".

Our local environments are set up to format the code with goimports (basically gofmt that also orders the imports) on save. That said, things go wrong and new members of the team may not be set up correctly so I check for formatting errors in the pipeline and fail the build if any are found.

First install with

```bash
go get -u golang.org/x/tools/cmd/goimports
```

Then run it with

```bash
test -z $($(go env GOPATH)/bin/goimports -l .)
```

It's worth noting that goimports always returns an exit code of 0 regardless of the result, so I use the `-l` argument to get a list of files with errors, then use the bash `test -z` command to return a non-zero exit code if there are any lines in the output. One minor issue with this setup is `test` will swallow the output, so you know that it failed but don't know why.

# Golangci Linter

The default go linter doesn't offer much in the way of configuration, so I use [golangci-lint](https://golangci-lint.run/) which offers a lot more power and configuration. By default I use the strict settings found [here](https://github.com/golangci/golangci-lint/blob/master/.golangci.yml)

First install with

```bash
curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin v1.32.2
```

Then run with

```bash
$(go env GOPATH)/bin/golangci-lint run ./...
```

Something to keep in mind with this is that it uses the local yml config file so it's worth keeping an eye on any changes to that file.

# Go Mod Outdated

This is an optional step, I don't break the build if this finds anything but log a warning so I can manually review the output. It simply checks to see if there are any new releases of the go packages being used.

First install with

```bash
go get -u github.com/psampaz/go-mod-outdated
```

Then run with

```bash
go list -u -m -json all | $(go env GOPATH)/bin/go-mod-outdated -direct -ci || { echo "##[warning]Some dependencies are out of date"; exit 0; }
```

# Nancy

Nancy is a tool created by Sonatype which checks your packages against a CVE database and breaks the build if there are any security issues found. It's similar to `npm audit` and [OWASP Dependency Check](https://owasp.org/www-project-dependency-check/)

First install with

```bash
curl -L "https://github.com/sonatype-nexus-community/nancy/releases/download/v1.0.1/nancy-linux.amd64-v1.0.1" -o "$(go env GOPATH)/bin/nancy"; chmod +x $(go env GOPATH)/bin/nancy
```

Then run with

```bash
go list -json -m all | $(go env GOPATH)/bin/nancy sleuth
```

# Conclusion

At this point you're ready for the test and build process to get the final executable.

![build time 55 seconds]({{ site.url }}/assets/post-images/build-time.png)

I tend to have the steps above in a separate pipeline file that's included with all go builds, then include it in the projects pipeline so these steps can be easily updated.

In the near future we're moving to a container instance as abuild server, so we can pre-install these tools and bring the build time down further.
