# Recording Service -- Pluggable Storage

## Example Description

This example shows how to implement a custom *RTI Recording Service* Storage
plug-in, build it into a shared library and load it with *RTI Recording
Service*, *RTI Replay Service* or *RTI Converter*.

In this concrete example, we show how a simple storage plug-in written in C++.
The example is composed of two parts: the writing side (that can be plugged into
Recorder or Converter) and the reading side (that can be plugged into Replay or
Converter - although only a Replay configuration file is supplied).

The example works with one single type called `HelloMsg`. The IDL for the type
is supplied. Any other topics or types won't be recorded or replayed. The
purpose of this example is to show how the *Recorder* and *Replay* APIs in C++
can be used to plug-in custom storage needs. In this simple example, Recorder
will store the discovered samples in a file called Cpp_PluggableStorage.dat and
a companion file called Cpp_PluggableStorage.dat.info, by using the storage
plug-in. The samples are stored in a textual format. These samples are later
retrieved by Replay by using the reading plug-in.

The code in this directory provides the following components:

-   `FileStorageWriter` implements the writer API that is loaded by *RTI
    Recorder*. It is responsible for writing the received data samples to file.

-   `pluggable_storage_example.xml` file: defines a *Recorder* configuration
    that loads the shared library resulting from building this example.

-   `FileStorageReader` implements the reader API that is loaded by *RTI Replay*
    or *RTI Converter*. This component is responsible for reading data samples
    from the file.

-   `pluggable_replay_example.xml` file: defines a *Replay* configuration that
    loads the shared library resulting from building this example.

-   `HelloMsg.idl`: this IDL file contains a type definition. For example
    purposes, the storage writer and reader plugins only record and replay this
    type.

-   `HelloMsg_publisher.cxx` and `HelloMsg_subscriber.cxx`: these files use the
    code for the `HelloMsg` defined in the IDL file above generated by using
    *RTI DDS Code Generator* in the build process, and publish or subscribe
    samples of this type.

For more details on how to implement custom plugins, please refer to the *RTI
Recording Service API* documentation.

## Building the C++ example

In order to build this example, you need to provide the following variables to
`CMake`:

-   `CMAKE_BUILD_TYPE`: specifies the build mode. Valid values are Release
    and Debug.

-   `BUILD_SHARED_LIBS`: specifies the link mode. Valid values are ON for
    dynamic linking and OFF for static linking.

Build the example code by running the following command:

```bash
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON ..
cmake --build .
```

> **Note**:
>
> When using a multi-configuration generator, make sure you specify
> the `--config` parameter in your call to `cmake --build .`. In general,
> it's a good practice to always provide it.

In case you are using Windows x64, you have to add the option -A in the cmake
command as follow:

```bash
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON .. -A x64
```

**Cross-compilation**.

When you need to cross-compile the example, the above command will not work, the
assigned compiler won't be the cross-compiler and errors may happen when linking
against the cross-compiled Connext binaries. To fix this, you have to create a
file with the architecture name and call CMake with a specific flag called
``-DCMAKE_TOOLCHAIN_FILE``. An example of the file to create with the toolchain
settings (e.g. for an ARM architectures):

```cmake
set(CMAKE_SYSTEM_NAME Linux)
set(toolchain_path "<path to>/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian")
set(CMAKE_C_COMPILER "${toolchain_path}/bin/arm-linux-gnueabihf-gcc")
set(CMAKE_CXX_COMPILER "${toolchain_path}/bin/arm-linux-gnueabihf-g++")
```

Then you can call CMake like this:

```bash
cmake -DCONNEXTDDS_DIR=<connext dir> -DCMAKE_TOOLCHAIN_FILE=<toolchain file created above>
      -DCONNEXTDDS_ARCH=<connext architecture>
      -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON ..
```

## Running the C++ example (Recorder storage writer)

First of all, you need to make sure that the `FileStorageWriter` shared library
is in the library path.  On OS X, you must use the variable
`RTI_LD_LIBRARY_PATH` instead of the `DYLD_LIBRARY_PATH`.

Run the `HelloMsg` publisher application:

