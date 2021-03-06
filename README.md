# Edward

[![Build Status](https://travis-ci.org/yext/edward.svg?branch=master)](https://travis-ci.org/yext/edward)
[![Go Report Card](https://goreportcard.com/badge/github.com/yext/edward)](https://goreportcard.com/report/github.com/yext/edward)

A command line tool for managing local instances of microservices.

## Installation

Edward uses the vendor folder, and as such, Go 1.6 is required.

    go get github.com/yext/edward

## Updating

To update an existing install to the latest version of Edward, run:

    go get -u github.com/yext/edward

## Usage

    NAME:
       Edward - Manage local microservices

    USAGE:
       edward [global options] command [command options] [arguments...]

    VERSION:
       1.6.1

    COMMANDS:
         list	List available services
         generate	Generate Edward config for a source tree
         status	Display service status
         messages	Show messages from services
         start	Build and launch a service
         stop	Stop a service
         restart	Rebuild and relaunch a service
         log, tail	Tail the log for a service

    GLOBAL OPTIONS:
       --config PATH, -c PATH	Use service configuration file at PATH
       --help, -h			show help
       --version, -v		print the version

## Running Services and Groups

Edward will attempt to autodetect Play, Java and Go projects under the current working directory (or the root of the current repo). If an edward.json config file is found in the CWD, repo or *~/.edward*, configuration for services and groups will be loaded from that instead.

For example, let's say your config has a group called *mygroup* which consolidates a few services required for a product you're working on:

To start the services necessary for running mygroup locally:

    edward start mygroup

This will build each service, before starting them in sequence. Failure in any build process will stop the command and nothing will launch. Failure in a start will stop further progress, but will not stop already running services

Once they are running, you can stop them with the command:

    edward stop mygroup

If you want to view the logs for a service in mygroup, say *myservice*:

    edward tail myservice

Note that you can only do this for a single service, `logmygroup`, for example, will cause an error.

## Generating and Modifying service/group configuration

The `generate` command will create a JSON file defining the detected services and hard-coded groups.

    edward generate

This file will be generated in the current working directory if no existing config file is found.

If this file exists when you run the Edward command, the config will be used to load services and groups. Feel free to add new groups to your config as you see fit!

Running `generate` when a config file already exists will attempt to autodetect any new services and add them to this config.

If you only want a certain set of services to be added to the config, specify these as a list in the arguments to `generate`, for example:

    edward generate service1 service2

Will add service1 and service2 to the config. Any other services below the current directory will be ignored.

## Autocomplete

To enable bash autocompletion, create a file with the following:

    #! /bin/bash

    : ${PROG:=$(basename ${BASH_SOURCE})}

    _cli_bash_autocomplete() {
         local cur opts base
         COMPREPLY=()
         cur="${COMP_WORDS[COMP_CWORD]}"
         opts=$( ${COMP_WORDS[@]:0:$COMP_CWORD} --generate-bash-completion )
         COMPREPLY=( $(compgen -W "${opts}" -- ${cur}) )
         return 0
     }

     complete -F _cli_bash_autocomplete $PROG

Then source it from your bash profile

    PROG=edward source FILE

Alternatively, name the file edward and place it in your system appropriate `bash_completion.d/` directory.

## sudo

Edward will not run if you try to launch it with sudo, but it may ask you to provide your password so that certain services can be run with elevated priviledges. The password request is triggered through a bash script that calls a command with sudo, to ensure that your bash session can make further sudo calls without prompting.

This has only been tested in one bash environment, so your mileage may vary. If services hang when starting (waiting for their log), this may be an indicator that they are waiting for a password prompt that isn't redirected anywhere.
