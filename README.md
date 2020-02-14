# Azure Marketplace Azure Application (formerly known as Solution Template) Helpers

The maven projects in this repository provide help for creating Azure
Marketplace offers of type "Azure Application".  For more details on
Azure Applications (formerly known as Solution Templates) see 
[Marketplace Solution Templates](https://docs.microsoft.com/en-us/azure/marketplace/marketplace-solution-templates).

## Usage

1. Clone this repository locally

2. Build and install the `arm-assembly` maven project locally.

   ```
   cd arm-assembly && mvn -DskipTests clean install
   ```

3. Build and install the `arm-parent` maven project locally.

   ```
   cd arm-parent && mvn -DskipTests clean install
   ```
   
4. Clone
   [arm-ttk](https://github.com/Azure/arm-ttk)
   as a sibling of this directory.  This is necessary because the test
   phase of the `arm-parent` maven project will run the tests in
   `${basedir}/../arm-ttk/arm-ttk` against your
   solution template.  You must ensure the preconditions for running the
   tests are met, as described in the test documentation.  Test
   documentation is found in [the arm-ttk subdirectory](https://github.com/Azure/arm-ttk/tree/master/arm-ttk).

   If you want to put the tests in a different location, set the
   `template.validation.tests.directory` property.

5. Create a maven project with the following layout

   ```
   ├── pom.xml
   └── src
       └── main
           ├── arm
           │   ├── createUiDefinition.json
           │   ├── mainTemplate.json
           └── scripts
               └── any scripts that are referenced in the mainTemplate.json
   ```

   In the `pom.xml` shown above, declare the parent like this:

   ```
   <parent>
       <groupId>com.microsoft.azure.iaas</groupId>
       <artifactId>azure-javaee-iaas-parent</artifactId>
       <version>whatever version you get from step 3</version>
       <relativePath></relativePath>
   </parent>
   ```
   
6. Run the build

   ```
   mvn -Ptemplate-validation-tests clean install
   ```
   
   This will run the tests against your solution template.  The output should look similar to the following.
   
   ```
   mvn -Ptemplate-validation-tests clean install
   [INFO] Scanning for projects...
   [INFO] 
   [INFO] -----------< com.oracle.weblogic.azure:arm-oraclelinux-wls >------------
   [INFO] Building arm-oraclelinux-wls 1.0.13
   [INFO] --------------------------------[ jar ]---------------------------------
   ...
   [INFO] --- exec-maven-plugin:1.6.0:exec (test) @ arm-oraclelinux-wls ---
                               Validating arm\createUiDefinition.json
     CreateUIDefinition
       [+] Allowed Values Should Actually Be Allowed (134 ms)

       [+] CreateUIDefinition Must Have Parameters (12 ms)
       [+] CreateUIDefinition Should Have Schema (30 ms)
       [+] Credential Confirmation Should Not Be Hidden (85 ms)
       [+] Handler Must Be Correct (12 ms)
       [+] Location Should Be In Outputs (16 ms)
       [+] Outputs Must Be Present In Template Parameters (15 ms)
       [+] Parameters Without Default Must Exist In CreateUIDefinition (26 ms)
       [+] Password Textboxes Must Be Used For Password Parameters (37 ms)
       [+] Textboxes Are Well Formed (37 ms)
       [+] Tooltips Should Be Present (49 ms)
       [+] Usernames Should Not Have A Default (39 ms)
       [+] VMSizes Must Match Template (40 ms)
   Validating arm\mainTemplate.json
     deploymentTemplate
       [+] DeploymentTemplate Schema Is Correct (15 ms)
       [+] IDs Should Be Derived From ResourceIDs (110 ms)
       [+] Location Should Not Be Hardcoded (49 ms)
       [+] ManagedIdentityExtension must not be used (13 ms)
       [+] Min And Max Value Are Numbers (15 ms)
       [+] Outputs Must Not Contain Secrets (16 ms)
       [+] Parameters Must Be Referenced (23 ms)
       [+] Parameters Property Must Exist (10 ms)
       [+] PublicIP DNS Should Not Contain Concat (13 ms)
       [+] ResourceIds should not contain (37 ms)
       [+] Resources Should Have Location (14 ms)
       [+] Secure String Parameters Cannot Have Default (12 ms)
       [+] Template Should Not Contain Blanks (23 ms)
       [+] VM Images Should Use Latest Version (14 ms)
       [+] VM Size Should Be A Parameter (62 ms)
       [+] Variables Must Be Referenced (21 ms)
       [+] Virtual Machines Should Not Be Preview (15 ms)
       [+] adminUsername Should Not Be A Literal (63 ms)
       [+] apiVersions Should Be Recent (101 ms)
       [?] artifacts parameter (32 ms) 
           ENV:SAMPLE_NAME is empty - using placeholder for manual verification: 100-blank-template

       [+] providers apiVersions Is Not Permitted (12 ms)
   ...
   [WARNING] The assembly descriptor contains a filesystem-root relative reference, which is not cross platform compatible /
   [INFO] Building zip: /mnt/c/Users/edburns/Documents/azure/workareas/20190801-jacob-thomas/arm-oraclelinux-wls/target/arm-oraclelinux-wls-1.0.13-arm-assembly.zip
   ...   
   ```

   The zip file is what must be uploaded to the Azure marketplace as described in ["Package file (.zip)"](https://docs.microsoft.com/en-us/azure/marketplace/cloud-partner-portal/azure-applications/cpp-skus-tab#package-details-for-solution-template) 

## Contents

### arm-parent

This is intended to be the parent POM for a maven project that
represents an Azure Application.

### arm-assembly

The maven assembly that creates the Package file (.zip).

# Tips for publishing Azure Applications to the Marketplace

These lessons are taken from the experience of publishing the Oracle
WebLogic on Azure Application Offers to the marketplace, but apply to
any Azure Application.  See
[arm-oraclelinux-wls](https://github.com/wls-eng/arm-oraclelinux-wls)
and its sibling projects for examples of how this parent pom is used.

Please also read the [ARM Template Best Practices document](https://github.com/Azure/azure-quickstart-templates/blob/master/1-CONTRIBUTION-GUIDE/best-practices.md).

### General Lessons for all json files

* Format all json files with `SHIFT+ALT+F` in VS Code.

### General Lessons for `mainTemplate.json`

* You must include a `trackingPid` in `resources` like so:

   ```
   {
      "apiVersion": "2018-02-01",
      "name": "${tracking.pid}",
      "type": "Microsoft.Resources/deployments",
      "properties": {
         "mode": "Incremental",
         "template": {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "resources": []
         }
      }
   },

   ```

  If you don't, you'll get this when you try to deploy:
  
   ```
   Error Message Azure usage tracking resource missing in one or more ARM
   templates. To resolve, add a tracking GUID to the ARM template in
   mainTemplate.json packages for the following plan:
   20190829-arm-oraclelinux-wls-cluster. For more details, see
   https://aka.ms/aboutinfluencedrevenuetracking.
   ```
   
   In the above snippet the value `${tracking.pid}` is templated by
   maven because we are using maven to create the zip bundle.

* `_artifactsLocationSasToken` is required in `parameters`.  Like so:

   ```
   "_artifactsLocationSasToken": {
       "type": "securestring",
       "metadata": {
           "description": "The sasToken required to access _artifactsLocation.  When the template is deployed using the accompanying scripts, a sasToken will be automatically generated. Use the defaultValue if the staging location is not secured."
       },
       "defaultValue": ""
   },
   
   ```
   

* Do not include any scripts that download resources from user specific sites.  For example:

   ```
   curl -s https://raw.githubusercontent.com/typekpb/oradown/master/oradown.sh  | bash -s -- --cookie=accept-weblogicserver-server --username="${otnusername}" --password="${otnpassword}" https://download.oracle.com/otn/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
   ```
  
  Is not allowed.  To get the same effect, include the script in the
  `scripts` directory of the zip and reference it like so:
  
   ```
   export SCRIPT_PWD=`pwd`
   chmod ugo+x ${SCRIPT_PWD}/oradown.sh 

   ${SCRIPT_PWD}/oradown.sh --cookie=accept-weblogicserver-server --username="${otnusername}" --password="${otnpassword}" https://download.oracle.com/otn/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
   ```

* Use the latest versions for all the things.

   For example:

   ```
   az provider show --namespace Microsoft.Storage --query "resourceTypes[*].resourceType" --out table
   ```

   From the output, pick `storageAccounts`.


   ```
   az provider show --namespace Microsoft.Storage --query "resourceTypes[?resourceType=='storageAccounts'].apiVersions | [0]" --out table
   ```

   Scan your mainTemplate.json for date strings:

   ```
   grep "[0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]" mainTemplate.json
   ```

* In `parameters`, `_artifactsLocation`, `defaultValue` must be
  `[deployment().properties.templateLink.uri]`.  This will cause any
  scripts bundled in the zip to be referencable like so:
  
   ```
   "settings": {
       "fileUris": [
           "[uri(parameters('_artifactsLocation'), concat('scripts/', variables('ScriptFileName'), parameters('_artifactsLocationSasToken')))]",
           "[uri(parameters('_artifactsLocation'), concat('scripts/', variables('oradownScript'), parameters('_artifactsLocationSasToken')))]"
       ],
   ```

* When referencing scripts to be run as `"type":
  "Microsoft.Compute/virtualMachines/extensions"`, include them in the
  zip bundle, put them in the `scripts` sub-directory of the top level
  of the zip bundle, and reference them like so:
  
   ```
   "settings": {
     "fileUris": [
        "[uri(parameters('_artifactsLocation'), concat('scripts/', variables('ScriptFileName'), parameters('_artifactsLocationSasToken')))]"
     ],
     "commandToExecute": "[concat('sh',' ',variables('ScriptFileName'),' ',parameters('acceptOTNLicenseAgreement'),' ',parameters('otnAccountUsername'),' ',parameters('otnAccountPassword'))]"
   }
   ```

* For any scripts referenced in mainTemplate.json, put the script name in the `variables` section, like so:

   ```
   "oradownScript": "oradown.sh",
   ```

* Define one or more parameters with `guid()` for when you need
  uniqueness.  This must be done in `parameters` with a `defaultValue`.
  Like so:

   ```
   "guidValue": {
         "type": "string",
         "defaultValue": "[newGuid()]"
   },
   
   ...
   
   "variables": {
      "storageAccountName": "[concat(take(replace(parameters('guidValue'),'-',''),6),'olvm')]",
   ```

* When referring to a `storageAccountName`, it must be unique, like so:

   ```
    "variables": {
      "storageAccountName": "[concat(take(replace(guid(toLower(resourceGroup().name)),'-',''),6),'olvm')]",
   ```

### General Lessons for `createUiDefinition.json`

* Be as specific as possible in tooltips.  For example:

   ```
   {
     "name": "adminUsername",
     "type": "Microsoft.Common.TextBox",
     "label": "Username for admin account of VMs",
     "defaultValue": "weblogic",
     "toolTip": "Use only letters and numbers",
     "constraints": {
         "required": true,
         "regex": "^[a-z0-9A-Z]{1,30}$",
         "validationMessage": "The value must be 1-30 characters long and must only contain letters and numbers."
     },
     "visible": true
   },   
   ```

   Instead of 
   
   ```
   "toolTip": "Use only allowed characters",
   ```
   
   Another example:
   
   ```
   {
       "name": "managedServerPrefix",
       "type": "Microsoft.Common.TextBox",
       "label": "Managed Server Prefix",
       "toolTip": "The string to prepend to the name of the managed server.  Must start with letters and not have a run of more than eight digits.",
       "defaultValue": "msp",
       "constraints": {
           "required": true,
           "regex": "^[a-z0-9A-Z]{3,79}$",
           "validationMessage": "The prefix must be between 3 and 79 characters long and contain letters, numbers only."
       }
   },   
   ```
   
   
* Try to sort things alphabetically.

* For elements of `"type": "Microsoft.Common.PasswordBox"`, use the most
  restrictive RegEx you possible.  Such as:
  
   ```
   {
       "name": "adminPasswordOrKey",
       "type": "Microsoft.Common.PasswordBox",
       "label": {
           "password": "Password for admin account of VMs",
           "confirmPassword": "Confirm password"
       },
       "toolTip": "Password for admin account of VMs",
       "constraints": {
           "required": true,
           "regex": "^((?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])|(?=.*[0-9])(?=.*[a-z])(?=.*[!@#$%^&*])|(?=.*[0-9])(?=.*[A-Z])(?=.*[!@#$%^&*])|(?=.*[a-z])(?=.*[A-Z])(?=.*[!@#$%^&*])).{12,72}$",
           "validationMessage": "Password must be at least 12 characters long and have 3 out of the following: one number, one lower case, one upper case, or one special character"
       },
       "options": {
           "hideConfirmation": false
       },
       "visible": true
   },
   ```

   Ensure the `validationMessage` expresses exactly what the RegEx requires.
   
   Ensure the minimum length of the password is at least 12 characters.
   
   Ensure `"hideConfirmation"` is `false`, like so.
   
   ```
   "options": {
     "hideConfirmation": false
   },
   
   ```
   
* If the element of type `"type": "Microsoft.Common.PasswordBox"` is
  actually a value from another site, such as the password for an user
  id that is an Oracle account or IBM account, use no RegEx and call it
  out specifically in the `validationMessage`, like so:
  
   ```
   {
       "name": "otnAccountPassword",
       "type": "Microsoft.Common.PasswordBox",
       "label": {
           "password": "Password for OTN Account",
           "confirmPassword": "Confirm password"
       },
       "toolTip": "Password for OTN Account",
       "constraints": {
           "required": true,
           "validationMessage": "Validation constraints for OTN accounts apply here."
       },
       "options": {
           "hideConfirmation": false
       },
       "visible": true
   }
   
   ```
   
* Ensure the type of each output is correct with respect to what the
  `mainTemplate.json` expects.  For example, if something is an `int`
  but it is entered by the user as a string, it must be converted with the `int()` function, like so:
  
   ```
   "outputs": {
   ...
       "numberOfInstances": "[int(basics('numberOfInstances'))]",
   ...
   }   
   ```
  

* Be mindful of the total length of strings.  For example, we had
  `dnsLabelPrefix` with a max length of 79.  We were told to shorten it
  a lot.  I chose 10, like so:
  
   ```
   {
       "name": "dnsLabelPrefix",
       "type": "Microsoft.Common.TextBox",
       "label": "DNS Label Prefix",
       "toolTip": "The string to prepend to the DNS label.",
       "defaultValue": "wls",
       "constraints": {
           "required": true,
           "regex": "^[a-z0-9A-Z]{3,10}$",
           "validationMessage": "The prefix must be between 3 and 10 characters long and contain letters, numbers only."
       }
   }
   ```
  
* If there are any legal things that require acceptance of TOS, prevent
  the deployment from proceeding unless they accept the TOS, like so:
  
   ```
   {
       "name": "acceptOTNLicenseAgreement",
       "label": "Accept OTN License Agreement",
       "type": "Microsoft.Common.TextBox",
       "tooltip": "A value of N indicates you do not accept the OTN License Agreement.  In that case the deployment will fail.",
       "visible": true,
       "defaultValue": "Y",
       "constraints": {
           "required": true,
           "regex": "^[Yy]$",
           "validationMessage": "The value must be Y/y to proceed with deployment."
       }
   },

   ```

### arm-oraclelinux-wls

#### mainTemplate.json

* in the dictionary with "type": "Microsoft.Compute/virtualMachines", there is a dictionary like this:

   ```
   "diagnosticsProfile": {
     "bootDiagnostics": {
        "enabled": true,
        "storageUri": ""
     }
   }
   ```
   
   The value for `storageUri`:
   
   * must not use `concat`.
   
   * must use `resoucreId()`
   
   Like so:
   
   ```
   "diagnosticsProfile": {
   "bootDiagnostics": {
      "enabled": true,
      "storageUri": "[reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName')), '2019-06-01').primaryEndpoints.blob]"
   }
   }
   ```
   
### arm-oraclelinux-wls-admin

#### mainTemplate.json

* Elements of type `"type": "Microsoft.Storage/storageAccounts"` must have an empty `properties`, like so:


   ```
   {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
         "name": "[variables('storageAccountType')]"
      },
      "kind":"Storage",
      "properties": {

      }
   },

   ```

* Things that show up in domain names must be both shortish and unique.  Like so:

   ```
   {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2018-11-01",
      "name": "[variables('publicIPAddressName')]",
      "location": "[parameters('location')]",
      "properties": {
         "publicIPAllocationMethod": "[variables('publicIPAddressType')]",
         "dnsSettings": {
            "domainNameLabel": "[concat(toLower(parameters('dnsLabelPrefix')),'-',take(replace(parameters('guidValue'), '-', ''), 10),'-',toLower(parameters('wlsDomainName')))]"
         }
      }
   }
   ```



# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
