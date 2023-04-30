---
title: "Student CLI"
linkTitle: "Student CLI"
weight: 30
---

## Installation

In order to install and use the student `scioer` command line tool you must have `python 3.7` or newer installed on your system.
In addition to python you must also have [Docker](https://docs.docker.com/get-docker/) installed and running on your computer.

The easiest way to install the instructor command line tool is to install it from the [pypi](https://pypi.org/project/sci-oer/) python registry:

{{< card-code header="**Installation**" lang="bash" >}}
pip install sci-oer
{{< /card-code >}}

## Getting started

The student command line tool is a wrapper around the docker command line to make it easier and less error-prone to run and maintain several scioer course resources.

The `scioer --help` or the `scioer -h` commands can be used to display a help message for the tool and to list all the commands that can be used.
Each of the sub commands also accept the `--help` option to get more details about how they work.

### Setting up a new Course

The first step to using a course resource is to get the name of the image from the instructor of the course.
For our example we are going to be using the `scioer/oo-java:W23` resource.
Once you have the course that you want to setup you can run the `scioer config` command to setup the new course.
The command will prompt you for:
- the name of the course, you can call it whatever you want, the name will be normalized to remove some difficult characters (like spaces)
- the docker image, this will be the one that the instructor gave you `scioer/oo-java:W23`
- if you want to automatically check for new versions of the course resource when it starts, accepting the default of 'no' is probably fine here
- where the files for the course should be stored, by default this is on the desktop, but any directory can be used
- if you want to use the default ports, you should say yes to this unless you have a good reason not to. If you say no you will be prompted to specify each port that will be used yourself

And that's it, you now have the new course configured.

{{< card-code header="**Configuring a Course**" >}}
$ scioer config
What's the name of the course?: test-course
What docker image does the course use? [scioer/oo-java:W23]: scioer/oo-java:W23
Automatically fetch new versions [y/N]:
Where should the files for the course be stored? [/Users/marshall/Desktop/testcourse]:
Use the default ports [Y/n]:
{{< /card-code >}}


### Starting the Resource

To start a course run `scioer start <course name>`.
This will install/fetch the latest version of the course resource (if that configuration option was set).
It may take some time for the resource to start the first time.
If you only have one course configured you do not need to specify the course name, it will automatically start the only course in the config file.
If there is no course configured with the specified course name, then the names of all the configured courses will be printed and one can be selected interactively.


### Starting a shell

To start a shell in a running course resource use the `scioer shell <course name>` command.
If you only have one course configured you do not need to specify the course name, it will automatically get a shell in the only course in the config file.

### Stopping the Resource

To stop a running course resource use the `scioer stop <course name>` command.
If you only have one course configured you do not need to specify the course name, it will automatically stop the only course in the config file.

## Advanced Usage

You can check the status of the currently configured courses by using the `scioer status` command.
This will print the location of the configuration file, and the current state of all the course resources.

By default, the course resource will only publish its ports to be available from `localhost` on the host machine.
This was done for security of the resource to prevent it from accidentally be available to any other computer on the local network.
However, there are certain scenarios where this may be desirable, in these cases you can manually edit the configuration file, which can be found at `~/.scioer.yml` and set the value `public:` to `true`.
This change will take effect the next time the resource is started.


## Getting Help and Reporting Issues

If the command line tool is not behaving correctly more information about its status can be obtained by running the command with the
`--verbose` or the `--debug` options.
We recommend first re-running the command in `--verbose` mode as `debug` will produce _a lot_ of output and is probably best left for use when reporting an issue.

The best way to report an issue is to create a new [Github Issue](https://github.com/sci-oer/student-cli/issues/new?assignees=&labels=type%3A+bug&template=bug_report.md&title=) and be sure to include the command, debug output, a description of what went wrong, as well as a brief description about the expected behavior and how the system is set-up.
