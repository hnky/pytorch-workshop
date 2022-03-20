# Lab 4 - Build a trainings Pipeline


## Part 1 - Create components


### 

```
mkdir pipeline
cd pipeline
mkdir components
cd components
mkdir src
```

### Create the train component

```
code train.yml
```

Add the content below to train.yml

```
$schema: https://azuremlschemas.azureedge.net/latest/commandComponent.schema.json
type: command

name: TrainModel
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

code:
  local_path: ./src

environment: azureml:AzureML-pytorch-1.10-ubuntu18.04-py38-cuda11-gpu:15

command: python train.py --data ${{inputs.training_data}} --train_output ${{outputs.train_output}} --num-epochs ${{inputs.epochs}} --learning_rate ${{inputs.learning_rate}} --momentum ${{inputs.momentum}}
```

Download the trainings code

```
wget 
```

### Create the register model component

```
code register.yml
```

Add the content below to register.yml

```
$schema: https://azuremlschemas.azureedge.net/latest/commandComponent.schema.json
type: command

name: Register model
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

code:
  local_path: ./src

environment: azureml:AzureML-pytorch-1.10-ubuntu18.04-py38-cuda11-gpu:15

command: python register.py --model_assets_path ${{inputs.model_assets_path}} --model_name ${{inputs.model_name}} --model_file_name ${{inputs.model_file_name}} --label_file_name ${{inputs.label_file_name}}
```

### Create the register model component

```
code convert_to_onnx.yml
```

Add the content below to convert_to_onnx.yml

```
$schema: https://azuremlschemas.azureedge.net/latest/commandComponent.schema.json
type: command

name: ConvertToOnnx
display_name: Convert PyTorch Model to ONNX
version: 1

inputs:
  input_assets_path:
    type: path
    
outputs:
  output_assets_path:
    type: path

code:
  local_path: ./src

environment: azureml:AzureML-pytorch-1.10-ubuntu18.04-py38-cuda11-gpu:15

command: python onnx.py --input_assets_path ${{inputs.input_assets_path}} --output_assets_path ${{outputs.output_assets_path}}
```

```

```


### Create the pipeline


```
$schema: https://azuremlschemas.azureedge.net/latest/pipelineJob.schema.json
type: pipeline

compute: azureml:OptimusPrime
settings:
  datastore: azureml:workspaceblobstore

inputs:
  training_data:
    dataset: azureml:LegoSimpsons-v3:3

outputs:
  final_pipeline_output:
    mode: upload

jobs:
  train_model:
    type: component
    component: file:./components/train.yml
    inputs:
      training_data: ${{inputs.training_data}}
    outputs:
      train_output: 
        mode: upload
  register_pytorch:
    type: component
    component: file:./components/register.yml
    inputs:
      model_assets_path: ${{jobs.train_model.outputs.train_output}}
      model_name: "simpsons-classification-pytorch"
      model_file_name: "model.pth"
  convert_to_onnx:
    type: component
    component: file:./components/convert_to_onnx.yml
    inputs:
      input_assets_path: ${{jobs.train_model.outputs.train_output}}
    outputs:
      output_assets_path: 
        mode: upload
  register_onnx:
    type: component
    component: file:./components/register.yml
    inputs:
      model_assets_path: ${{jobs.convert_to_onnx.outputs.output_assets_path}}
      model_name: "simpsons-classification-onnx"
      model_file_name: "model.onnx"
```