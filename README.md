# Rust bindings for ESP-IDF (Espressif's IoT Development Framework)

## Background

The ESP-IDF API in Rust, with support for each ESP chip (ESP32, ESP32S2, ESP32C3 etc.) based on the Rust target.

![CI](https://github.com/esp-rs/esp-idf-sys/actions/workflows/ci.yml/badge.svg)

## Build

- To build this crate, please follow all the build requirements specified in the [ESP-IDF Rust Hello World template crate](https://github.com/esp-rs/esp-idf-template)
- The relevant Espressif toolchain, as well as the `esp-idf` itself are all automatically
  downloaded during the build by
    - with the feature `pio` (default): utilizing [platformio](https://platformio.org/) (via the [embuild](https://github.com/ivmarkov/embuild) crate) or
    - with the feature `native` (*experimental*): utilizing native `esp-idf` tooling.
- Check the [ESP-IDF Rust Hello World template crate](https://github.com/esp-rs/esp-idf-template) for a "Hello, world!" Rust template demonstrating how to use and build this crate.
- Check the [demo](https://github.com/ivmarkov/rust-esp32-std-demo) crate for a more comprehensive example in terms of capabilities.

## Feature `pio`
This is currently the default for installing all build tools and building the ESP-IDF framework. It uses [PlatformIO](https://platformio.org/) via the
[embuild](https://github.com/ivmarkov/embuild) crate.

The `pio` builder installs all needed tools to compile this crate as well as the `esp-idf` itself. 
The location where the `esp-idf` source and tools are detected and installed is the following:
- **`<crate workspace-dir>/.embuild/platformio`**
  - This is the location used by default
- **`~/.platformio`**
  - This is the "standard" PlatformIO location, where all tooling as well as the ESP-IDF is installed
- **`$ESP_IDF_SYS_PIO_INSTALL_DIR`**
  - This is a user-provided location
  - To enable it, simply define the `ESP_IDF_SYS_PIO_INSTALL_DIR` variable to point to a directory of your preference

### (PIO builder only) Using cargo-pio to interactively modify ESP-IDF's `sdkconfig` file

To enable Bluetooth, or do other configurations to the ESP-IDF sdkconfig you might take advantage of the cargo-pio Cargo subcommand:
* To install it, issue `cargo install cargo-pio --git https://github.com/ivmarkov/cargo-pio`
* To open the ESP-IDF interactive menuconfig system, issue `cargo pio espidf menuconfig` in the root of your **binary crate** project
* To use the generated/updated `sdkconfig` file, follow the steps described in the "Bluetooth Support" section

## Feature `native`
This is an experimental feature for downloading all tools and building the ESP-IDF framework using the framework's "native" own tooling.
It will become the default in the near future.
It also relies on build and installation utilities available in the [embuild](https://github.com/ivmarkov/embuild) crate.

Similarly to the `pio` builder, the `native` builder also automatically installs all needed tools to compile this crate as well as the `esp-idf` itself. 
The location where the `esp-idf` source and tools are detected and installed can be one of the following ones:
- **`<crate workspace-dir>/.embuild/espressif`**
  - This is the location used by default
- **`~/.espressif`** 
  - This is the "standard" ESP-IDF tools location
  - To enable it, set the environment variable `ESP_IDF_GLOBAL_INSTALL` to 1
- **`$ESP_IDF_INSTALL_DIR`**
  - This is a user-provided location
  - To enable it, simply define the `ESP_IDF_INSTALL_DIR` variable to point to a directory of your preference

### (Native builder only) Using cargo-idf to interactively modify ESP-IDF's `sdkconfig` file

TBD: Upcoming

## Configuration

Environment variables are used to configure how the ESP-IDF framework is compiled. 

Note that instead of / in addition to specifying those on the command line, you can also put these in a `./config/cargo.toml` file inside your crate directory 
(or a parent directory of your crate) by using the recently stabilized Cargo [configurable-env](https://doc.rust-lang.org/cargo/reference/config.html#env) feature.

The following environment variables are used by the build script:

- `ESP_IDF_SDKCONFIG_DEFAULTS` (*native* and *pio*): 

    A `;`-separated list of paths to `sdkconfig.default` files to be used as base
    values for the `sdkconfig`. If such a path is relative, it will be relative to the
    cargo workspace directory (i.e. the directory that contains the `target` dir).


- `ESP_IDF_SDKCONFIG` (*native* and *pio*):     

    The path to the `sdkconfig` file used to [configure the
    `esp-idf`](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/kconfig.html).
    If this is a relative path, it is relative to the cargo workspace directory.

    **Note** (*native* only):   
    The cargo optimization options (`debug` and `opt-level`) are used by default to
    determine the compiler optimizations of the `esp-idf`, **however** if the compiler
    optimization options are already set in the `sdkconfig` **they will be used instead.**


- `ESP_IDF_PIO_INSTALL_DIR` (*pio* only):

    The path to the directory where PlatformIO is/will be installed. If it is set to a
    relative path, it is relative to the crate workspace-dir.

    If not set, the PlatformIO installation location defaults to the local install dir `<crate-workspace-dir>/.embuild/platformio`.
    However, the user can still force the crate to use the global PlatformIO install dir (`~/.platformio`)
    by setting `ESP_IDF_PIO_GLOBAL_INSTALL` to `1`.


- `ESP_IDF_INSTALL_DIR` (*native* only):

    The path to the directory where all esp-idf tools are installed. If it is set to a
    relative path, it is relative to the crate workspace-dir.

    If not set, the esp-idf source & tooling location defaults to the local install dir `<crate-workspace-dir>/.embuild/espressif`.
    However, the user can still force the crate to use the global esp-idf tooling install dir (`~/.espressif`)
    by setting `ESP_IDF_GLOBAL_INSTALL` to `1`.


- `ESP_IDF_PIO_GLOBAL_INSTALL` (*pio* only):

    If set to `1`, `true`, `y` or `yes` uses the global install directory (that is, unless `ESP_IDF_PIO_INSTALL_DIR` is also specified, which takes priority over this setting then).


- `ESP_IDF_GLOBAL_INSTALL` (*native* only):

    If set to `1`, `true`, `y` or `yes` uses the global install directory (that is, unless `ESP_IDF_INSTALL_DIR` is also specified, which takes priority over this setting then).


- `ESP_IDF_VERSION` (*native* only):

  The version used for the `esp-idf` can be one of the following:
  - `commit:<hash>`: Uses the commit `<hash>` of the `esp-idf` repository.
                     Note that this will clone the whole `esp-idf` not just one commit.
  - `tag:<tag>`: Uses the tag `<tag>` of the `esp-idf` repository.
  - `branch:<branch>`: Uses the branch `<branch>` of the `esp-idf` repository.
  - `v<major>.<minor>` or `<major>.<minor>`: Uses the tag `v<major>.<minor>` of the `esp-idf` repository.
  - `<branch>`: Uses the branch `<branch>` of the `esp-idf` repository.

  It defaults to `v4.3.1`.


- `ESP_IDF_REPOSITORY` (*native* only): 

  The URL to the git repository of the `esp-idf`, defaults to <https://github.com/espressif/esp-idf.git>.
  
  Note that when the `pio` builder is used, it is possibnle to achieve something similar to `ESP_IDF_VERSION` and `ESP_IDF_REPOSITORY` by using 
  the [`platform_packages`](https://docs.platformio.org/en/latest/projectconf/section_env_platform.html#platform-packages) PlatformIO option as follows:
    - `ESP_IDF_PIO_CONF="platform_packages = framework-espidf @ <git-url> [@ <git-branch>]"`
    - The above approach however has the restriction that PlatformIO will always use the ESP-IDF build tooling from its own ESP-IDF distribution, 
      so the user-provided ESP-IDF branch may or may not compile. The current PlatformIO tooling is suitable for compiling ESP-IDF branches derived from versions 4.3.X .


- `ESP_IDF_GLOB[_XXX]_BASE` and `ESP_IDF_GLOB[_XXX]_YYY` (*native* and *pio*):

  A pair of environment variable prefixes that enable copying files and directory trees that match a certain glob mask into the native C project used for building the ESP-IDF framework:
  - `ESP_IDF_GLOB[_XXX]_BASE` specifies the base directory which will be glob-ed for resources to be copied
  - `ESP_IDF_GLOB[_XXX]_BASE_YYY` specifies one or more environment variables that represent the glob masks of resources to be searched for and copied, using the directory designated by the `ESP_IDF_GLOB[_XXX]_BASE` environment variable as the root. For example, if the follwing variables are specified:
    - `ESP_IDF_GLOB_HOMEDIR_BASE=/home/someuser`
    - `ESP_IDF_GLOB_HOMEDIR_FOO=foo*`
    - `ESP_IDF_GLOB_HOMEDIR_BAR=bar*`
    ... then all files and directories matching 'foo*' or 'bar*' from the home directory of the user will be copied in theESP-IDF C project.

    Note also that `_HOMEDIR` in the above example is optional, and is just a mechanism allowing the user to specify more than base directory and its glob patterns.


- `ESP_IDF_PIO_CONF_XXX` (*pio* only):

  A PlatformIO setting (or multiple settings separated by a newline) that will be passed as-is to the `platformio.ini` file of the C project that compiles the ESP-IDF.
  - Check [the PlatformIO documentation](https://docs.platformio.org/en/latest/projectconf/index.html) for more information as to what settings you can pass via this variable.
  - Note also that this is not one variable - but rather - a family of variables all starting with `ESP_IDF_PIO_CONF_`. I.e., passing `ESP_IDF_PIO_CONF_1` as well as `ESP_IDF_PIO_CONF_FOO` is valid and all such variables will be honored


- `MCU` (*native* and *pio*):

   The MCU name (i.e. `esp32`, `esp32s2`, `esp32s3` `esp32c3` and `esp32h2`). 
   
   - If not set this will be automatically detected from the cargo target.
   
   - Note that [older ESP-IDF versions might not support all MCUs from above](https://github.com/espressif/esp-idf#esp-idf-release-and-soc-compatibility).

## More info

If you are interested how it all works under the hood, check the [build.rs](build/build.rs)
build script of this crate.
