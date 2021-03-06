# Lab 4 - Build a trainings Pipeline

**In this lab we are going create a reusable pipeline to train and retrain a model and convert it from PyTorch to ONNX**


## Part 1 - Create components

**The first thing we are going to do is create reusable components.**

**An Azure Machine Learning component is a self-contained piece of code that does one step in a machine learning pipeline. Components are the building blocks of advanced machine learning pipelines. Components can do tasks such as data processing, model training, model scoring, and so on.**

[Learn more about components in pipelines](https://docs.microsoft.com/azure/machine-learning/how-to-create-component-pipelines-cli)

```
# Go back to the home directory
cd

# Create a directory for the pipeline
mkdir pipeline
cd pipeline

# Create a directory for the components
mkdir components
cd components

# Create a directory for the source scripts (don't go to the directory)
mkdir src
```

### Component 1 - Create the train component

To define an Azure Machine Learning component, you must provide two files:

- A component specification in the valid YAML component specification format. This file specifies the following information:
  - Metadata: name, display_name, version, type, and so on.   
  - Interface: inputs and outputs   
  - Command, code, & environment: The command, code, and environment used to run the component    
- A script to provide the actual execution logic.


```
# Create the component specification file
code train.yml
```

Add the content below to component specification file ```train.yml```

```
$schema: https://azuremlschemas.azureedge.net/latest/commandComponent.schema.json
type: command

name: trainmodel
display_name: Train Pytorch Classification Model
version: 1

inputs:
  training_data:
    type: path
  epochs:
    type: integer
    default: 8
  learning_rate:
    type: number
    default: 0.001
  momentum: 
    type: number
    default: 0.9
    
outputs:
  train_output:
    type: path

code: ./src

environment: azureml:AzureML-pytorch-1.10-ubuntu18.04-py38-cuda11-gpu:15

command: python train.py --data ${{inputs.training_data}} --train_output ${{outputs.train_output}} --num-epochs ${{inputs.epochs}} --learning_rate ${{inputs.learning_rate}} --momentum ${{inputs.momentum}}
```

Download the trainings code:
```
wget https://raw.githubusercontent.com/hnky/pytorch-workshop/main/Lab%204%20-%20Build%20a%20trainings%20Pipeline/scripts/train.py -P src
```

Create the component in your Azure Machine Learning workspace.
```
az ml component create --file train.yml
```

###  Component 2 - Create the register model component

```
code register.yml
```

Add the content below to register.yml

```
$schema: https://azuremlschemas.azureedge.net/latest/commandComponent.schema.json
type: command

name: registermodel
display_name: Register model after run
version: 1

inputs:
  model_assets_path:
    type: path
  model_name:
    type: string
    default: "simpsons-classification"
  model_file_name:
    type: string
    default: "model.pth"
  label_file_name:
    type: string
    default: "labels.txt" 

code: ./src

environment: azureml:AzureML-pytorch-1.10-ubuntu18.04-py38-cuda11-gpu:15

command: python register.py --model_assets_path ${{inputs.model_assets_path}} --model_name ${{inputs.model_name}} --model_file_name ${{inputs.model_file_name}} --label_file_name ${{inputs.label_file_name}}
```

```
wget https://raw.githubusercontent.com/hnky/pytorch-workshop/main/Lab%204%20-%20Build%20a%20trainings%20Pipeline/scripts/register.py -P src
```

```
az ml component create --file register.yml
```


###  Component 3 - Create the register model component

```
code convert_to_onnx.yml
```

Add the content below to convert_to_onnx.yml

```
$schema: https://azuremlschemas.azureedge.net/latest/commandComponent.schema.json
type: command

name: convert2onnx
display_name: Convert PyTorch Model to ONNX
version: 1

inputs:
  input_assets_path:
    type: path
    
outputs:
  output_assets_path:
    type: path

code: ./src

environment: azureml:AzureML-pytorch-1.10-ubuntu18.04-py38-cuda11-gpu:15

command: python onnx.py --input_assets_path ${{inputs.input_assets_path}} --output_assets_path ${{outputs.output_assets_path}}
```

```
wget https://raw.githubusercontent.com/hnky/pytorch-workshop/main/Lab%204%20-%20Build%20a%20trainings%20Pipeline/scripts/onnx.py -P src
```

```
az ml component create --file convert_to_onnx.yml
```

## Part 2 - Create the pipeline

Pipelines in AzureML let you sequence a collection of machine learning tasks/components into a workflow. Data Scientists typically iterate with scripts focusing on individual tasks such as data preparation, training, scoring, and so forth.

```
cd ..

# Create the YAML file that holds the pipeline configuration
code pipeline.yml
```

Add the content below to the ```pipeline.yml``` file.

```
$schema: https://azuremlschemas.azureedge.net/latest/pipelineJob.schema.json
type: pipeline
experiment_name: Lego-Pipeline
description: Pipeline to create a Lego Classifier in PyTorch

settings:
  default_datastore: azureml:workspaceartifactstore
  default_compute: azureml:gpu-cluster

inputs:
  training_data:
    mode: ro_mount
    path: azureml:LegoSimpsons:1

jobs:
  train_model:
    type: command
    component: azureml:trainmodel:1
    inputs:
      training_data: ${{parent.inputs.training_data}}
      epochs: 15
    outputs:
      train_output: 
        mode: upload
      
  register_pytorch:
    type: command
    component: azureml:registermodel:1
    inputs:
      model_assets_path: ${{parent.jobs.train_model.outputs.train_output}}
      model_name: "pipeline-simpsons-pytorch"
      model_file_name: "model.pth"

  convert_to_onnx:
    type: command
    component: azureml:convert2onnx:1
    inputs:
      input_assets_path: ${{parent.jobs.train_model.outputs.train_output}}
    outputs:
      output_assets_path: 
        mode: upload

  register_onnx:
    type: command
    component:  azureml:registermodel:1
    inputs:
      model_assets_path: ${{parent.jobs.convert_to_onnx.outputs.output_assets_path}}
      model_name: "pipeline-simpsons-onnx"
      model_file_name: "model.onnx"
```

### Run the pipeline

```
az ml job create -f pipeline.yml --stream
```

Navigate to [Azure Machine Learning Studio](https://ml.azure.com) to view your pipeline running.

![](img/aml-pipeline.jpg)

### See the created models

The pipeline will deliver two models and register them in model management. To view them run the command below.

```
az ml model list -o table 
```


### Recap
In this lab you have:

- [ ] Created 3 useable components
- [ ] Create a pipeline
- [ ] Run the pipeline
