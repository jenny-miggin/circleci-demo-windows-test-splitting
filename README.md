# CircleCI Windows Test Splitting Demo

## Intro

This demo includes an example of how to use CircleCI's [Test Splitting](https://circleci.com/docs/parallelism-faster-jobs/) functionality to split tests, by timing data, across a number of parallel Windows machine executors.

## Test Splitting

These unit tests are executed using `dotnet test`. The most complex part of this operation is ensuring the output of listing the tests (by running `dotnet test -t`) is compatible with the input required for the `dotnet test --filter` operation.

Tests results are saved in `junit` format, which ensures that CircleCI can [collect and analyse](https://circleci.com/docs/collect-test-data/) these test results and use the data for subsequent test-splitting operations.

## Run Tests Locally

To run the tests locally, navigate to the *PrimeService.Tests* directory and type the following commands:

```bash
dotnet restore
dotnet test
```

`dotnet restore` restores the packages of both projects.

`dotnet test` builds both projects and runs all of the configured tests.

**Note:** Starting with .NET Core 2.0 SDK, you don't have to run [`dotnet restore`](https://docs.microsoft.com/dotnet/core/tools/dotnet-restore) because it's run implicitly by all commands that require a restore to occur, such as `dotnet new`, `dotnet build` and `dotnet run`.
It's still a valid command in certain scenarios where doing an explicit restore makes sense, such as [continuous integration builds in Azure DevOps Services](https://docs.microsoft.com/azure/devops/build-release/apps/aspnet/build-aspnet-core) or in build systems that need to explicitly control the time at which the restore occurs.

## Source

This sample is part of the [unit testing tutorial](https://docs.microsoft.com/dotnet/core/testing/unit-testing-with-dotnet-test) for creating applications with unit tests included. See that topic for detailed steps on the code for this sample.

This sample also demonstrates creating a library and writing effective unit tests that validate the features in that library. The example provides a service that indicates whether a number is prime.
