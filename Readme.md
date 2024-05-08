# Predicting Failures of Autoscaling Distributed Applications

This replication package can be used to fully replicate the results of our paper [_Predicting Failures of Autoscaling Distributed Applications_](https://2024.esec-fse.org/details/fse-2024-research-papers/55/Predicting-Failures-of-Autoscaling-Distributed-Applications) accepted by FSE 2024.

## Table of Contents

1. [Introduction](#introduction)
2. [Replication Package Structure](#replication-package-structure)
3. [Quick Start](#quick-start)
4. [Experimental Procedure](#experimental-procedure)

## Introduction

Our work introduces **PREFACE**, an approach which combines descriptive statistics with a generative neural network (autoencoder) to reveal anomalous KPI values that are symptoms of incoming system failures, and ranks the microservices that are likely responsible for the failure. **PREFACE** introduces a prepocessing step exploiting descriptive statistics, to deal with time series of KPI sets with size that varies over time, as in autoscaling distributed applications.

This replication package includes:

1. **A large dataset of KPIs** collected from **Alemira**, a commercial Learning Managing System developed in Constructor Tech and currently in use in several educational institutions, and **TrainTicket**, a microservice application widely used in research projects. Both are microservice-based applications deployed on Kubernetes that takes full advantage of its autoscaling mechanisms.
2. **The results of the experiments of PREFACE**, _PREdicting Failures in AutosCaling distributEd Applications_, the approach presented in our manuscript which predicts and localizes failures in autoscaling distributed applications.
3. **The toolset to execute PREFACE** to replicate the results obtained based on the provided dataset.

Besides, we also implemented a set of utilities, that are excluded here for the sake of simplicity, to automate the whole process of experiments, including [alemira-traffic-generator](https://github.com/ketaiq/alemira-traffic-generator), [alemira-metrics-aggregator](https://github.com/ketaiq/alemira-metrics-aggregator), [alemira-metrics-collector](https://github.com/ketaiq/alemira-metrics-collector), [chaos-mesh-failure-injector](https://github.com/ketaiq/chaos-mesh-failure-injector), [train-ticket-deployment](https://github.com/ketaiq/train-ticket-deployment), [train-ticket-hpa](https://github.com/ketaiq/train-ticket-hpa), [train-ticket-traffic-generator](https://github.com/ketaiq/train-ticket-traffic-generator), [gcloud-metrics-collector](https://github.com/ketaiq/gcloud-metrics-collector) and [gcloud-metrics-aggregator](https://github.com/ketaiq/gcloud-metrics-aggregator). These utilities can be useful for those who want to replicate experiments from scratch.


### Terminology

* **KPI**: Key Performance Indicators, values of the metrics collected from the **Alemira** and **TrainTicket** systems on a microservice level.
* **Anomalous KPI**: KPIs with a reconstruction error which is above the thershold of the KPI, calculated as three standard deviations of KPI's values on the normal dataset.
* **Deep Autoencoder**: the component of PREFACE that identifies the anomalous KPIs by computing the reconstruction error for each KPI alongside the overall reconstruction error. The architecture (size and number of layers) and hyperparameters of the Deep Autoencoder were defined and fine-tuned during the model validation process.
* **Localizer**: this component aggregates the score of the anomalous KPIs that belong to the same micorservice and ranks them, signaling as failing microserivces the top ranked at each timestamp for which PREFACE predicts an anomalous state.

### Dataset Naming Conventions

The datasets collected during normal execution are named as `normal_1_14.csv` for **Alemira** and `normal-2weeks.csv` for **TrainTicket**. The datasets comprise the data collected over two weeks of normal execution without failures and are used to train and validate the Deep Autoencoder. The datasets include time series of KPIs collected from multiple monitoring tools (described in Section 4.1.3 in our paper).

The datasets collected during the execution with injected failures are named as `linear-{failure-type}-{target service}-{unique identifier}.csv`, e.g., `linear-cpu-stress-ts-basic-service-020616.csv`.

## Replication Package Structure

In this replication package there are two folders, one for each system in our case study, named _Alemira_ and _TrainTicket_. Each folder is composed as follow

* _dataset_tune.ipynb_ is the notebook responsible to tune the datasets, including alligning the failure injection dataset to the training dataset, removing columns that are constant, and columns with empty values.


* _data_set_normalize.ipynb_ is the notebook that normalizes the datasets using the _min-max_ normalization technique.

These two last scripts are unified for Trainticket as _dataset_tune_normalize.ipynb_


* _predict.ipynb_ is responsible to train the Autoencoder model and generate the predictions according to its reconstruction error. This notebook also calculate the ranking of the services in order to allow the localization of the failure.


* _results.ipynb_ is used to generate the graphs and plots shown in the manuscript

* _input_ contains the folder _input_: this folder contains the dataset collected and needed to run the experiments. More specifically:
  * _datasets_ contains the subfolder _Consolidated_ where all the datasets related to both the normal execution and the failure injection execution can be found. This contains the dataset needed for the training of the model.
  * _other_ contains the _failure-injection-log.csv_, where the information of each failure injection are stored, including _Failure Type_, _Failure Pattern_, _Target Service_, _Beginning of the Experiment_, _End of the Experiment_, _Name of the Relative Dataset_, and _System Disruption Timestamp_.


* _output_ contains the folder _output-111_ for Alemira and _output-train_ticket_ for TrainTicket: this folder contains all the output files generated from the scripts used. This file are saved in multiple subfolder contained in _output-111_ and _output-train_ticket_. More specifically:
  * _datasets_ contains two subfolders, _Tuned_ and _Normalized_. These contain the preprocessed datasets and the normalized dataset according to the _min-max_ normalization technique respectively.
  * _predictions_ contains a .csv file for each failure injection dataset in which, for each timestamp, it stores a boolean value _1_ or _0_ indicating whether PREFACE predicted a failure or not.
  * _anomalies_list_ contains a .csv file for each failure injection dataset, where we stored the reconstruction error of each anomalous KPI for each timestamp. This is used for debugging purposes.
  * _anomalies_lists_services_only_ similarly, contains a .csv file for each failure injection dataset, where we stored the z-score of the reconstruction error of each anomalous KPI related to the services ranked from the biggest to the smallest.
  * _anomalies_lists_services_only_sliding_window_ as before, contains a .csv file for each failure injection dataset, where for each minute we stored the ranked z-score of the z-score of the reconstruction error of the anomalous KPIs, calculated using the 20-minutes sliding window method described in the manuscript.
  * _localisations_re_sliding_window_ includes a .csv file for each failure injection dataset, where we stored the ranking of the services using the z-score of the z-score of the reconstruction error calculated as described in the manuscript.
  * _models_ is the folder in which we store the trained Autoencoder.
  * _other_ stores a .csv file detailing the timing of each failure injection, including the _Failure Injection Experiment Name_, the _Total Number of Timestamps_, the _Timestamp at which Failure Injection Started_ and the _Timestamp at which Failure Injection Ended_
  * kpis_not_seen_in_prod
All the files in the _output_ folder are generated once the scripts are executed.

* _predict_notebook_sections_ folder contains some Jupyter Notebooks with functions that are used from the four main scripts described before.


* _functions.ipynb_ is a notebook containing additional useful functions used by the scripts described before.
  

* _Configs_ is a folder that contains the configurations needed from the scripts to run.


## Quick Start

### Prerequisites

To run the experiment we used a machine with the following configuration. This is a tested setup, but the scripts presented in this replication package can be run using also other OS (Windows or Linux).

- OS: MacOS Catalina
- Processor: 2.2 GHz 6-Core Intel Core i7
- Memory: 16 GB 2400 MHz DDR4
- Software packages
    - [Python 3](https://www.python.org/downloads/)
    - [Conda](https://docs.conda.io/projects/miniconda/en/latest/miniconda-install.html)
    - [tar](https://www.gnu.org/software/tar/)

### Step 1. Extract datasets from compressed tar archives

#### Alemira
```sh
# Extract the consolidated dataset for 2-week normal execution of Alemira
cd {PATH TO PREFACE}/PREFACE
cd Alemira/input/datasets/Consolidated
tar -xzf normal_1_14.csv.zip

# Extract the normalized dataset for 2-week normal execution of Alemira
cd {PATH TO PREFACE}/PREFACE
cd Alemira/output/output-111/datasets/Normalized
tar -xzf normal_1_14.csv.zip

# Extract the tuned dataset for 2-week normal execution of Alemira
cd {PATH TO PREFACE}/PREFACE
cd Alemira/output/output-111/datasets/Tuned
tar -xzf normal_1_14.csv.zip
```

#### TrainTicket
```sh
# Extract the consolidated dataset for 2-week normal execution of TrainTicket
cd {PATH TO PREFACE}/PREFACE
cd TrainTicket/input/datasets/Consolidated
cat normal-2weeks.tar.gz* | tar zx

# Extract the normalized dataset for 2-week normal execution of TrainTicket
cd {PATH TO PREFACE}/PREFACE
cd TrainTicket/output/output-train_ticket/datasets/Normalized
cat normal-2weeks.tar.gz* | tar zx

# Extract the tuned dataset for 2-week normal execution of TrainTicket
cd {PATH TO PREFACE}/PREFACE
cd TrainTicket/output/output-train_ticket/datasets/Tuned
cat normal-2weeks.tar.gz* | tar zx
```

### Step 2. Setup environment

```sh
# Create conda environment
conda create --name preface-analysis --channel conda-forge python=3.10 jupyterlab numpy pandas scikit-learn matplotlib plotly scipy tensorflow

# Activate conda environment
conda activate preface-analysis

# Open jupyter notebooks in the project folder
cd {PATH TO PREFACE}/PREFACE
jupyter lab
```

### Step 3. Run jupyter notebooks one by one

#### Alemira
1. dataset_tune.ipynb
2. data_set_normalize.ipynb
3. predict.ipynb 
4. results.ipynb

#### TrainTicket
1. dataset_tune_normalize.ipynb
2. predict.ipynb
3. results.ipynb

## Experimental Procedure

Running the experiments includes multiple passages, namely: _data preprocessing_, _predictions generation_, and _results visualization_. We indicate details of each script as follows.

### Data Preprocessing

Script: _dataset_tune.ipynb_

- Purpose: data preprocessing
  - Input: _input/datasets/Consolidated_ - raw data with each data set consolidated in a single .csv file
  - Output: _output/.../datasets/Tuned_ - preprocessed data

Script: _data_set_normalize.ipynb_

- Purpose: data normalization and smothing by average of 3 recent points
  - Input: _output/.../datasets/Tuned_ - preprocessed data
  - Output: _output/.../datasets/Normalized_ - normalized data 

### Predictions Generation

Script: _predict.ipynb_

- Purpose: trains the model 
  - Input: _output/.../datasets/Normalized_ - normalized normal data 
  - Output: _model_ - trained autoencoder model 
- Purpose: makes and vizualize timestamp level preditions
  - Input: _output/.../datasets/Normalized_ - normalized data with injected failures 
  - Output: _output/.../predictions_  
- Purpose: detects KPI level anomalies
  - Input: _output/.../datasets/Normalized_ - normalized data with injected failures
  - Output: _output/.../anomalies_lists_services_only_
  - Output: _output/.../anomalies_lists_services_only_sliding_window_file_path_
- Purpose: localize failures
  - Input: _output/.../anomalies_lists_services_only_sliding_window_file_path_ 
  - Output: _output/.../localisations_by_reconstruction_error_sliding_window_file_path_

### Results Visualization

Script: _results.ipynb_

- Purpose: vizualization of the results
  - Input: _output/.../predictions_
  - Input: _output/.../localisations_re_sliding_window_
  - Output: _visualization_ 

