---
title: Publish Go test results in Azure pipelines
published: true
---

A nice feature of Azure build pipelines is that you can publish test results and test coverage reports that are displayed with the results of a build. Unfortunately Go's test result file formats aren't supported, however in this post I'll show you how to convert them as part of the build process.

# Test Results

Aure build pipline report support a few test result formats, incluing the popular junit. Thankfully there's a tool to convert Go test results into junit reports: [go-junit-report](https://github.com/jstemmer/go-junit-report).

It's usage is simple, install with `go get` then pipe the results of `go test -v` into the tool, then pipe the results out into a file. This file can then be used by the PublishTestResults step in the Azure pipeline.

# Code Coverage

`go test` can create a coverage report using the `-coverprofile` flag. However, once again this isn't a file format supported by Azure. The good news is that the Cobertura format is supported, and there's a tool to convert the file: [gocover-cobertura](https://github.com/t-yuki/gocover-cobertura).

As always you install the tools (plus it's dependency) using `go get`, run your tests with the `-coverprofile=coverage.txt` flag, and finally pipe the coverage.txt file into the `gocover-cobertura` tool.

On thing to note is that - for reasons I've yet to dig in to - when you update the coverage results you'll have to set the `pathToSources` property otherwise the coverage report won't show the highlighted file contents in the report.

# Complete YAML steps

Here's the complete script to run all tests, convert the file formats and finally publish the reports. If you're using a dedicated build agent you could pre-install the tools then remove the go_test_report_tools_install step.

```yaml
- task: Bash@3
  name: "go_test_report_tools_install"
  inputs:
    targetType: "inline"
    script: |
      go get code.google.com/p/go.tools/cmd/cover
      go get github.com/t-yuki/gocover-cobertura
      go get -u github.com/jstemmer/go-junit-report
- task: Bash@3
  name: "go_test"
  inputs:
    targetType: "inline"
    script: "go test -v -coverprofile=coverage.txt -covermode count ./... 2>&1 | $(go env GOPATH)/bin/go-junit-report > report.xml"
- task: Bash@3
  name: "convert_coverage_to_cobertura"
  inputs:
    targetType: "inline"
    script: "$(go env GOPATH)/bin/gocover-cobertura < coverage.txt > coverage.xml"
- task: PublishTestResults@2
  inputs:
    testResultsFormat: "JUnit"
    testResultsFiles: "./report.xml"
    failTaskOnFailedTests: true
    testRunTitle: "Go Tests"
- task: PublishCodeCoverageResults@1
  inputs:
    codeCoverageTool: "cobertura"
    summaryFileLocation: "coverage.xml"
    pathToSources: "$(System.DefaultWorkingDirectory)"
```

Then when you look at the completed build pipeline you should see 2 new tabs (they can take a second to load):

![results tabs]({{ site.url }}/assets/post-images/test-coverage-tabs.png)

Any comments or suggestions? Chat to me on [twitter](https://twitter.com/RuddlesDev)
