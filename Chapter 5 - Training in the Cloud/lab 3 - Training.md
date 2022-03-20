# Lab 3 - Train your model

## Download the training Script

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
code:
  local_path: ./train
command: python train.py --data-path ${{inputs.training_data}} --num-epochs 12 
environment: azureml:AzureML-pytorch-1.10-ubuntu18.04-py38-cuda11-gpu:15
compute: azureml:gpu-cluster
inputs:
  training_data:
    mode: ro_mount
    dataset: azureml:LegoSimpsons:1
```

|                           |                                                                                                                                                                            |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p>code<br>local_path</p> | This is the folder that contains the train.py and other files needed for your job to run successful. Everything is this folder is copied over to the experiment artifacts. |
| command                   | The command                                                                                                                                                                |

Now we can create the job with the command below. The job takes around 5-10 minutes to complete.

```bash
az ml job create --file job.yml --query name -o tsv
```

The **"--query name -o tsv"** command prints the name of the run in the console. Copy this name and put it in \<run\_name> in the command below.

While the job is running, you can stream the live output of the job using the command below.&#x20;

```bash
az ml job stream -n <run_name>
```

> **In the current version of the SDK the command above does not work.**

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
