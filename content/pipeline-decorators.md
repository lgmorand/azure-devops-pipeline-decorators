# Discover the pipeline decorators in Azure DevOps

Pipeline decorators are probably the most unknown feature of Azure DevOps and, at the same time, one of the most powerful. Helping companies to implement DevOps methodology including by using Azure DevOps, the most complete ALM plateform, I often remark that organizations have difficulties implementing governance. On one hand, they want to give more freedom to developers/projects teams and on the other hand, they need to ensure a minimum level of quality/security during the software development phase.

Pipelines are a solution to that because they allow to inject steps to the beginning and end of every job in any workflow of your organization and this injection is done without the consent/control of the owners of the workflows. That's what we are going to cover in this article.

This subject will be covered in four parts:

- Part 1: What are the pipeline decorators and create our first decorator
- Part 2: Deploy our decorator, validate it and enhance it
- Part 3: Create a more advanced decorator: a docker linter
- Part 4: Create another advanced decorator: a credentials scanner

## Part 1: Our first decorator

### What are pipeline decorators?

Pipeline decorators are [custom tasks](https://learn.microsoft.com/en-us/azure/devops/extend/develop/add-build-task) which can be injected automatically in all workflows on an Azure DevOps organization without the consent of the owner of the pipeline. In a perfect world, within an organization, each development team is responsible to build their pipelines and ensure they follow common the company's good practices. In some cases, to help them, a team (often the one owning the Azure DevOps plateform) creates custom tasks and make them available to users in order to enrich their pipelines. It could be a wrapper to build something complex or to call a tool such as a SCA/SAST (security code analyzer).

The issue with this approach is that you can't ensure that users will add the required tasks to their pipelines and they could easily bypass quality processes you are trying to setup during development lifecycle. That's where the pipeline decorators are the solution.

### Requirements

In order to follow this guide and be able to create pipeline decorators and test them you will require several things:

- An Azure DevOps organization where you are administrator (you can [create one for free](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization))
- [TFX CLI](https://www.npmjs.com/package/tfx-cli) which requires [NodeJS](https://nodejs.org) to be installed on your machine
- (Optional) [Visual Code Extension Manager](https://github.com/microsoft/vscode-vsce) (VSCE)

### Create our first decorator

We are going to start with a very simple example. In this "hello world" example, we are going to see how to inject a simple task in all workflows of our organization to see the concept of build and deploying a pipeline decorator. Later, we will see how to leverage the customization of these decorators and then create more complex decorators.

Create a folder and name it banner-decorator and create two files *vss-extension.json* and *banner-decorator.yml*. The structure should look like this:

```bash
| banner-decorator
| -- vss-extension.json
| -- banner-decorator.yml
```

We need to declare the actions which will be performed by our decorator, for instance, we could name it *banner-decorator.yml*. The format is the same than for YAML pipelines where you can add different steps. In our first decorator, we will add only one step to display a message in the pipeline's logs.

```yaml
steps:
  - task: CmdLine@2
    displayName: '(Injected) Here is my super banner'
    inputs:
      script: |
        echo "This step is automatically injected in your workflow as part of the governance of the company"
        
        echo "_______  _        _        _______ "
        echo "|\     /|(  ____ \( \      ( \      (  ___  )"
        echo "| )   ( || (    \/| (      | (      | (   ) |"
        echo "| (___) || (__    | |      | |      | |   | |"
        echo "|  ___  ||  __)   | |      | |      | |   | |"
        echo "| (   ) || (      | |      | |      | |   | |"
        echo "| )   ( || (____/\| (____/\| (____/\| (___) |"
        echo "|/     \|(_______/(_______/(_______/(_______)"
        echo " "
        echo " _______                    _______  _ "
        echo "(  ____ \|\     /||\     /|(  ____ \( )"
        echo "| (    \/| )   ( |( \   / )| (    \/| |"
        echo "| |      | |   | | \ (_) / | (_____ | |"
        echo "| | ____ | |   | |  \   /  (_____  )| |"
        echo "| | \_  )| |   | |   ) (         ) |(_)"
        echo "| (___) || (___) |   | |   /\____) | _ "
        echo "(_______)(_______)   \_/   \_______)(_)"
```

We now need to create the manifest to describe our extension, specify its type (decorator) and also when the conditions to inject it. Several fields are mandatory:

- **id**: An ID to name your decorator.
- **type**: Specifies that this contribution is a pipeline decorator. Must be the string **ms.azure-pipelines.pipeline-decorator**.
- **targets**: Decorators can run before your job/specified task, after, or both. See the table below for available options.
- **properties.template**: The YAML template which defines the steps for your pipeline decorator. It's a relative path from the root of your extension folder.
- **properties.targettask** (Optional): The target task id used for ms.azure-pipelines-agent-job.pre-task-tasks or ms.azure-pipelines-agent-job.post-task-tasks targets. Must be GUID string like 89b8ac58-8cb7-4479-a362-1baaacc6c7ad

The question we have to ask ourselves is "where do we want to inject our decorator?". At the beginning, at the end of the pipeline? In release pipeline or only during build pipeline?

| Target | Description |
|---|---|
|ms.azure-pipelines-agent-job.pre-job-tasks|Run before other tasks in a classic build or YAML pipeline. Due to differences in how source code checkout happens, this target runs before checkout in a YAML pipeline but after checkout in a classic build pipeline.|
|ms.azure-pipelines-agent-job.post-checkout-tasks|Run after the last checkout task in a classic build or YAML pipeline.|
|ms.azure-pipelines-agent-job.post-job-tasks|Run after other tasks in a classic build or YAML pipeline.|
|ms.azure-pipelines-agent-job.pre-task-tasks|Run before the specified task in a classic build or YAML pipeline.|
|ms.azure-pipelines-agent-job.post-task-tasks|Run after the specified task in a classic build or YAML pipeline.|
|ms.azure-release-pipelines-agent-job.pre-task-tasks|Run before the specified task in a classic RM pipeline.|
|ms.azure-release-pipelines-agent-job.post-task-tasks|Run after the specified task in a classic RM pipeline.|
|ms.azure-release-pipelines-agent-job.pre-job-tasks|Run before other tasks in a classic RM pipeline.|

In our case, we want to display the banner at the beginning of any workflow and we need to reference our YAML file (banner-decorator.yml):

```json
"contributions": [
        {
            "id": "my-required-task",
            "type": "ms.azure-pipelines.pipeline-decorator",
            "targets": [
                "ms.azure-pipelines-agent-job.pre-job-tasks"
            ],
            "properties": {
                "template": "banner-decorator.yml"
            }
        }
    ],
```

The last part is related to the files that should be included during the packaging process:

```json
"files": [
        {
            "path": "banner-decorator.yml",
            "addressable": true,
            "contentType": "text/plain"
        }
    ]
```

The final version of your *vss-extension.json* file should look like this:

```json
{
    "manifestVersion": 1,
    "id": "bannerdecorator-by-lgmorand",
    "name": "A simple banner decorator",
    "version": "1.0.0",
    "publisher": "lgmorand",
    "targets": [
        {
            "id": "Microsoft.VisualStudio.Services"
        }
    ],    
    "description": "A simple banner decorator which will display a message and a warning.",
    "categories": [
        "Azure Pipelines"
    ],
    "contributions": [
        {
            "id": "my-required-task",
            "type": "ms.azure-pipelines.pipeline-decorator",
            "targets": [
                "ms.azure-pipelines-agent-job.pre-job-tasks"
            ],
            "properties": {
                "template": "banner-decorator.yml"
            }
        }
    ],
    "files": [
        {
            "path": "banner-decorator.yml",
            "addressable": true,
            "contentType": "text/plain"
        }
    ]
}
```

We now need to package it as an extension because it is an extension of Azure DevOps. There are several types of extensions such as build tasks, Web extensions to enrich the UI but also pipeline decorators. To create our extension we need to use [TFX cli](https://www.npmjs.com/package/tfx-cli). Start by installing it on your system:

```bash
npm install -g tfx-cli
```

Then, open a prompt and ensure that you are currently located in the root folder of your pipeline decorator. From there, you can use the command *extension create*

```bash
tfx extension create
```

You should obtain a new file with the VSIX extension, based on the information you put in *vss-extension.json*.

![Package created](images/extension-created.png)

Our pipeline decorator is ready, it's no time to publish it to our organization. It will be covered in the second part on this article.

## Part 2: Deploy a pipeline decorator

### Create a publisher account

Since Azure DevOps Services is a SaaS offering, the only way to customize your organization is to install extensions but these extensions have to come from the [Azure DevOps Marketplace](https://marketplace.visualstudio.com/azuredevops). In our case, we need to publish our pipeline decorator on this marketplace (which is a extension) but do so, we need to do it under a publisher name even if, as we'll see later, nobody except our organization will be able to see/install our extension.

> Publisher account is only required if you plan to install your extension to Azure DevOps Services (SaaS version). On Azure DevOps Server (on-premise), the standalone VSIX file is sufficient.

Go to the [management portal](https://marketplace.visualstudio.com/manage) which should ask you to create your publisher account. Be aware that the  user with who you create the publisher account is important, as this account would also need to be an administrator of your Azure DevOps organization

> During the creation of my publisher, I discovered that my ID was used already. The existing publisher was created using another account when I was younger. I tried to delete the existing publisher without success until I discovered a clean solution which I [documented here](https://lgmorand.github.io/blog/delete-publisher).

### Upload your extension

It's not time to upload our extension on the marketplace. For that you need:

- the generated extension (*.vsix)
- ensuring the ID contained in its manifest (vss-extension.json) matches exactly the name of the publisher account you just created

![Extension uploaded](./images/upload%20extension.png)

Once the extension is uploaded and validated, you can check the result by opening the contextual menu (...) and choosing "*View extension*" to see the final page of your extension. That's what a user would see if your extension was public:

![Extension home page](./images/extension-page.png)

We still need to make it available from our org. To do so, from the context menu, click on "*Share/Unshare*" to open a side panel where you can type the name of organizations you want your extension be available:

![Share the extension](./images/share-with-org.png)

Now, go back to your Azure DevOps organization. Once there, open the *Organization settings* screen and then the *extensions* tab. This screen shows the **Installed** extensions, the **Requested** extensions which are the extensions users without enough right tried to add to the organization and the **Shared** extensions. That's in this last screen that you should now see your custom decorator.

![The extension is shared with the org](./images/shared-within-org.png)

It only means that the extension is available but it does not mean it is installed. You must now click on the extension and select "*Install*" to fully install it on your organization.

### Time to check that the decorator is working

This part is the easiest one. We added no specific conditions to our manifest except that it should run for any pipeline (classic or YAML) by using the property *ms.azure-pipelines-agent-job.pre-job-tasks*.

Go and run any of your pipelines and check for the result.

### Enhance our decorator

A decorator is injected implicitly in any workflow and it could surprise the users to see that something has been added to their workflow but they can't explain where it comes from. We are going to improve it using different ways:

- add an icon to brand your decorator
- add a readme-like giving information regarding your decorator
- a hint in its title
- a log message

First, we need to help our users that the injected steps are not here by mistake or by their hand. I like to add "(injected)" in the title of the steps as a hint.

```yaml
steps:
  - task: CmdLine@2
    displayName: '(Injected) Here is my super banner'
```

And you could also add a simple log message with your decorator YAML file.

```bash
echo "This step is automatically injected in your workflow as part of the governance of the company"
```

A second quick win is to add an icon to your extension which will allow to make it more recognizable. In the folder of your decorator, add a folder named "images" et put an image in PNG format (size should 128x128 pixels). Then reference this icon in the *vss-extension.json* by adding:

```json
"icons": {
        "default": "images/extension-icon.png"        
},
```

Add an overview

If you try to repackage your decorator and try to upload it to the portal, you will get an error message:

![A extension with the same version already exists](images/version-already-exists.png)

It means that you have to increment the version inside the manifest (vss-extension.json). You can do it manually or you can use the --rev-version parameter which will increment it for you during packaging

```bash
tfx extenstion create --rev-version
```

Take few seconds to increase the number of the property *version*. Your final file should be something like this:

```json
{
    "manifestVersion": 1,
    "id": "bannerdecorator-by-lgmorand",
    "name": "A simple banner decorator",
    "version": "1.0.1",
    "publisher": "lgmorand",
    "targets": [
        {
            "id": "Microsoft.VisualStudio.Services"
        }
    ],    
    "description": "A simple banner decorator which will display a message and a warning.",
    "categories": [
        "Azure Pipelines"
    ],
    "icons": {
        "default": "images/extension-icon.png"        
    },
    "contributions": [
        {
            "id": "my-required-task",
            "type": "ms.azure-pipelines.pipeline-decorator",
            "targets": [
                "ms.azure-pipelines-agent-job.pre-job-tasks"
            ],
            "properties": {
                "template": "banner-decorator.yml"
            }
        }
    ],
    "files": [
        {
            "path": "banner-decorator.yml",
            "addressable": true,
            "contentType": "text/plain"
        }
    ]
}
```

![Updated version](./images/update-version.png)

If you go back to your organization, you will remark that the extension has been automatically updated to the last version; you don't need to reinstall it.

## Part 3: Create a docker linter

For this second decorator, we would like to build a task responsible to analyze a Dockerfile and check that it follow good practices. There are plenty of tools to do like [dockle](https://github.com/goodwithtech/dockle), [hadolint](https://github.com/hadolint/hadolint) and in our case [dockerfilelint](https://github.com/replicatedhq/dockerfilelint).

Technically, we need to create a task which:

- search if Dockerfile exist in the source code
- if any file is found, install dockerfileint
- scan any Dockerfile and report the results

```yml
steps:
  - task: CmdLine@2
    displayName: '(Injected) Docker linter'
    inputs:
      script: |
        
        echo 'looking for DockerFile'
        filePath=$(find -type f -name Dockerfile)
        if [ -z "$filePath" ]
        then
              echo "No Dockerfile was found"
        else
              echo "Dockerfile(s) found: $filePath"
              echo -n -e "\e[0;32mOK\e[0m";

              echo 'installing dockerfilelint'
              npm install -g dockerfilelint --silent
              echo 'Installed version: '
              dockerfilelint -v
              echo -n -e "\e[0;32mOK\e[0m";

              echo 'running dockerfilelint'
              find -type f -name 'Dockerfile' -exec dockerfilelint {} \;
              # explanation of find+exec: https://stackoverflow.com/questions/9612090/how-to-loop-through-file-names-returned-by-find
        fi 
```


### My decorator is injected too often

Our decorator is working perfectlyt but it is also injected in all workflows of our organization, including those who don't use Docker technology. It can be an issue because it will increase the time of each pipeline execution (especially if the source code contains a large number of files) and if you have several decorators in your organization, each pipeline could get polluted by irrelevant decorators.

We need to find a way to target only pipeline which are using Dockerfile. We could restrict the decorator to specific project by adding a condition but it would require to hard-code the GUID of each project like this:

```yaml
steps:
- ${{ if eq(resources.repositories['self'].project, '123455-2492-6524-9851-564526e8fc8')
```

Another way of doing it is to target the presence of specific tasks in the pipelines. In our case, we want to check with dockerfilelint a Dockerfile before we really use it to build a Docker image. In conclusion, we need to find pipelines where we build the Docker file. The simplest way to do it is to target pipelines which contain the built-in Docker task:

![Docker task](images/docker-task.png)

In our case, the ID of the Docker task is: "e28912f1-0114-4464-802a-a3a35437fd16"

> To find the ID of a task, you can either check [this repository](https://github.com/microsoft/azure-pipelines-tasks/tree/master/Tasks) if the task is a built-in task, and open the task.json file. If it a custom task from the marketplace, you just need to download its VSIX file and unzip it to find the task.json file. To obtain the VSIX from a custom task, just go on the marketplace and try to install it. Instead of installing it on your organization, choose the Azure DevOps Server option and download the file:
![Download VSIX](images/get-task-json.png)

Once we had the Id, we need to change the target of our decorator to say "inject it before each occurence of a specific task" and we can do it using the target *ms.azure-pipelines-agent-job.pre-task-tasks* and by adding the property *targettask* with the GUID of our task.

Our final vss-extension file looks like this:

```json
"contributions": [
    {
        "id": "my-required-task",
        "type": "ms.azure-pipelines.pipeline-decorator",
        "targets": [
            "ms.azure-pipelines-agent-job.pre-task-tasks"
        ],
        "properties": {
            "template": "docker-linter-decorator.yml",
            "targettask": "e28912f1-0114-4464-802a-a3a35437fd16"
        }
    }
],
```

> Note: if the users use directly the *docker build* command with a script or a simple CmdLine task, it would not work as we can't parse the pipeline file to detect a keyword. There is no solution for such usecase.

## Part 4: Create a smart credential scanner

For this last decorator, we are going to inject another security tool in the pipeline, only if

## Conclusion

I do hope this guide will help you to leverage the power of these wonderful pipeline decorators. Far from behind perfect, they are still super powerful in terms of governance.

> If you have any remarks or improvements, don't hesitate to contact me by opening an issue on [the repository](https://github.com/lgmorand/azure-devops-pipeline-decorators).

## Useful links

- Marketplace Azure DevOps: [https://marketplace.visualstudio.com/azuredevops](https://marketplace.visualstudio.com/azuredevops)
- Develop a pipeline decorator: [https://learn.microsoft.com/en-us/azure/devops/extend/develop/add-pipeline-decorator](https://learn.microsoft.com/en-us/azure/devops/extend/develop/add-pipeline-decorator)
- Pipeline decorator express context: [https://learn.microsoft.com/en-us/azure/devops/extend/develop/pipeline-decorator-context](https://learn.microsoft.com/en-us/azure/devops/extend/develop/pipeline-decorator-context)
- Built-in Azure DevOps tasks (to get their ID in task.json): [https://github.com/microsoft/azure-pipelines-tasks](https://github.com/microsoft/azure-pipelines-tasks)
