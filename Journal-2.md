## Journal 2 - 2/08/2022

The purpose of this week was an introduction too plugin development. This involved reading up on Microsoft documentation as well as any other tutorials present on the internet as well as implementing test plugins. This was a steep learning curve as there are many different purposes for plugins which require you to program in completely different ways. After reading plenty of documentation, it was then time to implement a test plugin for the dynamics environment. As it was a requirement that there would be custom entities, an ERD was presented that displayed the tables that were specific too FuseIT's needs (Dynamics has default tables made when a solution is created). Therefor, the "visits" entity was chosen for the test and would be used to learn the integration with Dynamics. The objective for this test plugin was to generate the tables when the solution is first created. 

### The steps involved for creating a plugin is rather simple:

#### Creating the Plugin

1. Firstly, you create a mew Class library (.NET Framework) project using .NET Framework 4.6.2. It is important that you use this version as other versions are not compatible with the plugin registration tool that is used later on.

   ![](https://i.imgur.com/mNrKkFs.png)

2. The next step involves installing the required NuGet Packages. This involves right clicking the project (BasicPlugin) in the file explorer and selecting **Manage NuGet Packages**

   ![](https://i.imgur.com/KzsbQ7W.png)

3. The required package is as follows:

   ```html
   Microsoft.CrmSdk.CoreAssemblies
   ```

4. The next step performed was writing the code. This would be done in the **Class1** file, however this was changed to fit the naming of the given plugin:

   ```c#
   public class GenerateEntitiesPlugin : IPlugin
   {
       public void Execute(IServiceProvider serviceProvider)
       {
           //NEED TO ADD MY CODE HERE!!!!!!!!!!!!!!
       }
   }
   ```

   5. If the code is valid, it should build when `F6` is pressed. However, if there are any issues, this will throw related errors.

   6. All assemblies containing plugins must be strongly signed. This is done using a **KeyFile**. To do this, you again right click on the project, but this time click **Properties**

      ![](https://i.imgur.com/V1lqWV2.png)

   7. Under the **Signing** tab, you click **Sign the assembly** followed by **<New...>** in the **Create Strong Name Key** dialog.
   8. Click ok and a Key is generated.
   9. Once again, build the plugin with `F6`
   10. The plugin can be found in the file explorer at `\bin\Debug\BasicPlugin.dll`

   #### Registering the Plugin

   1. After downloading the Registration Tool through the Microsoft Website, run `PluginRegistration.exe`
   2. Click **Create new Connection**
   3. Select **Office 365** as the Deployment Type
   4. Sign in using your login credentials
   5. After you are connected, you will see any existing registered plug-ins, custom workflow activities and data providers.
   6. In the **Register** drop-down, select the **New Assembly** option or press `Ctrl + A`
   7. In the **Register New Assembly** dialog, select the ellipses (**…**) button and browse to the assembly you built in the previous step.
   8. Click **Register Selected Plugins**
   9. You will see a **Registered Plugins** confirmation
   10. Click **OK** to close the dialog and close the **Register New Assembly** dialog.
   11. You should now see the **(Assembly) BasicPlugin** assembly which you can expand to view the **(Plugin) BasicPlugin.GenerateEntitiesPlugin** plugin.

   #### Registering a New Step

   1. Right-click the **(Plugin) BasicPlugin.GenerateEntitiesPlugin** and select **Register New Step**.

   2. In the **Register New Step** dialog, set the following fields (however this will be different for the actual implementation as we don't want this to be executed every time an account is created. This is just for testing purposes):

      | Setting                           | Value         |
      | --------------------------------- | ------------- |
      | Message                           | Create        |
      | Primary Entity                    | account       |
      | Event Pipeline Stage of Execution | PostOperation |
      | Execution Mode                    | Asynchronous  |

   3. Click **Register New Step** to complete the registration and close the **Register New Step** dialog.
   4. You can now see the registered step.
   5. After some time, the plugin should now work.

### Issues Faced

There were a few issues throughout the week, these were as follows:

```asciiarmor
ISSUE: Additions to the plugin were not changing in the application side

FIX: The plugin needs to be updated using the plugin registration tool every time there are changes made
```

```asciiarmor
ISSUE: When the plugin is executed, it creates the table. However, if the plugin is run again (as for testing purposes it is executed when an account is created) it throws a "Business Error" (Which is an error related to the plugin displayed on the Dynamics Interface side)

FIX: This issue has not been solved  as it will not be an issue when it is correctly implemented (as in tied to the creation of the solution and never executed again)
```

```asciiarmor
ISSUE: An error message was thrown when trying to import a solution

ERROR: Failed to find solution "GoalsandVisits_1_0_0_0.zip". Solution is not found when trying to export package

FIX: Started the export wizard again and it worked
```

```asciiarmor
ISSUE: Postman was returning a 402 (No Content) when trying to post a new visit with a link to a lead

PROBLEM: It was thought that the post would past the name of the field and the value as parameters in the body of the post method like shown below: 
	"new_new_parent_leadid": "7da38ae0-4d0e-ea11-a813-000d3a1bbd52"
	
FIX:It was discovered that you are supposted to use "odata.bind." Like it was originally thought, you still pass these parameters in the body of the post method, however, you write it as follows:
	"new_new_parent_leadid@odata.bind": "/leads(7da38ae0-4d0e-ea11-a813-000d3a1bbd52)"
```

Overall, this week was a big learning curve, but a lot of new knowledge was gained and a lot of experience was gained using Microsoft Dynamics 365.
