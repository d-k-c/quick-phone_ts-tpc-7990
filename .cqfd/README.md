# What is cqfd ? #

cqfd provides a quick and convenient way to run commands in the current
directory, but within a Docker container defined in a per-project config
file. This becomes useful when building an application designed for
another Linux system, e.g. building a RHEL7 app when your workstation
runs on Ubuntu 16.04.

# Using cqfd #

## Getting started ##

Just follow these steps:

* Install cqfd (see below)
* Go into your project's directory
* Create a .cqfdrc file
* Put a Dockerfile and save it as .cqfd/docker/Dockerfile
* Run ``cqfd init``

Examples are available in the samples/ directory.

cqfd will use the provided Dockerfile to create a normalized runtime
build environment for your project.

## Using cqfd on a daily basis ##

### Regular builds ###

To build your project from the configured build environment with the
default build command as configured in .cqfdrc, use:

    $ cqfd

Alternatively, you may want to specify a custom build command to be
executed from inside the build container.

    $ cqfd run make clean
    $ cqfd run "make linux-dirclean && make foobar-dirclean"

When ``cqfd`` is running, the current directory is mounted by Docker
as a volume. As a result, all the build artefacts generated inside the
container are still accessible in this directory after the container
has been stopped and removed.

### Release ###

The release command behaves exactly like run, but creates a release
tarball for your project additionally. The release files (as specified
in your ``.cqfdrc``) will be included inside the release archive.

    $ cqfd release

The resulting release file is then called unique job name, or the string
"local-build" when run from outside Jenkins, and BUILD\_ID is a
Jenkins-generated unique and date-based string, or the current date.

### Flavors ###

You may also want to build a specific build flavor, for a regular build
or a release. To do so use the -b option, for example:

    $ cqfd -b debug run

When building with a flavor, as when building a regular project, the
``run`` option can be omitted.

## The .cqfdrc file ##

The .cqfdrc file at the root of your project contains the information
required to support project tooling. samples/dot-cqfdrc is an example.

Here is a sample .cqfdrc file:

    [project]
    org='fooinc'
    name='buildroot'

    [build]
    command='make foobar_defconfig && make && asciidoc README.FOOINC'
    files='README.FOOINC output/images/sdcard.img'
    archive='cqfd-%Gh.tar.xz'

### The [project] section ###

``org``: a short, lowercase name for the project’s parent organization.

``name``: a short, lowercase name for the project.

Generated Docker images for your project will be named $org_$name.

### The [build] section ###

``command``: the command (or list of commands) to be executed when cqfd is
invoked. This string will be passed as an argument to a classical
``bash -c "commands"``, within the build container, to generate the
build artefacts.

``files``: an optional space-separated list of files generated by the
build process that we want to include inside a standard release
archive.

``archive``: the optional name of the release archive generated by
cqfd. You can include environment variable names, as well as the
following template marks:

* ``%Gh`` - git short hash of last commit
* ``%GH`` - git long hash of last commit
* ``%D3`` - RFC3339 date (YYYY-MM-DD)
* ``%Cf`` - current cqfd flavor name (if any)
* ``%Po`` - value of the ``project.org`` configuration key
* ``%Pn`` - value of the ``project.name`` configuration key
* ``%%`` - a litteral '%' sign

By default, cqfd will generate a release tarball named
org-name.tar.xz, where 'org' and 'name' come from the project's
configuration keys.

flavors: the list of build flavors (see below). Each flavor has its
own command just like build.command.

### Using build flavors ###

In some cases, it may be desirable to build the project using
variations of the build and release methods (for example a debug
build). This is made possible in cqfd with the build flavors feature.

In the .cqfdrc file, one or more flavors may be listed in the
``[build]`` section, referencing other sections named following
flavor's name.

    [debug]
    command='make DEBUG=1'
    files='myprogram Symbols.map'

    [build]
    command='make'
    files='myprogram'
    flavors='debug'

A flavor will typically redefine the keys of the build section:
command, files, archive.

### Environment variables ###

The following environment variables are supported by cqfd to provide
the user with extra flexibility during his day-to-day development
tasks:

``CQFD_EXTRA_VOLUMES``: A space-separated list of additional volume
mappings to be configured inside the started container. Format is the
same as (and passed to) docker-run’s -v option.

### Other command-line options ###

In some conditions you may want to use an alternate config file with
cqfd. This is what the -f option is for:

    $ cqfd -f .cqfdrc.test

## Build Container Environment ##

When cqfd runs, a docker container is launched as the environment in
which to run the *command*.  Within this environment, commands are
run as the 'builder' user.  So that this user has access to local
files, the current working directory is mapped to ~builder/src/.

### SSH Handling ###

The local ~/.ssh directory is mapped to the corresponding directory in
the build container i.e. ~builder/.ssh.  This effectively enables SSH agent
forwarding so a build can, for example, pull authenticated git repos.

Note that it may be helpful to specify the local user name in the
.ssh/config file as this isn't the default on the builder e.g.

	$ echo "User $USER" >> ~/.ssh/config

## Requirements ##

To use cqfd, ensure the following requirements are satisfied on your
workstation:

-  Bash 4.x

-  Docker

-  A ``docker`` group in your /etc/group

-  Your username is a member of the ``docker`` group

-  Restart your docker service if you needed to create the group.

## Installing/removing cqfd ##

The cqfd script can be installed system-wide.

Install or remove the script and its resources:

    $ make install
    $ make DESTDIR=/usr install
    $ make uninstall

## Testing cqfd (for developers) ##

The codebase contains tests which can be invoked using the following
command, if the above requirements are met on the system:

    $ make tests

## Trivia ##

CQFD stands for "Ce qu'il fallait Dockeriser", french for "what needed
to be dockerized".