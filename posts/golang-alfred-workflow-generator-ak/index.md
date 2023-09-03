# Golang Alfred Workflow Generator Ak


<!--more-->


As loyal users of Gopher and Alfred workflows, we often write Alfred workflows in our daily lives to improve and speed up our workflow. Due to the nature of the Golang language, we encounter some issue when developing Alfred workflows using the Golang language.

1. Go is a binary language, so you need to compile it into a binary before you can use it when developing, and you need to package the binary into `.alfreworflow` when publishing. Here we will encounter binary security issues, so we need to code sign and notarize our binary to pass macOS's Gatekeeper check.

1. The development directory of the Alfred workflow needs to be symbolic link to `Alfred.alfredpreferences`, so that our workflow can communicate with Alfred.

Earlier in the development of Alfred workflow, I used Python's [workflow-install.py](https://gist.github.com/cage1016/a633149f4672fb40ecb4e16e04664ff7) to do the related Alfred interactive But it wasn't very easy to use, so I needed a better solution. So there is this project [cage1016/ak: A generator for golang alfred workflow that helps you create boilerplate code.](https://github.com/cage1016/ak)

## Features

1. Quickly create boilerplate code for Golang Alfred workflow projects
    - Use [deanishe/awgo](https://github.com/deanishe/awgo) and [spf13/cobra](https://github.com/spf13/cobra) as the development framework for Alfred workflow
    - Create two sets of Github Action release processes (automatic update version and Alfred Gallery release compatible version), and support Code Sign and Notarization
    - Create Makefile to simplify the development process
    - Create LICENSE
1. Workflow development environment management
    - Use `make link` to symbolic link the development directory of the workflow to `Alfred.alfredpreferences`
    - Use `make unlink` to unlink the development directory of the workflow to `Alfred.alfredpreferences`
    - Use `make build` to build the development directory of the workflow to `Alfred.alfredpreferences`
    - Use `make info` to display the information of the development directory of the workflow to `Alfred.alfredpreferences`
    - Use `make pack` to package the development directory of the workflow to `Alfred.alfredpreferences`
1. Support `arm64` & `amd64`

## Install

Install with Go

```bash
go install github.com/cage1016/ak@latest
```

## Usage

1. Create a project directory
1. Run `ak` to create the `ak.json` file and set the subsequent project information
   ```json
   {
       "go_mod_package": "github.com/xxx/ak-test",
       "workflow": {
           "folder": ".workflow",
           "name": "Ak Test",
           "description": "",
           "category_comment": "category: Tools, Internet, Productivity, Uncategorised",
           "category": "",
           "bundle_id": "com.xxx.aktest",
           "created_by": "",
           "web_address": "https://github.com/xxx/ak-test",
           "version": "0.1.0"
       },
       "update": {
           "github_repo": "https://github.com/xxx/ak-test"
       },
       "license": {
           "type_comment": "support license https://github.com/nishanths/license",
           "type": "mit",
           "year": "",
           "name": ""
       },
       "gon": {
           "application_identity": ""
       }
   }
   ```
1. Run `ak init` to initialize the project boilerplate code
    - `.workflow` directory is the development directory of Alfred workflow
    - `.gitingore` is the git ignore file
    - `.env` is the environment variable file
1. Run `ak new cmd` to create a command based on the Cobra CLI
1. Run `ak add ga` to create the Github Action release process
    - `-s` Enable Code Sign and Notarization features
    - `-c` Enable Go test Codecov release function
1. Run `ak add mf` to create Makefile development management tool
1. Run `ak add l` to create LICENSE file

Basically, you can start developing here, and if you need to, you can add more commands via `cobra add` to improve the workflow functionality. [ak/_example](https://github.com/cage1016/ak/tree/master/_example) Here's a complete example project.

## Development process

1. Run `go run main.go` to execute the workflow, which will provide the download of the go mod package and install it using `go get`
1. Run `make build` to build the development directory of the workflow to `Alfred.alfredpreferences`, which will put the compiled Binary `exe` in the `.workflow` directory at this time
1. Run `make link` to symbolic link the development directory of the workflow to `Alfred.alfredpreferences`

After completing the Alfred workflow symbolic link, we can see our workflow in Alfred workflow, and then we can start developing.

## Release process

1. Run `make pack` to package the development directory of the workflow as `.alfredworkflow` file
1. `ak add ga -s -c` to create two sets of Github Action release processes (automatic update version and Alfred Gallery release compatible version), and support Code Sign and Notarization. The release process will be triggered when the tag is pushed to the Github repository. Please refer to [Golang Base Alfred Code Sign and Notarize - KaiChu](https://kaichu.io/posts/golang-base-alfred-code-sign-and-notarize/) to learn how to use Code Sign and Notarization with Github action.

## Summary

This project is my development of Alfred workflow, in order to solve the problem of the development process and release process and developed, I hope that this project can help you, if you have any questions welcome to [cage1016/ak: A generator for golang alfred workflow that helps you create boilerplate code.](https://github.com/cage1016/ak) to raise an issue.

## Source Code

[cage1016/ak: A generator for golang alfred workflow that helps you create boilerplate code.](https://github.com/cage1016/ak)
