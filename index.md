<!--
## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/TIGHEDEV/GitHub-Actions-Exam/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/TIGHEDEV/GitHub-Actions-Exam/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
-->

# GitHub Actions Revision Guide

## GitHub Actions Introduction

- GitHub Actions are packaged scripts in the form of a yml file that automate tasks in a workflow.
- GitHub Actions can be found in the marketplace or you can write one yourself.
- There are 2 types of Actions:
  - **Container Actions** - The environment is part of the action's code. They can only run in a Linux environment hosted by GitHub. They support many different languages
  - **JavaScript Actions** - The environment is not part of the action's code, it has to be specified. Can run in a VM and supports Linux, MacOS and Windows environments.

### GitHub Actions Components

- **Workflows** - Each workflow must contain at least 1 job. Add the workflow to the github/workflows directory. Can be triggered by many different events.
- **Jobs** - Part of the workflow that will be associated to a runner.
- **Steps** - An individual task that can run commands in a job.
- **Actions** - Standalone commands that are executed. Can reference other Actions or use commands like ```run: npm install -g bats```

### Events to run an Action

- Actions can be configured to run on a schedule. Configure it by using: ``` ***** ```. Each star stands for a different component of time. Minute/Hour/Day of the month/month/Day of the week.
- You can manually trigger a workflow by using the **workflow_dispatch** command.
- You can trigger a webhook event called **repository_dispatch**. You will trigger the workflow by making a POST request to the GitHub endpoint: **/repos/{owner}/{repo}/dispatches** with the event names in the request body.
- You can use conditional keywords by using the **if** statement. You will need to use ```${{ <expression> }}``` to tell GitHub to evaluate the statement as an expression rather than a string.
- If you want to disable a workflow so that you don't have to delete the workflow file then you can do that.

## Using GitHub Actions to implement CI

### Using a matrix

- If you want to test your action on multiple versions of Node then you will need to use a matrix. 
- It is good to separate out the tests and builds to make the logs more clear.
- Use the matrix command to create the matrix and then call it like a variable in the with command. 

### Artifacts

- Anything that is produced by a workflow, other than logs, is called an artifact. E.g creating a docker container.
- Uploading an artifact can be done using the **actions/upload-artifact** command, and downloaded from storage using the **actions/download-artifact** comnand.
- Storing an artifact allows you to preserve it between jobs. Each job uses a fresh VM so you need the artifact in storage to be able to call it in multiple jobs.

### Automating reviews

- If you want to trigger a workflow but onyl after someone has reviewed the pull request then you can do this with a trigger on **pull-request-review** or by adding a label to the pull request by using the **pullreminders/label-when-approved-action** action.
- When using the label action you can set the number of approvals needed and specify the GitHub token so that a change can be made by the action. This is done in the **env** section.

## Customise your workflow with environment variables and artifact data

### Default environment variables

- There are default enviornment variables available but they can only be used within the runner that is executing the job.
- Default variables are **case sensitive**
- To use them you need to sepecify the **$** sign followed by the environment variable name.
- Default variables are all **uppercase**

### Contexts

- Similar to environment variables but span the entire workflow.
- Can be used in an if statement prior to a runner being executed.
- Context variables are all **lowercase**.

### Custom environment variables

- To use them they need to be defined in your workflow file using the **env** context
- Can use the runner's OS usual way of reading variables to read them.

### Scripts

- The **run** keyword is used to run scripts and commands on the runner.
- You can store scripts in your repo, usually something like **.github/scripts/**
- Make sure to provide the path to the script and the shell it needs to run

### Cache dependancies

- To save time when reusing dependencies/outputs you can cache them so that your workflow is more efficient.
- You can cache using GitHub's cache action. The action works by retrieving a cache that you have identified by a unique key.
- The action returns the cached data to you via the path that you provide

### Getting detailed logs

- If the default logs don't provide enough detail then you can enable additional debug logging for runs and steps. 
- To enable runner diagnostic logging you need to set the **ACTIONS_RUNNER_DEBUG** secret in the workflow repository to **true**
- To enable step diagnostic logging you need to set the **ACTIONS_STEP_DEBUG** secret in the workflow repository to **true**
- You can access the logs from the UI and REST API: ```GET /repos/{owner}/{repo}/actions/runs/{run_id}/logs```

## Using GitHub Actions to implement CD

### Triggering a CD workflow

- You can trigger a workflow on a pull request that has specific labels by using the command **types: [labeled]**
- You can also trigger a workflow using the **if** condition, e.g. ```if: contains(github.event.pull_request.labels.*.name, 'stage')```

### Using secrets

- To deply to Azure the GitHub ACtion must have permission to access the resource, therefore you can store your Azure credentials in a GitHub secrets so that you are not storing it in plain text.
- Example: ```steps:
      - name: "Login via Azure CLI"
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}```
- Secrets can be added through the UI.

## Using GitHub Actions to publish to GitHub Packages

### What are packages?

- Packet management service.
- Allows you to share your project dependencies with your organisation or publicly.
- Compatible with NPM, NuGet, RubyGems, Maven and Gradle.
- You8 can also publish container images.
- Package permissions are assigned at the repository level.

## Publish to GitHub Packages

- You need to specify a **registry-url** and an **access token** for authentication when publishing. Plus your GitHub username
- A command like **mpn ci** gets the packages that you want.
- You can generate a PAT in your profiele settings
- When logging in to packages you need to authenticate with the endpoint. Depending on what packet manager you are using the endpoint will be different. For NPM it will look like this: ```bash npm login --registry=https://npm.pkg.github.com```. NuGet will look like this: ```dotnet nuget add source https://nuget.pkg.github.com/OWNER/index.json -n github -u OWNER -p [Your PAT Token]```
- It follows this pattern: ```https://PACKAGE_TYPE.pkg.github.com/OWNER/REPOSITORY```
- When  you update a package you can delete the old version withb the **delete-package-versions** command.

## Using GitHub Container Registry

- You can store container images within your organisation and user account, rather than just a repository.
- You can set permissions for the container images
- You can access public images anonymously.
- A command like this will allow you to push your container image to GHCR: ```docker push ghcr.io/OWNER/IMAGE_NAME:latest```
