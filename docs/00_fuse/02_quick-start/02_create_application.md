---
title: 'Create application'
id: create-application
---

# Create application

To simplify getting started, [GenX CLI](/fuse/introduction/prerequisites/#genx-cli) tool can create  projects from a number of available seeds (application templates). We will use a seed tailored for Fuse in this guide.

> A project seed takes care of the initial file structure and dependencies, allowing you to focus on the task at hand.

## Expected Result
By the end of this step, we should have:
- created and configured a new project

This will allow us to start building application functionality.

## Using GenX CLI

:::important
> Install the [GenX CLI](/fuse/introduction/prerequisites/#genx-cli) before proceeding with the following steps.
:::

You are now ready to generate a new project.

### Choosing Project Type

From the terminal, run:

```shell
foundation-cli
```

If this is the first time running the CLI tool, you'll need to provide Artifactory credentials. 

> No credentials? See our [Pre-requisites](/fuse/introduction/prerequisites/)

```shell
? Genesis Username example.username
? Genesis Password **************
√ Logged into Genesis
```

:::tip
We persist details to help speed things up, so this won't happen every time.
:::


Select `create fuse application`:

```shell
? Please select an option: (Use arrow keys)
> create fuse application - Creates a unified low-code DSL application.
  create workspace - Generates a local workspace to use for your Genesis based apps.
  configure workspace - Configure a local workspace.
  create application - Generates a local application.
  configure application - Configure a local app.
```

### Configuration
We now want to configure our project. There are a number of fields to fill in to help us get the best possible start.


:::tip
When possible answers are presented in grey text, the capitalised option is the default. You can just hit **Enter** to select it. 
For example `y/N`: default here is 'N' (No).
:::

:::info
We will use `alpha` in our examples from now on, but you are free to choose any name. 
:::

Let's choose the target directory and name for our project:

```shell
? Create an app in current directory Yes
? App name alpha
```

Select the seed from which you'd like to create an application (select `Pre-release` option if working with Docker):

```shell
? App seed (Use arrow keys)
  Low-code Application
// highlight-next-line
> Low-code Application (Pre-release)
```

:::caution
If you have previously created an application with this name, you will be asked whether you want to overwrite it. Overwriting is not recommended - it's preferable to exit the wizard (`Ctrl + C`) and rename or remove the old application.

```shell
? Overwrite existing files (y/N)
```
:::

Application is now created and dependencies are installed. Our expected output should be:

```shell
✔ Create path alpha
✔ Create directory alpha
✔ Creating from seed 'Low-code Application'
ℹ Installing dependencies.
✔ Install success.
```

### Package and environment settings

We can now configure the NPM package scope (typically name of your organisation) and name (defaults to your chosen application name):

```shell
? NPM package scope (genesislcap)
? NPM package name (alpha)
```

You will then be able to choose whether to use Docker (highly recommended) or WSL/CentOS environment.

```shell
Use Docker (Y/n)
```

Next, you will be asked whether we want to configure a custom API host. Hit Enter unless you need to override the default.

:::info
Web front end will attempt to connect to your local Genesis server by default. If you want to connect to a remote server instead, choose `Yes` and specify a WebSocket URL.
:::

```shell
? (Optional) Override the default API Host URL (N/y)
```

Continue with the remaining prompts:

```shell
? Genesis Server version
? Auth Server version
? GPL version
? Kotlin version
? Group Id global.genesis
? Application Version 1.0.0-SNAPSHOT
```

Next, you will be able to set the login details:

```
? User Name
? Password
```

At this point, project will be updated based on your answers. You should see the following shortly:

```shell
✔ Configuring Seed
✔ Writing environment variables
ℹ Application created successfully! 🎉 Please open the application and follow the README to complete setup.
```

Now open your chosen IDE (e.g. IntelliJ) and locate the newly created project.

## Recap

Congratulations, the local project is now ready. You have:

- created a new Fuse project
- configured package, dependency and development environment settings