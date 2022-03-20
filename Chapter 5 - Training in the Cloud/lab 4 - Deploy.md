# Lab 5 - Deploy to Managed Endpoint

> In this lab you are going to deploy the model in managed endpoint.


Before you continue you have to make sure you have "Microsoft.PolicyInsights" resource policy enabled in your Azure subscription. 

```bash
az provider register -n 'Microsoft.PolicyInsights'
az provider show -n Microsoft.PolicyInsights --query registrationState
```


## Get the scoring script

```yaml
# Create a directory
mkdir deploy
cd deploy

# Download the scoring script
wget https://raw.githubusercontent.com/hnky/pytorch-workshop/main/workshop-assets/amls/score.py
```

> Stay in the same directory


## Create an online endpoint

> Replace: <your-endpoint-name> with your unique name, like: henks-endpoint-v1

```
az ml online-endpoint create -n <your-endpoint-name>
```

## Create an online endpoint deployment

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
environment: azureml:AzureML-pytorch-1.7-ubuntu18.04-py37-cpu-inference:32
instance_type: Standard_F2s_v2
instance_count: 1
```

Use the command below to deploy the online endpoint deployment configuration and route all the traffic to this deployment.

```yaml
az ml online-deployment create -f deployment.yml --all-traffic
```
  

## Test the online endpoint

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
az ml online-endpoint invoke --request-file request.json -n <your-endpoint-name>
```

As a result you show see that it has seen Bart Simpson on the image.

## Recap

In this lab you have created an endpoint running your PyTorch model.

**You have:**

* [ ] Downloaded the inference script
* [ ] Created a endpoint
* [ ] Created a deployment
* [ ] Tested you endpoint
