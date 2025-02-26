# Example Code: Group Coherent Presentation

If you haven't used the RTI Connext C# API before, first check the
[Getting Started Guide](https://community.rti.com/static/documentation/connext-dds/6.1.1/doc/manuals/connext_dds_professional/getting_started_guide/index.html).

## Building the Example :wrench:

First use **rtiddsgen** to generate the C# type and project files from
`GroupCoherentExample.idl`:

```sh
<install dir>/bin/rtiddsgen -language c# -platform net5 -create typefiles -create makefiles GroupCoherentExample.idl
```

Where `<install dir>` refers to your RTI Connext installation.

You will see messages that look like:

```plaintext
WARN com.rti.ndds.nddsgen.emitters.FileEmitter File exists and will not be
overwritten : /some/path/GroupCoherentExample.csproj
```

This is normal and is only informing you that some of the files that **rtiddsgen**
can generate were already available in the repository.

Then build it using the dotnet CLI:

```sh
dotnet build
```

## Running the Example

In two separate command prompt windows for the publisher and subscriber run the
following commands from the example directory (this is necessary to ensure the
application loads the QoS defined in *USER_QOS_PROFILES.xml*):

```sh
dotnet run -- --pub
dotnet run -- --sub
```

For the full list of arguments:

```sh
dotnet run -- -h
```
