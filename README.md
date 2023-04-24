# CircleCI Windows Test Splitting Demo

Test splitting on Windows using `dotnet test` can be complex due to the way in which the list of tests available is generated, and how the list needs to be formatted in order to be accepted after splitting. While this guide uses `dotnet test`, it can serve as some guidance on how splitting using other frameworks may be implemented.

## Splitting your tests

### Generate a list of tests

Use `dotnet test --list-tests | tee raw-output-from-vstest.txt` to generate a list of tests that are currently available and save the list to a file.

### Format the full test list correctly

As the output in `raw-output-from-vstest.txt` has additional text in it, this then needs to be stripped away and saved to `pool.txt` before CircleCI can read through it and split the list: `sed -e '1,/The following Tests are available:/d' -e 's/^[[:space:]]\{1,\}//g' raw-output-from-vstest.txt | tee pool.txt`

### Split the tests

Now that we have a file with just a list of tests within it, we can now pass this list through CircleCI's [test splitting](https://circleci.com/docs/parallelism-faster-jobs/) functionality to split the tests and save the data to a file:
`circleci tests split --split-by timings --timings-type testname <pool.txt | tee tests-to-run.txt`.  Once the tests have been executed the first time, test timing data will be available and used to split the tests by timing across the number of parallel containers you have specified.

#### Executing Test Splitting on Self-Hosted Runner

If executing test splitting on self-hosted Windows executors, like an EC2 instance in AWS, then the command for splitting tests changes to: `circleci-agent tests split...` otherwise you will see the below error:

```bash
Passing through `circleci tests`
Error: failed to proxy command [tests], expected this to be called inside a job: not supported by windows
```

There is no requirement for the CircleCI CLI to be available in this case

### More formatting before execution

Thank you `dotnet` for not making this easy :) This part was the trickiest as it was unclear how the `dotnet test --filter` command expected the list of tests to be formatted. Eventually we discovered that this format is acceptable: `DisplayName='test1'|DisplayName='test4'|DisplayName='test23'` so we now need to format our newly-generated list of assigned tests in this way: `sed -e 's/\([()]\)/\\\1/g' -e 's/\r\{0,1\}$/|DisplayName=/g' tests-to-run.txt | tr -d '\r\n' | sed -e 's/^/DisplayName=/' -e 's/|DisplayName=$//' | tee filter-value.txt`

### Execute the tests

Finally, we have a list of tests that are formatted correctly, and have been split correctly and saved to the `filter-value.txt` file. Now we can execute the tests: `dotnet test --filter "$(Get-Content filter-value.txt)" --logger junit`

## Saving Test Results

### Using JUnit Logger

A crucial part of this is ensuring test results are being saved in the [correct JUnit XML format](https://circleci.com/docs/collect-test-data/) so that test results can be interpreted for test timing data, as well as used by Insights to provide data on test duration, failure rates etc. Ensure you include the [JUnit XML Logger](https://www.nuget.org/packages/JunitXml.TestLogger/) in the `*.csproj`: `<PackageReference  Include="JunitXml.TestLogger"  Version="3.0.124"  />`

## References

### Credits

Thank you to [@Makoto](https://github.com/makotom) for the sedding patience and Windows wizardry :) 

### Source

This sample is part of the [unit testing tutorial](https://docs.microsoft.com/dotnet/core/testing/unit-testing-with-dotnet-test) for creating applications with unit tests included. See that topic for detailed steps on the code for this sample.

This sample also demonstrates creating a library and writing effective unit tests that validate the features in that library. The example provides a service that indicates whether a number is prime.
