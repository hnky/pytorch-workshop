# Lab 5 - Deploy to Managed Endpoint


> **This lab may not work in an Azure Pass Subscription.**
> 
> For this lab to work, the Microsoft.PolicyInsights needs to be registered in your subscription. [Follow this tutorial](https://docs.microsoft.com/azure/azure-resource-manager/management/resource-providers-and-types#register-resource-provider)
{% endhint %}

```bash
# Delete the compute cluster (only needed if you have an Azure Pass subscription)
az ml compute delete -n gpu-cluster
```

In this lab you are going to deploy the model in managed endpoint.

### Get the scoring script

```yaml
# Create a directory
mkdir deploy
cd deploy

# Download the scoring script
wget https://raw.githubusercontent.com/hnky/pytorch-workshop/main/workshop-assets/amls/score.py
```

> Stay in the same directory


### Create the scoring environment

Create an empty yml file for the environment configuration.

```yaml
code conda.yml
```
Add the content below to the conda.yml file.

```yaml
name: project_environment
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.6.2
  - pip:
    - azureml-defaults
    - torch
    - torchvision
    - pillow==5.4.1
```

### Create a managed endpoint

> Replace: \<your-name> with your unique name, like: henks-endpoint

```
az ml online-endpoint create -n <your-name>
```

### Create the endpoint configuration

Create an empty yml file for the deployment configuration.

```yaml
code deployment.yml
```

Add the content below to the file

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/managedOnlineEndpoint.schema.json
name: <your-name>
type: online
auth_mode: key
traffic:
  version1: 100
deployments:
  - name: version1    
    app_insights_enabled: true
    model: azureml:SimpsonsClassification-pytorch:1
    code_configuration:
      code: 
        local_path: ./
      scoring_script: score.py
    environment: 
      name: simpsons-inference
      version: 1           
      path: .
      conda_file: file:conda.yml
      docker:
        image: mcr.microsoft.com/azureml/openmpi3.1.2-ubuntu18.04:20210727.v1
    instance_type: Standard_F4s_v2
    scale_settings:
      scale_type: Manual
      instance_count: 1
      min_instances: 1
      max_instances: 1
```

> Replace: \<your-name> with your unique name, like: henks-endpoint

Use the command below to create a managed endpoint

```yaml
az ml endpoint create --file deployment.yml
```

### Test the endpoint

Create a new file for the request json.

```yaml
code request.json
```

Add the content below to the file

```yaml
{
    "image":"https://raw.githubusercontent.com/hnky/dataset-lego-figures/master/_test/Bart.jpg"
}
```

Use the command below to invoke the endpoint!

```yaml
az ml endpoint invoke -n simpsons-endpoint --request-file request.json
```

As a result you show see that it has seen Bart Simpson on the image.

### Recap
