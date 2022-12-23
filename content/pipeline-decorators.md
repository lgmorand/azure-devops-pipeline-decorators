# Discover the pipeline decorators in Azure DevOps

Pipeline decorators are probably the most unknown feature of Azure DevOps and at the same time one of the most powerful. Helping companies to implement DevOps methodology including by using Azure DevOps, the most complete ALM plateform, I often remark that organizations have difficulties to implement governance. On hand they want to give more freedom to developers/projets teams and on the other hand, they need to ensure a minimum level of quality/security during the software development phase.

Pipeline are a solution to that because they allow to inject steps to the beginning and end of every job in any workflow of your organization and this injection is done without the consent/control of the owners of the workflows. That's what we are going to cover in this article.

This suject will be covered in four parts:

- Part 1: What are the pipeline decorators and create our first decorator
- Part 2: Deploy our decorator, validate it and enhance it
- Part 3: Create a more advanced decorator: a docker linter
- Part 4: Create another advanced decorator: a cred scanner

## Part 1: Our first decorator

### What are the pipeline decorators ?

The pipeline decorators are [custom tasks](https://learn.microsoft.com/en-us/azure/devops/extend/develop/add-build-task) which can be injected automatically in all workflows on an Azure DevOps organization without the consent of the owner of the pipeline. In a perfect world, within an organization, each team is responsible to build their own pipelines and ensure they follow common good practices. In some case, to help them, a team (often the one owning the Azure DevOps plateform) creates custom tasks and make them available to users in order to enrich their pipelines. It could be a wrapper to build something complex or to call a tool such as a SCA/SAST (security code analyzer).

The issue with this approach is that you can't ensure that users will add the required tasks to their pipelines and they could easily bypass quality processes you are trying to setup during development lifecycle. That's where the pipeline decorators are the solution.

### Create our first decorator

In this "hello world" example, we are going to see how to inject a simple task in all workflows of our organization to see the packaging concept first. Later, we'll see how to leverage the customization of these decorators.


The question we have to ask ourself is "where do we want to inject our decorator?". At the beginning, at the end of the pipeline ? In release pipeline or only during build pipeline ?

| Target | Description |
|---|---|
|ms.azure-pipelines-agent-job.pre-job-tasks|Run before other tasks in a classic build or YAML pipeline. Due to differences in how source code checkout happens, this target runs before checkout in a YAML pipeline but after checkout in a classic build pipeline.|
|ms.azure-pipelines-agent-job.post-checkout-tasks|Run after the last checkout task in a classic build or YAML pipeline.|
|ms.azure-pipelines-agent-job.post-job-tasks|Run after other tasks in a classic build or YAML pipeline.|
|ms.azure-pipelines-agent-job.pre-task-tasks|Run before specified task in a classic build or YAML pipeline.|
|ms.azure-pipelines-agent-job.post-task-tasks|Run after specified task in a classic build or YAML pipeline.|
|ms.azure-release-pipelines-agent-job.pre-task-tasks|Run before specified task in a in a classic RM pipeline.|
|ms.azure-release-pipelines-agent-job.post-task-tasks|Run after specified task in a in a classic RM pipeline.|
|ms.azure-release-pipelines-agent-job.pre-job-tasks|Run before other tasks in a classic RM pipeline.|



Our pipeline decorator is ready, it's no time to publish it to our organization. It will be covered in the second part on this article.

## Part 2: Deploy a pipeline decorator

### Create a publisher account

Since Azure DevOps Services is a SaaS offering, the only way to customizing your own organization is to install extensions but these extensions have to come from the [Azure DevOps Marketplace](https://marketplace.visualstudio.com/azuredevops). In our case, we need to publish our pipeline decorator on this marketplace (which is a extension) but do so, we need to do it under a publisher name even if, as we'll see later, nobody except our organization will be able to see/install our extension.

> You can not install directly an extension Azure DevOps Services. It is feasible with Azure DevOps Server (on-prem version)

Go to the [management portal](https://marketplace.visualstudio.com/manage) which should offer you to create your publisher account. Be aware that the account which who you create the publisher account is important, as this account would also need to be an administrator of your Azure DevOps organization

> During the creation of my publisher, I discovered that my ID was used already. The existing publisher was created using another account when I was younger. I tried to delete the existing publisher without success until I discovered a clean solution which I [documented here](https://lgmorand.github.io/blog/delete-publisher).






### Upload your extension

It's not time to upload our extension on the marketplace. For that you need:

- the generated extension (*.vsix)
- ensuring the ID contained in its manifest (vss-extension.json) matches exactly the name of the publisher account you just created




![Extension uploaded](./images/upload%20extension.png)

Once the extension is uploaded and validated, you can check the result by opening the contextual menu (...) and choose "*View extension*" to see the final page of your extension. That's what a user would see if your extension was public:

![Extension home page](./images/extension-page.png)

We still need to make it available from our org. To do so, from the context menu, click on "*Share/Unshare*" to open a side panel when you can type the name of organizations you want your extension be available:

![Share the extension](./images/share-with-org.png)

Now, go back in your Azure DevOps organization. Once there, open the *Organization settings* and the *extensions* tab. This screen shows the **Installed** extensions, the **Requested** extensions which are the extensions users without enough right tried to add to the organization and the **Shared** extensions. That's in this last screen that you should now see your custom decorator.

![The extension is shared with the org](./images/shared-within-org.png)

It only means that the extension is available but it does not mean it is installed. You must now click on the extension and select "*Install*" to fully install it on your organization.

### Time to check that the decorator is working

This part is the easiest one. We added no specific conditions to our manifest expect that it should run for any pipeline (classic or YAML) by using the property *ms.azure-pipelines-agent-job.pre-job-tasks*.

Go and run any of your pipeline and check for the result.

### Enhance our decorator

A decorator is injected implicitely in any workflow and it could surprise the users to see that something has been added to their workflow but they can't explain where it comes from.

First we need to help our users that the injected steps are not here by mistake or by their hand. Personally, I like to add "(injected)" in the title of the steps as a hint.

```yaml
steps:
  - task: CmdLine@2
    displayName: '(Injected) Here is my super banner'
```

And you could also add a simple log message with your decorator YAML file.

```bash
echo "This step is automatically injected in your workflow as part as the governance of the company"
```

A second quick win is to add an icon to your extension which will allow to make it more recognizable. In the folder of your decorator, add a folder named "images" et put an image in PNG format (size should 128x128 pixels). Then reference this icon in the vss-extension.json by adding:

```json
"icons": {
        "default": "images/extension-icon.png"        
},
```

Add an overview


dfsdfsdf
sdfsdfsdfsdfsd
fsd
fs
df
sdf
sdf

If you try to repackage your decorator and try to upload it to the portal, you will get an error message:


It means that you have to increament the version inside the manifest (vss-extension.json). You can do it manually or you can use the --rev-version parameter which will increment it for you during packaging

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


If you go back your organization you will remark that the extension has been automatically updated to the last version, you don't need to reinstall it.


### Useful links

- Marketplace Azure Devops: [https://marketplace.visualstudio.com/azuredevops](https://marketplace.visualstudio.com/azuredevops)
- Develop a pipeline decorator: [https://learn.microsoft.com/en-us/azure/devops/extend/develop/add-pipeline-decorator](https://learn.microsoft.com/en-us/azure/devops/extend/develop/add-pipeline-decorator)
- Pipeline decorator express context: [https://learn.microsoft.com/en-us/azure/devops/extend/develop/pipeline-decorator-context](https://learn.microsoft.com/en-us/azure/devops/extend/develop/pipeline-decorator-context)
- Built-in Azure DevOps tasks (to get their ID in task.json): [https://github.com/microsoft/azure-pipelines-tasks](https://github.com/microsoft/azure-pipelines-tasks)

### Conclusion

I do hope this guide will help you to leverage the power of these wonderful pipeline decorators. Far from behind perfect, they are still super powerful in terms of governance.

> If you have any remark, critic, improvment, don't hesitate to contact me by opening an issue on [the repository](https://github.com/lgmorand/azure-devops-pipeline-decorators).
