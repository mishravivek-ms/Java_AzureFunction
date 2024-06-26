
Quickstart: Create a Java function in Azure using Visual Studio Code
====================================================================


In this article
---------------

In this article, you use Visual Studio Code to create a Java function that responds to HTTP requests. After testing the code locally, you deploy it to the serverless environment of Azure Functions.

If Visual Studio Code isn't your preferred development tool, check out our similar tutorials for Java developers:

*   [Gradle](functions-create-first-java-gradle)
*   [IntelliJ IDEA](/en-us/azure/developer/java/toolkit-for-intellij/quickstart-functions)
*   [Maven](create-first-function-cli-java)

Completing this quickstart incurs a small cost of a few USD cents or less in your Azure account.

Configure your environment
--------------------------

Before you get started, make sure you have the following requirements in place:

*   An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

*   The [Java Development Kit](/en-us/azure/developer/java/fundamentals/java-support-on-azure), version 8, 11, 17 or 21(Linux).

*   [Apache Maven](https://maven.apache.org), version 3.0 or above.

*   [Visual Studio Code](https://code.visualstudio.com/) on one of the [supported platforms](https://code.visualstudio.com/docs/supporting/requirements#_platforms).

*   The [Java extension pack](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)

*   The [Azure Functions extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) for Visual Studio Code.


Install or update Core Tools
----------------------------

The Azure Functions extension for Visual Studio Code integrates with Azure Functions Core Tools so that you can run and debug your functions locally in Visual Studio Code using the Azure Functions runtime. Before getting started, it's a good idea to install Core Tools locally or update an existing installation to use the latest version.

In Visual Studio Code, select F1 to open the command palette, and then search for and run the command **Azure Functions: Install or Update Core Tools**.

This command starts a package-based installation of the latest version of Core Tools.

Create your local project
-------------------------

In this section, you use Visual Studio Code to create a local Azure Functions project in Java. Later in this article, you'll publish your function code to Azure.

1.  Choose the Azure icon in the Activity bar. Then in the **Workspace (local)** area, select the **+** button, choose **Create Function** in the dropdown. When prompted, choose **Create new project**.

    ![Screenshot of create a new project window.](media/functions-create-first-java-VSCODE/create-new-project.png)

2.  Choose the directory location for your project workspace and choose **Select**. You should either create a new folder or choose an empty folder for the project workspace. Don't choose a project folder that is already part of a workspace.

3.  Provide the following information at the prompts:

    Prompt

    Selection

    **Select a language**

    Choose `Java`.

    **Select a version of Java**

    Choose `Java 8`, `Java 11`, `Java 17` or `Java 21`, the Java version on which your functions run in Azure. Choose a Java version that you've verified locally.

    **Provide a group ID**

    Choose `com.function`.

    **Provide an artifact ID**

    Choose `myFunction`.

    **Provide a version**

    Choose `1.0-SNAPSHOT`.

    **Provide a package name**

    Choose `com.function`.

    **Provide an app name**

    Choose `myFunction-12345`.

    **Select a template for your project's first function**

    Choose `HTTP trigger`.

    **Select the build tool for Java project**

    Choose `Maven`.

    **Provide a function name**

    Enter `HttpExample`.

    **Authorization level**

    Choose `Anonymous`, which lets anyone call your function endpoint. For more information about the authorization level, see [Authorization keys](functions-bindings-http-webhook-trigger#authorization-keys).

    **Select how you would like to open your project**

    Choose `Open in current window`.

4.  Visual Studio Code uses the provided information and generates an Azure Functions project with an HTTP trigger. You can view the local project files in the Explorer. For more information about the files that are created, see [Generated project files](functions-develop-vs-code?tabs=java#generated-project-files).


Run the function locally
------------------------

Visual Studio Code integrates with [Azure Functions Core tools](functions-run-local) to let you run this project on your local development computer before you publish to Azure.

1.  To start the function locally, press F5 or the **Run and Debug** icon in the left-hand side Activity bar. The **Terminal** panel displays the Output from Core Tools. Your app starts in the **Terminal** panel. You can see the URL endpoint of your HTTP-triggered function running locally.

    ![Screenshot of the Local function VS Code output.](media/functions-create-first-java-VSCODE/functions-vscode-f5.png)

    If you have trouble running on Windows, make sure that the default terminal for Visual Studio Code isn't set to **WSL Bash**.

2.  With Core Tools still running in **Terminal**, choose the Azure icon in the activity bar. In the **Workspace** area, expand **Local Project** > **Functions**. Right-click (Windows) or Ctrl - click (macOS) the new function and choose **Execute Function Now...**.

    ![Execute function now from Visual Studio Code](media/functions-create-first-java-VSCODE/execute-function-now.png)

3.  In **Enter request body** you see the request message body value of `{ "name": "Azure" }`. Press Enter to send this request message to your function.

4.  When the function executes locally and returns a response, a notification is raised in Visual Studio Code. Information about the function execution is shown in **Terminal** panel.

5.  With the **Terminal** panel focused, press Ctrl + C to stop Core Tools and disconnect the debugger.


After you've verified that the function runs correctly on your local computer, it's time to use Visual Studio Code to publish the project directly to Azure.

Sign in to Azure
----------------

Before you can create Azure resources or publish your app, you must sign in to Azure.

1.  If you aren't already signed in, choose the Azure icon in the Activity bar. Then in the **Resources** area, choose **Sign in to Azure...**.

    ![Screenshot of the sign-in to Azure window within VS Code.](media/functions-create-first-java-VSCODE/functions-vscode-f5.png)
    functions-sign-into-azure.png)

    If you're already signed in and can see your existing subscriptions, go to the next section. If you don't yet have an Azure account, choose **Create an Azure Account...**. Students can choose **Create an Azure for Students Account...**.

2.  When prompted in the browser, choose your Azure account and sign in using your Azure account credentials. If you create a new account, you can sign in after your account is created.

3.  After you've successfully signed in, you can close the new browser window. The subscriptions that belong to your Azure account are displayed in the sidebar.


Create the function app in Azure
--------------------------------

In this section, you create a function app and related resources in your Azure subscription.

1.  Choose the Azure icon in the Activity bar. Then in the **Resources** area, select the **+** icon and choose the **Create Function App in Azure** option.

    ![Create a resource in your Azure subscription](media/functions-create-first-java-VSCODE/functions-vscode-f5.png)
    function-app-create-resource.png)

2.  Provide the following information at the prompts:

    Prompt

    Selection

    **Select subscription**

    Choose the subscription to use. You won't see this prompt when you have only one subscription visible under **Resources**.

    **Enter a globally unique name for the function app**

    Type a name that is valid in a URL path. The name you type is validated to make sure that it's unique in Azure Functions.

    **Select a runtime stack**

    Choose the language version on which you've been running locally.

    **Select a location for new resources**

    For better performance, choose a [region](https://azure.microsoft.com/regions/) near you.

    The extension shows the status of individual resources as they're being created in Azure in the **Azure: Activity Log** panel.

    ![Log of Azure resource creation](media/functions-create-first-java-VSCODE/functions-vscode-f5.png)
    resource-activity-log.png)

3.  When the creation is complete, the following Azure resources are created in your subscription. The resources are named based on your function app name:

    *   A [resource group](../azure-resource-manager/management/overview), which is a logical container for related resources.
    *   A standard [Azure Storage account](../storage/common/storage-account-create), which maintains state and other information about your projects.
    *   A function app, which provides the environment for executing your function code. A function app lets you group functions as a logical unit for easier management, deployment, and sharing of resources within the same hosting plan.
    *   An App Service plan, which defines the underlying host for your function app.
    *   An Application Insights instance connected to the function app, which tracks usage of your functions in the app.

    A notification is displayed after your function app is created and the deployment package is applied.

    Tip

    By default, the Azure resources required by your function app are created based on the function app name you provide. By default, they're also created in the same new resource group with the function app. If you want to either customize the names of these resources or reuse existing resources, you need to [publish the project with advanced create options](functions-develop-vs-code#enable-publishing-with-advanced-create-options) instead.


Deploy the project to Azure
---------------------------

Important

Deploying to an existing function app always overwrites the contents of that app in Azure.

1.  In the **Resources** area of the Azure activity, locate the function app resource you just created, right-click the resource, and select **Deploy to function app...**.

2.  When prompted about overwriting previous deployments, select **Deploy** to deploy your function code to the new function app resource.

3.  After deployment completes, select **View Output** to view the creation and deployment results, including the Azure resources that you created. If you miss the notification, select the bell icon in the lower right corner to see it again.

    ![Screenshot of the View Output window.](media/functions-create-first-java-VSCODE/function-create-notifications.png)


Run the function in Azure
-------------------------

1.  Back in the **Resources** area in the side bar, expand your subscription, your new function app, and **Functions**. Right-click (Windows) or Ctrl - click (macOS) the `HttpExample` function and choose **Execute Function Now...**.

    ![Screenshot of executing function in Azure from Visual Studio Code.](media/functions-create-first-java-VSCODE/execute-function-now.png)

2.  In **Enter request body** you see the request message body value of `{ "name": "Azure" }`. Press Enter to send this request message to your function.

3.  When the function executes in Azure and returns a response, a notification is raised in Visual Studio Code.


Clean up resources
------------------

When you continue to the [next step](#next-steps) and add an Azure Storage queue binding to your function, you'll need to keep all your resources in place to build on what you've already done.

Otherwise, you can use the following steps to delete the function app and its related resources to avoid incurring any further costs.

1.  In Visual Studio Code, press F1 to open the command palette. In the command palette, search for and select `Azure: Open in portal`.

2.  Choose your function app and press Enter. The function app page opens in the Azure portal.

3.  In the **Overview** tab, select the named link next to **Resource group**.

    ![Screenshot of select the resource group to delete from the function app page.](media/functions-create-first-java-VSCODE/functions-app-delete-resource-group.png)

4.  On the **Resource group** page, review the list of included resources, and verify that they're the ones you want to delete.

5.  Select **Delete resource group**, and follow the instructions.

    Deletion may take a couple of minutes. When it's done, a notification appears for a few seconds. You can also select the bell icon at the top of the page to view the notification.


For more information about Functions costs, see [Estimating Consumption plan costs](functions-consumption-costs).
