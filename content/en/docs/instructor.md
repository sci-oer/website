---
title: "Instructor CLI"
linkTitle: "Instructor CLI"
weight: 20
---

{{% alert title="Stability Note" color="warning" %}}
The `scioer-builder` instructor command line tool is still in a beta phase and while we will do our best to avoid breaking
changes in its behavior and interface, it is not guaranteed on versions less than `1.0.0`.
{{% /alert %}}

## Installation

In order to install and use the instructor `scioer-builder` command line tool you must have `python 3.7` or newer installed on your system.
In addition to python you must also have [Docker](https://docs.docker.com/get-docker/) installed and running on your computer.
[Docker Buildx](https://docs.docker.com/build/install-buildx/) is optional dependency that is only required if you are planning on building multiple CPU architecture images. Buildx should be installed by default when Docker Desktop is installed or when the official Docker package is installed on Linux.

The easiest way to install the instructor command line tool is to install it from the [pypi](https://pypi.org/project/scioer-builder/) python registry:

{{< card-code header="**Installation**" lang="bash" >}}
pip install scioer-builder
{{< /card-code >}}

Alternatively the builder has also been published as a docker image [here](https://hub.docker.com/r/scioer/automated-builder).
Which can be installed using `docker pull scioer/automated-builder:latest`.

## Helpful Tips and Commands

Once the tool is installed the following are some helpful commands that can be used to build a custom course image.

- Get help with the tool `scioer-builder --help`
- Be prompted for the configuration values for a new course `scioer-builder --interactive`

Once all / most of your custom course content has been created, we would recommend that you run the first few builds with the following options, in addition to the ones to set the actual content:
- Do _not_ use the `--push` option, to ensure that the test versions don't get published yet
- Do _not_ use the `--multi-arch` option, to make the build run faster for testing
- Use `--tag` value of something like `test-course:latest` this should make it a little easier to differentiate the development versions from the real one that's ready to be published. This should also help prevent the docker registry from accepting your image if you accidentally set the `--publish` option
- Experiment with the different configuration options, specifically the `--motd-file` and the various wiki options.

## Configuration Options

The `scioer-builder` command supports a large number of configuration options.
Nearly all of them can be set either through flags that are passed to the command, _or_ through prompts when running in interactive mode.
There are no options that are required to be set in order for the tool to be used.

The main 4 components that can be configured are:
- The Jupyter Notebooks content
- Lectures content
- The Wiki content
- the resource itself

### Jupyter Content

The Jupyter content can be set from a git repository by using the `--jupyter-repo` option and specifying either the `https` or `ssh` url to clone the repository.
If the `ssh` option is used then the `--key-file` option must _also_ be specified with a path to an _unlocked_ ssh private key that has access to the repository.
If the `https` option is used then either the repository must be public, _or_ the credentials to get access must be encoded into the url.

Alternatively, the `--jupyter-directory` option can be used to load the notebooks from a local directory instead of from a remote git repository.
These two options cannot be used at the same time.

### Lectures Content

The lectures content can be loaded the same way as the [Jupyter Content](#jupyter-content), by using the `--lectures-repo` or `--lectures-directory` options.
However, loading the lectures content from a git repository has been deprecated because lecture files are typically too large to hold in a git repository.
There plans (tracked in [issue #81](https://github.com/sci-oer/automated-builder/issues/81)) to add support for loading lectures from other sources.

### Wiki Content

The wiki content has the most configuration options available that can be used to configure it.
The content must come from a git repository.
That repository can be loaded over `ssh` (using the same `--key-file` as the rest of the repositories), or over `https`.
If `https` is used credentials for the repository can be specified using the `--wiki-git-user` and `--wiki-git-password` options, but they are not needed if the repository is public.

The wiki also supports configuring additional attributes such as: a title, if comments are enabled, and the page navigation mode.

## The Resource

There are a number of options that can be used to configure the resource its self.
The main ones are `--base` which is used to specify the base image that the custom course should be built for.
Currently, we maintain the following images that can be used to make course resources from:
- [scioer/python-resource](https://hub.docker.com/r/scioer/python-resource)
- [scioer/java-resource](https://hub.docker.com/r/scioer/java-resource)
- [scioer/c-resource](https://hub.docker.com/r/scioer/c-resource)
- [scioer/mariadb-resource](https://hub.docker.com/r/scioer/mariadb-resource)

The default `--base` image is for the java resource.

The `--tag` option is used to specify the name of the image, including the docker tag for the image that will be built.
The default value is `sci-oer/custom:latest`, but you should make sure that the name you pick is in your namespace in the docker registry that is being used.
If you are just building for local testing and _not_ pushing to a remote registry then the name does not matter as much.

The `--push` option must be specified to actually push the built image to the registry once it has been built.
We recommend that you do not enable this option at first to make sure that the initial image you build is correct and behaves as expected.
In order to be able to push the image, you must already be authenticated with the registry by using the `docker login` command.

The auto builder supports building the custom resource for both `amd64` (intel / AMD) and `arm64` (arm / Apple Silicone).
By default, it will only build for the current CPU architecture, because it is faster when building to test the resource.
It can be built for both by specifying the `--multi-arch` option and having [Docker Buildx](https://docs.docker.com/build/install-buildx/) installed and enabled.

The full list of options that are available can be found by using the `scioer-builder --help` command.

## Examples

### Non-Interactive Mode

The following code block demonstrates an example command that will build a course resource for an object-oriented course in java.
It will set a custom course title, and will not include any lecture resources, instead the lectures will be served through a reverse proxy within that are hosted on example.com.

{{< card-code header="**Non-interactive Building**" lang="bash" >}}
scioer-builder \
  --jupyter-repo https://github.com/sci-oer/oo-course-tutorials.git  \
  --example https://github.com/sci-oer/oo-course-practice.git  \
  --wiki-git-repo https://github.com/sci-oer/oo-course-wiki.git \
  --wiki-title "Introduction to Object Oriented Programming with Java"  \
  --tag "scioer/oo-java:W23" \
  --base "scioer/java-resource:sha-62b706b" \
  --static-url "https://example.com"
{{< /card-code >}}

### Interactive Mode

In this next example several of the options will be set through the command line flags and some others can be set when promoted, such as the wiki title.
This example will add multiple examples that will be copied into the resources practice problem directories.

{{< card-code header="**Interactively set the title**" lang="bash" >}}
scioer-builder \
  --verbose \
  --interactive \
  --example-dir /data/example1 \
  --example-dir /data/example2 \
  --wiki-git-repo https://github.com/sci-oer/c-course-wiki.git \
  --tag "my-resource:latest" \
  --base "scioer/c-resource:sha-ba6af63"
....
## Information about the wiki to be created.
Enter the title for the Wiki: <Enter the title for the wiki here>
....
{{< /card-code >}}

### Using Docker

The following example uses the builder installed via docker and _not_ through `pip`.
When building a custom resource using the docker image there are a few extra steps that are required.
As you can see in the command bellow, when specifying the ssh key file that should be used, the path that is specific must be
the path in the container and _not_ the path to the ssh key on your host computer.
Additionally, all of the files that are used by the builder must be passed into the container as mounted volumes using the `-v` docker option.

{{< card-code header="**Using the Docker Image**" lang="bash" >}}
docker run -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /Users/marshall/.ssh/id_rsa:/id_rsa:ro \
  -v /Users/marshall/examples:/data \
  scioer/automated-builder:latest \
  --verbose \
  --key-file "/rd_rsa" \
  --example-dir /data/example1 \
  --example-dir /data/example2 \
  --wiki-git-repo https://github.com/sci-oer/c-course-wiki.git \
  --tag "my-resource:latest" \
  --base "scioer/c-resource:sha-ba6af63"
{{< /card-code >}}

## Getting Help and Reporting Issues

If the auto builder is not behaving correctly more information about its status can be obtained by running the command with the
`--verbose` or the `--debug` options.
These need to be specified on the command line and can not be enabled from within the interactive mode prompts.
We recommend first re-running the command in `--verbose` mode as `debug` will produce _a lot_ of output and is probably best left for use when reporting an issue.

The best way to report an issue is to create a new [Github Issue](https://github.com/sci-oer/automated-builder/issues/new?assignees=&labels=type%3A+bug&template=bug_report.md&title=) and be sure to include the command, debug output, a description of what went wrong, as well as a brief description about the expected behavior and how the system is set-up.
