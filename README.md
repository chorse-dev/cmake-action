# CMake Action

Use [GitHub Actions](https://github.com/features/actions) as a
[Dashboard Client](https://cmake.org/cmake/help/latest/manual/ctest.1.html#dashboard-client)
for [CMake](https://cmake.org/) projects.

Unlike other "cmake-action" implementations, this action does not provide custom
wrappers for the [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html)
command-line, but instead builds on top of CTest's
[scripting](https://cmake.org/cmake/help/latest/manual/ctest.1.html#dashboard-client-via-ctest-script)
capabilities. 

## Available Inputs

| Name | Description |
| --- | --- |
| `cmake-generator` | Specify a build system generator |
| `coverage-command` | Command-line tool to perform software coverage analysis |
| `memorycheck-command` | Command-line tool to perform dynamic analysis |
| `memorycheck-type` | Specify the type of memory checking to perform |
| `submit-url` | The URL of the dashboard server to send the submission to |

## Available Outputs

| Name | Description |
| --- | --- |
| `build-errors` | Store the number of build errors detected |
| `build-warnings` | Store the number of build warnings detected |
| `memcheck-defects` | Store the number of memcheck defects found |

## Example Usages

When not giving any other options, this action configures, builds, and tests
a CMake project.  The default generator is "Unix Makefiles".  A different
generator may be selected with the `cmake-generator` input:

```yaml
jobs:
  Ubuntu:
    name: Ubuntu - Unix Makefiles
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: purpleKarrot/cmake-action@master
  Ubuntu-Ninja:
    name: Ubuntu - Ninja
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: seanmiddleditch/gha-setup-ninja@master
      - uses: purpleKarrot/cmake-action@master
        with:
          cmake-generator: Ninja
  MacOS:
    name: MacOS - Xcode
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - uses: purpleKarrot/cmake-action@master
        with:
          cmake-generator: Xcode
  Windows:
    name: Windows - Visual Studio 17 2022
    runs-on: windows-2022
    steps:
      - uses: actions/checkout@v4
      - uses: purpleKarrot/cmake-action@master
        with:
          cmake-generator: Visual Studio 17 2022
```

### Setting compiler flags

When using [gcov](https://gcc.gnu.org/onlinedocs/gcc/Gcov.html), it is necessary
to pass `--coverage` to the compiler.  It is easiest to set the flag in the
environment:

```yaml
env:
  CFLAGS: --coverage
  CXXFLAGS: --coverage
steps:
  - uses: actions/checkout@v4
  - uses: purpleKarrot/cmake-action@master
    with:
      coverage-command: gcov
```

Similarly, if you want to use any of clang's sanitizers, set both the compiler
and necessary compiler flags as environment variables:

```yaml
env:
  CC: clang
  CXX: clang++
  CFLAGS: -fno-omit-frame-pointer -fsanitize=address
  CXXFLAGS: -fno-omit-frame-pointer -fsanitize=address
steps:
  - uses: actions/checkout@v4
  - uses: purpleKarrot/cmake-action@master
    with:
      memorycheck-type: AddressSanitizer
```

### Submit the results to CDash

Results may be sent to a dashboard server:

```yaml
- uses: purpleKarrot/cmake-action@master
  with:
    submit-url: https://open.cdash.org/submit.php?project=MyProject
```
