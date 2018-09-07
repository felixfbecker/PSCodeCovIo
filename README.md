# PSCodeCovIo <img src="https://raw.githubusercontent.com/codecov/media/master/logos/logo.svg?sanitize=true" height="90" align="left">

[![powershellgallery](https://img.shields.io/powershellgallery/v/PSCodeCovIo.svg)](https://www.powershellgallery.com/packages/PSCodeCovIo)
[![downloads](https://img.shields.io/powershellgallery/dt/PSCodeCovIo.svg?label=downloads)](https://www.powershellgallery.com/packages/PSCodeCovIo)

Helper to use in CI to convert test coverage data emitted by [Pester](https://github.com/pester/Pester) to JSON understood by [codecov.io](https://codecov.io/).

Requires `git` to be installed and in the `PATH` (this is true for Travis and AppVeyor).

## How to use

### Travis

```yml
# ...

install:
  # Install modules
  - pwsh -c '& {
      Install-Module Pester;
      Install-Module PSCodeCovIo;
    }'

script:
  # Run tests and generate coverage report
  - pwsh -c '& {
      $res = Invoke-Pester -PassThru -CodeCoverage;
      Export-CodeCovIoJson -CodeCoverage $res.CodeCoverage -RepoRoot $pwd -Path coverage.json;
      if ($res.FailedCount -gt 0) {
        throw "$($res.FailedCount) tests failed."
      }
    }'

after_success:
  # Upload coverage report to codecov
  - bash <(curl -s https://codecov.io/bash) -f coverage.json
```

### AppVeyor

```yml
# ...

install:
  - ps: Install-Module Pester -Scope CurrentUser -Force -SkipPublisherCheck

test_script:
  # Run tests and generate coverage report
  - ps: |
      $testResultsFile = ".\TestsResults.xml"
      $res = Invoke-Pester -PassThru -CodeCoverage;
      Export-CodeCovIoJson -CodeCoverage $res.CodeCoverage -RepoRoot $pwd -Path coverage.json
      if ($res.FailedCount -gt 0) {
        throw "$($res.FailedCount) tests failed."
      }

after_test:
  # Upload coverage report to codecov
  - ps: |
      $env:PATH = 'C:\msys64\usr\bin;' + $env:PATH
      Invoke-WebRequest -Uri 'https://codecov.io/bash' -OutFile codecov.sh
      bash codecov.sh -f coverage.json
```

## Reference

### Syntax
```powershell
Export-CodeCovIoJson [-CodeCoverage] <Object> [-RepoRoot] <String> [[-Path] <String>] [<CommonParameters>]
```

### Parameters
- `-CodeCoverage <Object>`
  The CodeCoverage property of the output of Invoke-Pester with the -PassThru and -CodeCoverage options
- `-RepoRoot <String>`
  The path to the root of the repo.  This part of the path will not be included in the report.  Needed to normalize all the reports.
- `-Path <String>`
  The path of the file to write the report to.
