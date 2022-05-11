# Lab 3 - Build and deploy a PyTorch model

**In this lab you are going to train a PyTorch model on the compute cluster in Azure Machine learning you have created in the [previous lab](../Lab%202%20-%20Setup%20AML). When the training is done, you are going to take the model and deploy it to an online endpoint, so the model is available to it in an API**

## Part 1 - Data

Every machine learning project start with data. For this lab we are going to a dataset that contains photos of 10 Lego Simpson figures.
   
![Simpsons Dataset](images/dataset.jpg)


### Download the dataset

```bash
# Create a directory
mkdir data
cd data

# Download the data and validation set
wget https://github.com/hnky/dataset-lego-figures/raw/master/_download/train-and-validate.zip

# Unzip the dataset 
unzip train-and-validate.zip

# remove the zip file
rm train-and-validate.zip

# Go back to the root of your project
cd ..
```

### Create a dataset in your Azure Machine Learning workspace

```bash
code dataset.yml
```

Add this content to the file

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/data.schema.json
name: LegoSimpsons
description: Dataset with 6 Lego Figures
path: data
```

Now run the CLI command to upload the data to your default datastore and create the dataset

```bash
az ml data create -f dataset.yml
```

To see if the dataset is created you can list all the datasets in your workspace with the command below.

```bash
az ml dataset list --output table
```

### Checklist

Now you have versioned dataset of Simpson Images in your Azure Machine Learning Workspace

* [ ] Downloaded the dataset
* [ ] Unzipped the dataset
* [ ] Create a dataset configuration file in YAML
* [ ] Used the CLI to create the dataset in Azure Machine Learning


## Part 2 - Train your model


### Download the training Script

Start with creating a folder for your training scripts.

```bash
# Create a directory
mkdir train
cd train

# Download the training script
wget https://raw.githubusercontent.com/hnky/pytorch-workshop/main/workshop-assets/amls/train.py

# Go back to the project directory
cd .. 
```

>This training script is a slightly modified version of the Transfer Learning for Computer vision Tutorial on the [PyTorch website](https://pytorch.org/tutorials/beginner/transfer\_learning\_tutorial.html).


### Create a training job

Now that we have a training script we need to configure how the training job is going to run in the cloud.

We start with creating an empty yaml file.

```bash
code job.yml
```

In this file we are going to configure how to execute and run our training file. Copy and paste the content below in the job.yml.

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/commandJob.schema.json
experiment_name: SimpsonsClassification

code: ./train
command: python train.py --data-path ${{inputs.training_data}} --num-epochs 12 --model-name LegoSimpsons

environment: azureml:AzureML-pytorch-1.10-ubuntu18.04-py38-cuda11-gpu:15

compute: azureml:gpu-cluster

inputs:
  training_data:
    mode: ro_mount
    path: azureml:LegoSimpsons:1
```

Now we can create the job with the command below. The job takes around 5-10 minutes to complete.

```bash
az ml job create --file job.yml --query name -o tsv
```

The **"--query name -o tsv"** command prints the name of the run in the console. Copy this name and put it in \<run\_name> in the command below.

While the job is running, you can stream the live output of the job using the command below.&#x20;

```bash
az ml job stream -n <run_name>
```

If you just want to see the status of the job use the command below.

```bash
az ml job show -n <run_name> --query status -o tsv
```

The final step in the training scripts registers a PyTorch Model and a PyTorch model converted to ONNX. \
The names of the models are: SimpsonsClassification-onnx and SimpsonsClassification-pytorch

```bash
az ml model list -o table
```

### Checklist

Now you have 2 versioned models that can classify Simpson Images in your Azure Machine Learning Workspace.

**You have:**

* [ ] Downloaded the training script
* [ ] Create a job configuration
* [ ] Run the job
* [ ] Monitored the output
* [ ] Validated that the models have been registered

## Part 3 - Create an online Endpoint

> In this part you are going to deploy the model in an online endpoint.

Before you continue you have to make sure you have "Microsoft.PolicyInsights" resource policy enabled in your Azure subscription. This is mostly not yet enabled in Azure Pass subscription.

```bash
az provider register -n 'Microsoft.PolicyInsights'
az provider show -n Microsoft.PolicyInsights --query registrationState
```


### Get the scoring script

```yaml
# Create a directory
mkdir deploy
cd deploy

# Download the scoring script
wget https://raw.githubusercontent.com/hnky/pytorch-workshop/main/workshop-assets/amls/score.py
```

*Stay in the same directory*


### Create an online endpoint

Replace: ```<your-endpoint-name>``` with your own unique name, like: henks-endpoint-v1.

```
az ml online-endpoint create -n <your-endpoint-name>
```

### Create an online endpoint deployment

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
  code: ./
  scoring_script: score.py
environment: azureml:AzureML-pytorch-1.7-ubuntu18.04-py37-cpu-inference:32
instance_type: Standard_F2s_v2
instance_count: 1
```

Use the command below to deploy the online endpoint deployment configuration and route all the traffic to this deployment.

```yaml
az ml online-deployment create -f deployment.yml
```

When the deployment is finished you can route all the traffic in the endpoint to this deployment.

```yaml
az ml online-endpoint update --name henk  --traffic "version-1=100"
```
  

### Test the online endpoint

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

### Recap

In this lab you have created an endpoint running your PyTorch model.

**You have:**

* [ ] Downloaded the inference script
* [ ] Created a endpoint
* [ ] Created a deployment
* [ ] Tested you endpoint