```bash
cd build
HelloMsg_publisher <your domain ID>
```

To run the example, you just need to run the following command from the `build`
folder (where the storage writer plugin shared library has been created).

Also, you have to export your architecture. This way the script can use the
specific target binary instead of using the standard host binary.

```bash
cd build
export CONNEXTDDS_ARCH=x64Linux3gcc5.4.0
<connext dir>/bin/rtirecordingservice -cfgFile ../pluggable_storage_example.xml
        -cfgName CppFileWriterExample -domainIdBase <your domain ID>
```

After running this command, you will see the following output:

```bash
RTI Recording Service (Recorder) 6.1.1 starting...
RTI Recording Service started
```

*Recorder* will create two files, `Cpp_PluggableStorage.dat` and
`Cpp_PluggableStorage.dat.info`. The `HelloMsg` recorded samples are in the *.dat*
file. The *.dat.info* file contains information about when the service started
and finished.

## Running the C++ example (Replay storage reader)

For *Replay* to have some data to replay, we assume that you have run the
storage writer example and recorded some data. We also assume that you correctly
set up the environment to find the `FileStorageReader` library, located in the
same place as the `FileStorageWriter` one.

If you haven't done it yet, stop the `HelloMsg` publisher application so it
doesn't interfere.

Now run the `HelloMsg` subscriber application like this:

```bash
cd build
HelloMsg_subscriber <your domain ID>
```

To run the example, you just need to run the following command from the `build`
folder (where the storage reader plugin shared library has been created).

```bash
cd build
<connext dir>/bin/rtireplayservice -cfgFile ../pluggable_replay_example.xml
        -cfgName CppFileReaderExample -domainIdBase <your domain ID>
```

You should see the samples in the file being published and received by the
subscribing application. *Note*: If you started Recording Service before
starting the publisher aplicationp, it may look like Replay is taking a long
time to start. This happens because the start time of the recording (written in
the `Cpp_PluggableStorage.dat.info` file) will be much earlier than the first
recorded sample. For this example, we recommend that you run the publisher
before you run *Recorder*, so the start time and the time of the first sample
will be similar.

## Customizing the Build

### Configuring Build Type and Generator

By default, CMake will generate build files using the most common generator for
your host platform (e.g., Makefiles on Unix-like systems and Visual Studio
solution on Windows), \. You can use the following CMake variables to modify the
default behavior:

-   `-DCMAKE_BUILD_TYPE` -- specifies the build mode. Valid values are Release
    and Debug. See the [CMake documentation for more details.
    (Optional)](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html)

-   `-DBUILD_SHARED_LIBS` -- specifies the link mode. Valid values are ON for
    dynamic linking and OFF for static linking. See [CMake documentation for
    more details.
    (Optional)](https://cmake.org/cmake/help/latest/variable/BUILD_SHARED_LIBS.html)

-   `-G` -- CMake generator. The generator is the native build system to use
    build the source code. All the valid values are described described in the
    CMake documentation [CMake Generators
    Section.](https://cmake.org/cmake/help/latest/manual/cmake-generators.7.html)

For example, to build a example in Debug/Static mode run CMake as follows:

```sh
cmake -DCMAKE_BUILD_TYPE=Debug -DBUILD_SHARED_LIBS=ON .. -G "Visual Studio 15 2017" -A x64
```

### Configuring Connext DDS Installation Path and Architecture

The CMake build infrastructure will try to guess the location of your Connext
DDS installation and the Connext DDS architecture based on the default settings
for your host platform.If you installed Connext DDS in a custom location, you
can use the CONNEXTDDS_DIR variable to indicate the path to your RTI Connext DDS
installation folder. For example:

```sh
cmake -DCONNEXTDDS_DIR=/home/rti/rti_connext_dds-x.y.z ..
```

Also, If you installed libraries for multiple target architecture on your system
(i.e., you installed more than one target rtipkg), you can use the
CONNEXTDDS_ARCH variable to indicate the architecture of the specific libraries
you want to link against. For example:

```sh
cmake -DCONNEXTDDS_ARCH=x64Linux3gcc5.4.0 ..
```
