# Lab 2 - Data

## Download the dataset

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

## Create a dataset in your Azure Machine Learning workspace

```bash
code dataset.yml
```

Add this content to the file

```yaml
$schema: https://azuremlschemas.azureedge.net/latest/dataset.schema.json
name: LegoSimpsons
version: 1
local_path: ./data
```

Now run the CLI command to upload the data to your default datastore and create the dataset

```bash
az ml dataset create -f dataset.yml
```

To see if the dataset is created you can list all the datasets in your workspace with the command below.

```bash
az ml data list --output table
```

### Checklist

Now you have versioned dataset of Simpson Images in your Azure Machine Learning Workspace

* [ ] Downloaded the dataset
* [ ] Unzipped the dataset
* [ ] Create a dataset configuration file in YAML
* [ ] Used the CLI to create the dataset in Azure Machine Learning
