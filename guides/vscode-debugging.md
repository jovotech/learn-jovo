# Visual Studio Code Debugging

- [Visual Studio Code Debugging](#visual-studio-code-debugging)
    - [Introduction](#introduction)
    - [Configuration](#configuration)
    - [Test the Debugger](#test-the-debugger)
    - [Community](#community)

## Introduction

Debugging your voice application can be quite tedious at times. Using a debugger can make a big difference in your development process and is pretty easy to set up. You will learn how in this short guide.

## Configuration

To use the Visual Studio Code debugger with Jovo you have to first install the [`Jovo CLI`](https://github.com/jovotech/jovo-cli) locally to access it in the debugger's configuration:

```sh
$ npm install jovo-cli --save-dev
```

After that have to configure VSCode's debugger using the `Open Configurations` button. You can find the option in the `Debug` tab on the menu bar. If it asks you to specify an environment select `Node.js`.

```javascript
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "program": "${workspaceFolder}\\index.js"
        }
    ]
}
```

Change the `program` attribute to `${workspaceFolder}/node_modules/jovo-cli/index` and add the `"args": ["run", "--inspect"]` attribute to pass arguments to the program. You also need the `autoAttachChildProcesses` to be set to `true`. At the end it should look like this:

```javascript
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Program",
            "program": "${workspaceFolder}/node_modules/jovo-cli/index",
            "args": ["run", "--inspect"],
            "autoAttachChildProcesses": true
          }
    ]
}
```

## Test the Debugger

To test the debugger add sample breakpoints to your project and simply press `F5` to start debugging. After that start your Alexa Skill or your Google Action and your app should stop at the breakpoints.

## Community

You can also check out the blogpost from [Ben Force](https://medium.com/@benforce/debugging-jovo-skills-in-vs-code-d117f908fbc2) about debugging Jovo applicatioins in VSCode.
