# AZedgePipeline

The build and deplyoment of Azure IoT Edge Moduls is quite simple. The templates for the Azure DevOps CI/CD pipelines
are already exiting and most of the steps to make them work is straightforward. 

But let's start, and first some prework has to be done :

1. to deploy the Azure Edge Modul we need Azure Container Registry ACR. The ACR should already
   be there, due it is needed even working only local.

2. Create a Service Principle aka "Service Connection" to Push the container :
  -> go to Azure DevOps  Projects Settings
  -> got to "Service connections" and "Add New Service Connection"
  -> in the Menu choose "Azure Resource Manager" and press "Next" add the bottom
  -> the next Step will show several options, here take "Service principal (automatic)"
  -> last step choose your "Subscription" and give the  "Service Conncetion" a name
!! Note !!
You need administations access to create the "Service Connection", because a service principal will be created
!! Note !!

3. After creating the service connection in 2. we can add the build yaml 
 -> go in Azure pipelines and "New Pipeline"
 -> select "Azure Repos Git" 
 -> next step the repo 
 -> starter pipeline (better to start with an empty sheet:-)
 
 4. I copies the tasks from MS https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/build/azure-iot-edge?view=azure-devops  
    
  -> first Task is to build the new module, here you can follow the MS documentation quite foreward. Check the deployment.template.json
     that everything matches.
  
      - task: AzureIoTEdge@2
        displayName: AzureIoTEdge - Build module images
        inputs:
              action: Build module images
              templateFilePath: deployment.template.json
              defaultPlatform: amd64
              
  -> second is the upload of the module after the build into ACR, here we need the "Service Connection" from 2.
     In order to make this work, the variable "azureSubscriptionEndpoint" is the name of the "Service Connection" and
     the original yaml from MS is missing the '' see  azureContainerRegistry: '{"loginServer":"$(azureContainerRegistry)"}'.
     
     In the repo i placed a working build yaml, download and modify.
     
    variables:
 
    azureSubscriptionEndpoint: Contoso
    azureContainerRegistry: contoso.azurecr.io

    steps:    
    - task: AzureIoTEdge@2
      displayName: AzureIoTEdge - Push module images
      inputs:
            action: Push module images
            containerregistrytype: Azure Container Registry
            azureSubscriptionEndpoint: $(azureSubscriptionEndpoint)
            azureContainerRegistry: '{"loginServer":"$(azureContainerRegistry)"}'
            templateFilePath: deployment.template.json
            defaultPlatform: amd64
            fillRegistryCredential: true
   
   5. Time to test the pipeline -> save and queue a new build. The new container should be seen in your ACR.
   
   Next Steps :
   - Current you are using a MS Build Agent, i would recomend to setup an own Build Agent with a fixed OS version
   - Add Addtitional Setps to your build like deployment manifest and push to your edge device 
   - ...
   
   
  
  
