A collection of scripts for downloading a prebuilt version of LiteCore.

These are for Couchbase internal use only and will not function otherwise.

There are two actual scripts here, each with a similar purpose:

## fetch_litecore.py (SHA based)

```
usage: fetch_litecore.py [-h] -v PLATFORM [PLATFORM ...] [-d] [-D] [-s SHA] [--ce CE] [--ee EE] [-x EXT_PATH] [-o OUTPUT] [-q]

Fetch a specific prebuilt LiteCore by SHA

optional arguments:
  -h, --help            show this help message and exit
  -v PLATFORM [PLATFORM ...], --variants PLATFORM [PLATFORM ...]
                        A space separated list of variants to download
  -d, --debug           If specified, download debug variants
  -D, --dry-run         Check for existience of indicated artifacts, but do not perform download
  -s SHA, --sha SHA     The SHA to download. If not provided, calculated based on the provided CE and (optionally) EE repos. Required if
                        CE not specified.
  --ce CE               The path to the CE LiteCore repo. Required if SHA not specified.
  --ee EE               The path to the EE LiteCore repo
  -x EXT_PATH, --ext-path EXT_PATH
                        The path in which the platform specific extensions to this script are defined (platform_fetch.py). If a relative
                        path is passed, it will be relative to fetch_litecore.py. By default it is the current working directory.
  -o OUTPUT, --output OUTPUT
                        The directory in which to save the downloaded artifacts
  -q, --quiet           Suppress all output except during dry run.
  ```

This is the old style of downloading based on the git commit SHA of the CE, and optionally EE, repos of LiteCore.  There are two operation modes:

**Automatic Mode**

If you pass the `--ce` argument (and optionally the `--ee` argument) then the script will automatically use the SHAs of whatever you have checked out locally for calculating what to download.

**Manual Mode**

Otherwise, if you pass in the `-s` argument this calculation will skip and the script will directly go to fetch based on the SHA you have specified.

## fetch_litecore_version.py (Build ID based)

```                                                                                   [9:06:58]
usage: fetch_litecore_version.py [-h] -v PLATFORM [PLATFORM ...] [-d] [-D] [-b BUILD] [--ee] [-r REPO] [-x EXT_PATH] [-o OUTPUT] [-q]

Fetch a specific prebuilt LiteCore by build version

optional arguments:
  -h, --help            show this help message and exit
  -v PLATFORM [PLATFORM ...], --variants PLATFORM [PLATFORM ...]
                        A space separated list of variants to download
  -d, --debug           If specified, download debug variants
  -D, --dry-run         Check for existience of indicated artifacts, but do not perform download
  -b BUILD, --build BUILD
                        The build version to download (e.g. 3.1.0-97 or 3.1.0-97-EE). Required if repo is not specified.
  --ee                  If specified, download the enterprise variant of LiteCore
  -r REPO, --repo REPO  The path to the CE LiteCore repo. Required if build not specified.
  -x EXT_PATH, --ext-path EXT_PATH
                        The path in which the platform specific extensions to this script are defined (platform_fetch.py). If a relative
                        path is passed, it will be relative to fetch_litecore.py. By default it is the current working directory.
  -o OUTPUT, --output OUTPUT
                        The directory in which to save the downloaded artifacts
  -q, --quiet           Suppress all output except during dry run.
  ```

This is the new style of downloading based on a version and build number (aka build ID) of LiteCore.  It also has two operation modes:

**Automatic Mode**

If you pass in the `-r` argument then the build ID will be read from the commit message of the commit checked out currently.  It must be on a branch that contains this information (e.g. `staging/master`) in the form `Build-To-Use: <ID>` where ID is something like *3.1.0-107*.  This will fetch CE unless you pass the `--ee` switch.

**Manual Mode**

Otherwise, if you pass the `-b` argument this processing is skipped and the sript will directly go to fetch based on the build ID you have specified.  As with automatic mode this means downloading CE unless the `--ee` switch is specified.

## Extending the Extraction Functionality

You will notice that both scripts take an `-x` argument to specify a directory containing a file called `platform_fetch.py`.  Since various projects have differing requirements in folder structure for the extracted artifacts, the projects have the ability to override the path to which the artifacts are extracted based on the current platform and architecture.  The following is a list of applicable values:

|       VARIANT       |    OS    |      ABI      |
| ------------------- | -------- | ------------- |
| android-x86_64      | android  | x86_64        |
| android-x86         | android  | x86           |
| android-armeabi-v7a | android  | armeabi-v7a   |
| android-arm64-v8a   | android  | arm64-v8a     |
| centos6 (deprecated)| centos6  | x86_64        |
| linux               | linux    | x86_64        |
| macosx              | macos    | \<empty>      |
| ios                 | ios      | \<empty>      |
| windows-arm-store   | windows  | arm-store     |
| windows-win32       | windows  | x86           |
| windows-win32-store | windows  | x86-store     |
| windows-win64       | windows  | x86_64        |
| windows-win64-store | windows  | x86_64-store  |

ios and macos are both universal binaries and thus have no specific single ABI inside and are therefore empty.

The platform fetch script should contain one function with the following signature:

`subdirectory_for_variant(os: str, abi: str) -> str` 

It takes as arguments the values of OS and ABI above and returns a path relative to the specified output directory into which the archives will be extracted.