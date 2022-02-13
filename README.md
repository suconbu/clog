# clog

This is logging utility for C++.

Example:

```cpp
#include "clog/clog.h"

int main()
{
    CLOG << "message";
    CLOG_ERROR << "error no:" << 99;
    CLOG_DEBUG.format("debug count:%d", 9999);

    return 0;
}
```

Result:

```
[2022-02-11 19:42:07.280] [-----] [example.cpp] [int main()@5] message
[2022-02-11 19:42:07.280] [error] [example.cpp] [int main()@6] error no:99
[2022-02-11 19:42:07.280] [debug] [example.cpp] [int main()@7] debug count:9999
```

Please see [example.cpp](/example/example.cpp) for more example.

# Features

- Single header
- Severity filter (At comple time)
- Tag filter by pattern matching (At comple time)
- Stringify arguments and their values
- Supports C++11 and above

# Setup

1. Place `clog/clog.hpp` in include path of your project.
2. Add `#include "clog/clog.hpp"` into your source code.

# Usage

## Logging marcros

```cpp
CLOG << "log";
CLOG_DEBUG << "debug";
CLOG_INFO << "info";
CLOG_WARN << "warning";
CLOG_ERROR << "error";

// Specify the tag string
CLOG_("tag") << "log";
CLOG_DEBUG_("tag") << "debug";
CLOG_INFO_("tag") << "info";
CLOG_WARN_("tag") << "warning";
CLOG_ERROR_("tag") << "error";
```

## Option definitions

You can change the behavior by enabling some definitions before `#include "clog/clog.hpp"`.

Example:

```cpp
#define CLOG_OPTION_MIN_SEVERITY_WARN
#define CLOG_OPTION_OUTPUT_TO std::cerr
#include "clog/clog.hpp"
...
```

Definitions:
[example.cpp](/example/example.cpp)
| Name | Description | Default | Example |
| ------------------------------ | ------------------------------------------------- | --------------- | ------------------------------------------------- |
| CLOG_OPTION_MIN_SEVERITY_INFO | Limits the output severity to INFO/WARN/ERROR | (No definition) | [example_severityfilter.cpp](/example/example_severityfilter.cpp) |
| CLOG_OPTION_MIN_SEVERITY_WARN | Limits the output severity to WARN/ERROR | (No definition) | [example_severityfilter.cpp](/example/example_severityfilter.cpp) |
| CLOG_OPTION_MIN_SEVERITY_ERROR | Limits the output severity to ERROR | (No definition) | [example_severityfilter.cpp](/example/example_severityfilter.cpp) |
| CLOG_OPTION_DENY_TAGS | Regular expressions for tag that suppress logging | {} | [example_tagfilter.cpp](/example/example_tagfilter.cpp) |
| CLOG_OPTION_DENY_INCLUDE_TAGS | Regular expression that excludes log suppression | {} | [example_tagfilter.cpp](/example/example_tagfilter.cpp) |
| CLOG_OPTION_DISABLE_LOGGING | Turn off the all logging | (No definition) | [example_disable.cpp](/example/example_disable.cpp) |
| CLOG_OPTION_OUTPUT_TO | Specifies the output stream / handler | std::cout | [example_fileoutput.cpp](/example/example_fileoutput.cpp), [example_outputhandler.cpp](/example/example_outputhandler.cpp) |

## Output stream / handler

`CLOG_OPTION_OUTPUT_TO` specify where to output the log in following types.

- Output stream (std::ostream and inherited class)
- Function (void func(const clog::record&))

Example:

```cpp
#define CLOG_OPTION_OUTPUT_TO std::cout                 // Default
#define CLOG_OPTION_OUTPUT_TO my_stream                 // To file stream
#define CLOG_OPTION_OUTPUT_TO clog::windows_debugger    // Preset handler for Windows (Use "OutputDebugStringA")
#define CLOG_OPTION_OUTPUT_TO clog::android_debugger    // Preset handler for Android (Use "__android_log_print")
#define CLOG_OPTION_OUTPUT_TO my_handler                // Use custom handler
#include "clog/clog.hpp"

std::ofstream my_stream("my_output.log");

void my_handler(const clog::record& record)
{
    printf("[my-logging] %s\n", record.message().c_str());
}
...
```

## Tag filter by regular expression

`CLOG_OPTION_DENY_TAGS` and `CLOG_OPTION_DENY_EXCLUDE_TAGS` macros control supression the unnecessary logs by pattern matching for tag string.

The special characters you can use in regular expressions are:

| Special character | Description                                                |
| ----------------- | ---------------------------------------------------------- |
| .                 | matches any single character                               |
| ^                 | matches the beginning of the input string                  |
| $                 | matches the end of the input string                        |
| \*                | matches zero or more occurrences of the previous character |

Example code:

```cpp
// Suppress logs which contained "noisy" in tags.
#define CLOG_OPTION_DENY_TAGS { "nois.", "verbose" }
// However, logs which tag starts with "not" will still be output.
#define CLOG_OPTION_DENY_EXCLUDE_TAGS { "^not" }
#include "clog/clog.hpp"

int main()
{
    CLOG_("noise") << "noise noise noise";
    CLOG_("verbose") << "the significantly verbose message";
    CLOG_("more noisy") << "noisy noisy noisy";
    CLOG_("not noisy") << "tweet tweet";
    CLOG << "chirp chirp";

    return 0;
}
```

Result:

```
[2022-02-11 19:42:07.280] [-----] [example.cpp] [int main()@12] tweet tweet
[2022-02-11 19:42:07.280] [-----] [example.cpp] [int main()@13] chirp chirp
```

## Stringify arguments and their values

`CLOG_ARGS` macro converts arguments and their values into strings.  
You can safely inputs variables here that possibility of nullptr.

Example code:

```cpp
#include "clog/clog.hpp"

int main()
{
    int n = 1;
    unsigned int u = 0xFFFFu;
    double f = 3.141592653589793;
    std::string s = "String";
    void* p = nullptr;

    CLOG << CLOG_ARGS(n, u, f, s, p);

    return 0;
}
```

Result:

```
[2022-02-11 19:42:07.280] [-----] [example.cpp] [int main()@11] (n, u, f, s, p) -> (1, 65535, 3.14159, String, (null))
```

# Lisence

MIT License.
