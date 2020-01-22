{% import "views/_helper.njk" as docs %}

# Command-Line Interface Guide

The command-line interface (CLI) can be used to deploy, publish, or revert your project. It can also be used to publish the same project to multiple applications, retrieve logs, and batch-upload files to the cloud.

## Installation

### macOS

You can install the CLI with [Homebrew](https://brew.sh):

```sh
brew update
brew install lean-cli
```

### Windows

You can go to our [releases page](https://releases.leanapp.cn/#/leancloud/lean-cli/releases) and download the 32-bit or 64-bit **msi** file to install. After installing, you can run `lean` under any directory in Command Prompt or PowerShell to use the CLI.

You can also download the pre-compiled **exe** file, rename it to `lean.exe`, and add it into **PATH** environment variable (instructions available [here](https://www.java.com/en/download/help/path.xml)). After doing this, you can run `lean` under any directory in Command Prompt or PowerShell to use the CLI. As an alternative, you can move the file into any directory that is already declared in PATH, like `C:\Windows\System32`.

### Linux

For Debian-based Linux distributions, you can download and install the Debian package `lean-cli-x64.deb` from our [releases page](https://releases.leanapp.cn/#/leancloud/lean-cli/releases).

For other distributions, download the pre-compiled file `lean-linux-x64` from our [releases page](https://releases.leanapp.cn/#/leancloud/lean-cli/releases), rename it to `lean`, and move it into any directory that is already declared in PATH.

#### Arch Linux

You can install the CLI from this AUR: https://aur.archlinux.org/packages/lean-cli-git/.

### Installing from Source Code

See [README](https://github.com/leancloud/lean-cli) of our GitHub repository.

### Upgrading

If the CLI is installed with Homebrew, run the following command to upgrade:

```sh
brew upgrade
```

Otherwise, refer to the instructions mentioned earlier to download the latest version of the file and overwrite the existing version on your computer.

## Usage

After installing the CLI, run `$ lean help` in terminal to see the help information:

```sh
 _                        ______ _                 _
| |                      / _____) |               | |
| |      ____ ____ ____ | /     | | ___  _   _  _ | |
| |     / _  ) _  |  _ \| |     | |/ _ \| | | |/ || |
| |____( (/ ( ( | | | | | \_____| | |_| | |_| ( (_| |
|_______)____)_||_|_| |_|\______)_|\___/ \____|\____|
NAME:
   lean - Command line to manage and deploy LeanCloud apps

USAGE:
   lean [global options] command [command options] [arguments...]

VERSION:
   0.21.0

COMMANDS:
     login    Log in to LeanCloud
     metric   Obtain LeanStorage performance metrics of current project
     info     Show information about the current user and app
     up       Start a development instance locally
     init     Initialize a LeanEngine project
     switch   Change the associated LeanCloud app
     deploy   Deploy the project to LeanEngine
     publish  Publish code from staging to production
     upload   Upload files to the current application (available in the '_File' class)
     logs     Show LeanEngine logs
     debug    Start the debug console without running the project
     env      Output environment variables used by the current project
     cache    LeanCache shell
     cql      Start CQL interactive mode
     help, h  Show all commands or help info for one command

GLOBAL OPTIONS:
   --version, -v  print the version
```

You can check the version of the CLI installed with `--version` option:

```sh
$ lean --version
lean version 0.21.0
```

To look up help information for a specific command, run `lean command -h`. For example:

 ```sh
 NAME:
    lean login - Log in to LeanCloud

 USAGE:
    lean login [command options] [-u username -p password (--region <CN> | <US> | <TAB>)]

 OPTIONS:
    --username value, -u value  Username
    --password value, -p value  Password
    --region value, -r value    The LeanCloud region to log in to (e.g., US, CN)
 ```

We will use `$ lean` in the rest of this documentation to indicate a command that can be run in terminal.

## Logging in

The first thing you need to do after installing the CLI is to log in to your LeanCloud account:

```sh
$ lean login
```

Follow the prompts to finish the login process.

### Switching Accounts

You can run `$ lean login` at any time to switch to a different account.

## Initializing a Project

After logging in, you can initialize a project and connect it to a LeanCloud application by running `$ lean init`:

```sh
[?] Please select an app: 
 1) AwesomeApp
 2) Foobar
```

Select the language and framework for the project:

```sh
[?] Please select a language
 1) Node.js
 2) Python
 3) Java
 4) PHP
 5) Others
```

The CLI will then download the template of the project:

```sh
[INFO] Downloading templates 6.33 KiB / 6.33 KiB [==================] 100.00% 0s
[INFO] Creating project...
```

After this, you will see a new directory with the name of your application. This directory contains the project you just created.

## Connecting to an Existing Project

If you already have a project in your local computer, you can run the following command to connect it to a LeanCloud application:

```sh
$ lean switch
```

## Switching Groups

If you have multiple groups enabled on LeanEngine, you can also use `$ lean switch` to switch the group connected to the current project.

## Running Locally

{% call docs.noteWrap() %}
You may skip this section if you plan to run your project directly in the cloud.
{% endcall %}

To run your project locally, first go inside its directory:

```sh
$ cd AwesomeApp
```

Now you need to install dependencies for it. Check the documentation of your project's language to get more instructions:

- [Node.js](leanengine_webhosting_guide-node.html#running-and-debugging-locally)
- [Python](leanengine_webhosting_guide-python.html#running-and-debugging-locally)
- [PHP](leanengine_webhosting_guide-php.html#running-and-debugging-locally)
- [Java](leanengine_webhosting_guide-java.html#running-and-debugging-locally)

After dependencies are installed, you are good to run your project:

```sh
$ lean up
```

- Open http://localhost:3000 in your browser to view the homepage of your project.
- Open http://localhost:3001 in your browser to debug cloud functions and hooks.

{% call docs.noteWrap() %}
You can specify the port your project uses with `$ lean up --port new-port-number`.
{% endcall %}

{% call docs.noteWrap() %}
Attach `-h` in the end of any command of the CLI to get help information (like `$ lean up -h`).
{% endcall %}

Besides using the CLI, you can also run a project **natively** by using commands like `node server.js` or `python wsgi.py`, which helps you better integrate LeanEngine into your existing workflows and IDEs. Keep in mind that projects created with CLI depend on several environment variables, so make sure they are configured correctly.

You can get a list of environment variables mentioned above by running `$ lean env`. By setting them up manually in the terminal, you will not be required to run your project with the CLI anymore. If your shell supports `sh`, you can run `eval $(lean env)` to get this done.

`$ lean up` supports pass-through custom start-up parameters. All parameters after `--` will be passed to the actual start-up command. For example, if you want to pass `--inspect` to the start-up command of a Node.js project to enable the remote debugging function, you can run `$ lean up -- --inspect`.

You can also use your own start-up command with `--cmd`, like `$ lean up --cmd=my-custom-command`.

Sometimes you may want to run your project within an IDE or test cloud functions in a virtual machine or a remote computer. You can use the following command to run only the debugging function rather than the entire project:

```sh
$ lean debug --remote=http://remote-url-or-ip-address:remote-port --app-id=xxxxxx
```

For more information regarding LeanEngine, see [LeanEngine Overview](leanengine_overview.html).

## Deploying

### Deploying from Local Computer

After testing your project locally, you can deploy it to LeanEngine with the following command:

```sh
$ lean deploy
```

If you only have a *trial instance* in the production environment, the command above will deploy the project directly to the **production environment** and override the previous version in it (no matter if you are deploying from local computer, deploying from Git repository, or editing cloud functions on the web console). If you have a *standard instance* in the production environment, the command above will deploy the project to the **staging environment** first. You can run `$ lean publish` later to publish your project in the production environment.

The progress will be printed out while the project is being deployed:

```sh
$ lean deploy
[INFO] Current CLI tool version:  0.20.1
[INFO] Retrieving app info ...
[INFO] Preparing to deploy AwesomeApp(xxxxxx) to region: us group: web production
[INFO] Node.js runtime detected
[INFO] Uploading file 52.10 KiB / 52.10 KiB [=======] 100.00% 30s
[REMOTE] Building 20190227-062423
[REMOTE] Downloading source code ...
[REMOTE] Unpacking source code ...
[REMOTE] Runtime: nodejs
[REMOTE] Downloading and installing dependencies ...
[REMOTE] Storing version to registry(62.35MB) ...
[REMOTE] [Node.js] Using Node.js v10.15.0, Node SDK 3.3.2, JavaScript SDK 3.11.1
[REMOTE] Version 20190227-062423 build finished
[REMOTE] Deploying 20190227-062423 to web1
[REMOTE] Creating new instance ...
[REMOTE] Starting new instance ...
[REMOTE] Instance started: {"runtime":"nodejs-v10.15.0","version":"3.3.2"}
[REMOTE] Updating cloud functions metadata ...
[REMOTE] Deploy finished: 1 instances deployed
[INFO] Deleting temporary files
```

The default message for deployment is `Creating from the CLI` which will be displayed in your app's [Dashboard > LeanEngine > App logs](https://console.leancloud.app/cloud.html?appid={{appid}}#/log). You can customize this message with `-m`:

```sh
$ lean deploy -m 'Be more awesome!'
```

After deploying, you can start testing cloud functions with `curl` or visit the homepage of the project via `stg-${App Domain}.avosapps.us`.

#### Ignoring Certain Files when Deploying

If your project directory contains temporary files that do not need to be uploaded to the cloud, you can add them into `.leanignore`.

`.leanignore` shares a similar format with `.gitignore` (the syntax for `.leanignore` is actually a subset of that for `.gitignore`), with each single line as a file or directory to be ignored. If your project does not contain a `.leanignore`, the CLI will automatically create one [according to the language used for the project](https://github.com/leancloud/lean-cli/blob/master/runtimes/ignorefiles.go#L13). Make sure to check if the content in the file meets the need of your project.

### Deploying from Git Repository

If your project is hosted on a Git platform (like [GitHub](https://github.com/)) and you have already configured the Git repository and deploy key on the web console, you can run the following command to have your project deployed with the source code in the repository:

```sh
$ lean deploy -g
```

- `-g` means to deploy from Git repository. Make sure it is already set up on the web console.
- The command will use the latest commit on **master** branch by default. You can specify the commit or branch by attaching `-r <revision>`.
- See [Node.js Web Hosting Guide > Deploying from Git Repository](leanengine_webhosting_guide-node.html#deploying-from-git-repository) for instructions on how to set up Git repository and deploy key.

## Publishing

{% call docs.noteWrap() %}
The instructions below only works if you have a [standard instance](leanengine_plan.html#standard-instance).
{% endcall %}

If you have finished testing your project in the staging environment, you can publish it to the production environment by going to your app's [Dashboard > LeanEngine > Deploy](https://console.leancloud.app/cloud.html?appid={{appid}}#/deploy) or by running the following command:

```sh
$ lean publish
```

You can view the progress in your terminal:

```sh
$ lean publish
[INFO] Current CLI tool version:  0.20.0
[INFO] Retrieving app info ...
[INFO] Deploying AwesomeApp(xxxxxx) to region: us group: web production
[REMOTE] Deploying 20181207-115634 to web1,web2
[REMOTE] Creating new instance ...
[REMOTE] Creating new instance ...
[REMOTE] Starting new instance ...
[REMOTE] Starting new instance ...
[REMOTE] Instance started: {"version": "2.1.8", "runtime": "cpython-3.7.1"}
[REMOTE] Instance started: {"version": "2.1.8", "runtime": "cpython-3.7.1"}
[REMOTE] Updating cloud functions metadata ...
[REMOTE] Deploy finished: 2 instances deployed
```

## Viewing Logs

You can view the latest logs of LeanEngine with `logs`:

```sh
$ lean logs
      2019-11-20 17:17:12  Deploying 20191120-171431 to web1
      2019-11-20 17:17:12  Creating new instance ...
      2019-11-20 17:17:22  Starting new instance ...
web1  2019-11-20 17:17:22  
web1  2019-11-20 17:17:22  > node-js-getting-started@1.0.0 start /home/leanengine/app
web1  2019-11-20 17:17:22  > node server.js
web1  2019-11-20 17:17:22  
web1  2019-11-20 17:17:23  Node app is running on port: 3000
      2019-11-20 17:17:23  Instance started: {"runtime":"nodejs-v12.13.1","version":"3.4.0"}
      2019-11-20 17:17:23  Updating cloud functions metadata ...
      2019-11-20 17:17:23  Deploy finished: 1 instances deployed
```

The command will return 30 entries by default, with the latest ones on the bottom.

You can specify the number of entries returned by using the `-l` option. For example, to return the latest 100 entries:

```sh
$ lean logs -l 100
```

You can also use the `-f` option to have logs fetched continuously (similar to using `tail -f`):

```sh
$ lean logs -f
```

If you want to fetch the logs generated within a certain period of time, you can provide a `--from` and `--to` parameter:

```sh
$ lean logs --from=2017-07-01 --to=2017-07-07
```

If you want to fetch the logs for a single day, you can just use `--from`:

```sh
$ lean logs --from=2017-07-01
```

If you prefer viewing logs with a different tool in your computer, you can export the logs into a file in the format of JSON:

```sh
$ lean logs --from=2017-07-01 --to=2017-07-07 --format=json > leanengine.logs
```

`--from` and `--to` use local timezone (the timezone on the machine executing `lean-cli`).

## Viewing LeanStorage Status Reports

You can get a report of LeanStorage status with `$ lean metric`:

```sh
$ lean metric --from 2017-09-07
[INFO] Retrieving xxxxxx storage report
Date                 2017-09-07   2017-09-08   2017-09-09
API Requests         49           35           14
Max Concurrent       2            2            2
Mean Concurrent      1            1            1
Exceed Time          0            0            0
Max QPS              5            5            5
Mean Duration Time   9ms          21ms         7ms
80% Duration Time    15ms         22ms         9ms
95% Duration Time    26ms         110ms        25ms
```

Item | Description
--- | ---
`Date` | Date.
`API Requests` | API requests made.
`Max Concurrent` | Maximum number of threads.
`Mean Concurrent` | Average number of threads.
`Exceed Time` | Number of requests that exceeded the limit.
`Max QPS` | Maximum QPS.
`Mean Duration Time` | Average response time.
`80% Duration Time` | 80% response time.
`95% Duration Time` | 95% response time.

`metric` takes in similar parameters as `logs`:

```sh
$ lean metric -h
NAME:
   lean metric - Obtain LeanStorage performance metrics of current project

USAGE:
   lean metric [command options] [--from fromTime --to toTime --format default|json]

OPTIONS:
   --from value    Start date, formatted as YYYY-MM-DD，e.g., 1926-08-17
   --to value      End date formated as YYYY-MM-DD，e.g., 1926-08-17
   --format value  Output format，'default' or 'json'
```

## Managing Multiple Applications

The source code of the same project can be deployed to multiple applications.

### Viewing the Current Application

You can see the application connected to the current project by running `$ lean info`:

```sh
$ lean info
[INFO] Retrieving user info from region: cn
[INFO] Retrieving app info ...
[INFO] Current region:  cn User: lan (lan@leancloud.rocks)
[INFO] Current region: cn App: AwesomeApp (xxxxxx)
[INFO] Current group: web
```

If you run `deploy`, `publish`, or `logs` at this time, these commands will be applied to the application shown above.

### Switching Applications

If you want to connect the current project to another application, you can use `$ lean switch`:

```sh
$ lean switch
```

You will see a list of applications that you can switch to.

You can also run `$ lean switch the-id-of-another-app` to quickly switch to another application.

## Uploading Files

You can use `$ lean upload` to upload a file or all the files in a directory (including those in subdirectories) to the cloud:

```sh
$ lean upload public/index.html
Uploads /Users/dennis/programming/avos/new_app/public/index.html successfully at: http://ac-7104en0u.qiniudn.com/f9e13e69-10a2-1742-5e5a-8e71de75b9fc.html
```

A URL (the part after `successfully at:`) will be generated after a files is successfully uploaded.

To upload all the files under `images`:

```sh
$ lean upload images/
```

## LeanCache

You can perform CRUD operations to your LeanCache instances with the CLI.

If you have enabled LeanCache for your application, you can run `$ lean cache` to connect to the LeanCache instance. Keep in mind that each LeanCache instance is divided into 16 `db`s by default and `db0` will be used by default.

After the connection is built, you can run commands directly in your terminal to perform data operations. For example, to see the value of a key:

```sh
LeanCache (db 0) > GET foo
"bar"
```

Since LeanCache is based on Redis, most commands available for Redis can also be used on LeanCache. See https://redis.io/commands for more instructions.

You can get a list of LeanCache instances of the current application with the following command:

```sh
$ lean cache list
```

Keep in mind that the CLI interacts with LeanCache through HTTPS, which means that commands demanding persistent connections (including `pub/sub` and `blpop`) cannot be used here. You can still use such commands through other channels.

### Custom Commands

If you have certain operations that need to be performed frequently (like checking the total amount of records in `_User`), you can define your own commands.

Simply create an executable file with its name starting with `lean-` (like `lean-usercount`) and put it into a directory included in `PATH` or `.leancloud/bin` under the project directory. After this, you will be able to run the file with `$ lean usercount`. Comparing to running `$ lean-usercount`, it gains the ability to access the environment variables related to each application.

Below are the environment variables available:

Variable Name | Description
--- | ---
`LEANCLOUD_APP_ID` | The App ID of the current application.
`LEANCLOUD_APP_KEY` | The App Key of the current application.
`LEANCLOUD_APP_MASTER_KEY` | The Master Key of the current application.
`LEANCLOUD_APP_HOOK_KEY` | The Hook Key of the current application.
`LEANCLOUD_APP_PORT` | The default port used to run the project with `$ lean up`.
`LEANCLOUD_API_SERVER` | The host providing API services to the current application.
`LEANCLOUD_REGION` | The region that the application belongs to, which could be `us`, `cn`, or `tab`.

If you create a file with the following script and put it into a directory included in `PATH` (like `/usr/local/bin`):

```python
#! /bin/env python

import sys

import leancloud

app_id = os.environ['LEANCLOUD_APP_ID']
master_key = os.environ['LEANCLOUD_APP_MASTER_KEY']

leancloud.init(app_id, master_key=master_key)
print(leancloud.User.query.count())
```

Grant the execute permission to it by running `$ chmod +x /usr/local/bin/lean-usercount`, and then run `$ lean usercount` under the project directory. Now you will see the total amount of records in `_User`.
