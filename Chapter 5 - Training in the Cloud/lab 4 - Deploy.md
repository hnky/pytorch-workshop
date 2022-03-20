# Lab 5 - Deploy to Managed Endpoint


Before you continue you have to make sure you have "Microsoft.PolicyInsights" resource policy enabled in your Azure subscription. 

```bash
az provider register -n 'Microsoft.PolicyInsights'
az provider show -n Microsoft.PolicyInsights --query registrationState
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

> Replace: <your-endpoint-name> with your unique name, like: henks-endpoint-v1

```
az ml online-endpoint create -n <your-endpoint-name>
```

### Create the endpoint configuration

Create an empty yml file for the deployment configuration.

```yaml
code deployment.yml
```

Add the content below to the file

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/managedOnlineDeployment.schema.json
endpoint_name: <your-endpoint-name>
name: version-1
app_insights_enabled: true
model: azureml:LegoSimpsons-pytorch:1
code_configuration:
  code: 
    local_path: ./
  scoring_script: score.py
environment: 
  conda_file: ./conda.yml
  image: mcr.microsoft.com/azureml/openmpi3.1.2-ubuntu18.04:20210727.v1
instance_type: Standard_F2s_v2
instance_count: 1
```

Use the command below to create a managed endpoint

```yaml
az ml online-deployment create -f deployment.yml --all-traffic
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
az ml online-endpoint invoke -n <your-endpoint-name> --request-file request.json
```

As a result you show see that it has seen Bart Simpson on the image.

### Recap
