# Quickstart

This file is made to enable you to set up your model for hyperparameter search using the 
[cluster-hyperopt repository](https://github.com/aimat-lab/cluster-hyperopt) .

This Quickstart is separated into 3 Sections:

- Section 1: Preconditions

- Section 2: Creating a wrapper function, which executes the training and evaluation of your model

- Section 3: Installation of cluster-hyperopt and creation of necessary files


A more detailed Tutorial can be found in the Tutorial.md file.


### Section 1: Preconditions
You need:
- access to SigOpt
- access to BWUniCluster or HoreKA
- a GitHub repo for your project/model
- a conda environment for your model on the BWUniCluster/HoreKA
- a training and an evaluation function for your model


### Section 2:
create a wrapper function (lets call it "run_hyperopt(config)", but you can name it how you want), 
that trains and evaluates the model.
  - Input: python dict
  - Output: The loss/score/metric of the evaluated model
  - Structure:
    - extract hyperparams from the input dict
    - run the training algorithm of your model using the extracted hyperparameters
    - run the evaluation algorithm of your model
    - return the loss/score/metric of your model
  - Location: In an arbitrary python file in your github repo. Make sure, that it is uploaded to github.
  
The python dict mentioned as input has the following structure:
We added some example hyperparameters.
```
dict: {
    'suggestion': {
        "hyperparameter_x": value
        "bootstrap": 1,
        "criterion": "gini",
        "max_depth": 5,
        "max_features": "log2",
        "n_estimators": 2
    },
    'dataset': 'path/to/dataset/'
    'output_path': path/to/output/dir
}    
```

example structure of "run_hyperopt(config)":
```python
    def run_hyperopt(config):
        # extract hyperparams. Example:
        max_depth = config["suggestion"]["max_depth"]
        
        #run the training algorithm. Example:
        model = train(max_depth)
        
        #evaluiate the model. Example:
        loss = evaluate(model)
        
        #return loss/score/metric
        return loss

```
### Section 3:
- Installation of cluster-hyperopt:
  - download the repository [cluster-hyperopt](https://github.com/aimat-lab/cluster-hyperopt) to a
  location of your choice on the BWUniCluster or HoreKA.
  - you can use an existing conda environment or a new one to install cluster-hyperopt
  - open a shell in the topmost directory of your local cluster-hyperopt repository
  - activate the conda environment of your choice and execute "pip install -e ." 
  This will install cluster-hyperopt into the active environment.



- Creating the environment.yaml file:
  - Is needed to create a conda environment with all dependencies your model needs
  - Will be used by cluster hyperopt to create an environment where it will execute your model.
  - needs to be in the topmost folder of your project repo.
  - can be created by opening a shell, activating the model env. and executing 
  ```conda env export > environment.yaml```. (Note that the environment.yaml will be
  created in the directory, in which you created the shell. You might need to move it)
  - commit the changes inside your repository and push it to github
  
  
- Creation of the config file:
  - arbitrary place on the server (BWUniCluster/HoreKA)
  - includes most information needed for hyperparameter optimization:
  - see "config_file_variables.md" for information about every variable
  - example file:
    ```yaml
    model:
      entry_point: "main.py" # The python file name that includes the function for evaluating the suggestions
      function_name: "run_hyperopt" # the function that executes the training and evaluation
    data_options:
      dataset_path: "/path/to/dataset" # can be empty 
    git_options:
      git_uri: "git@github.com:u-adrian/Tutorial.git"
      branch: "main" # Either branch or version can be used. Using the option version allows to load specific tags
    experiment:
      cluster: "horeka"  # Either "bwunicluster" or "horeka"
      number_chain_jobs: 4 # How many times should a job - the suggestion evaluation - be chained together. It is used to
                           # cirumvent the problem of time outs in the cluster
      observation_budget: 60 # Max number of trials
      parallel_bandwidth: 4 # Number of parallel evaluations
      multimetric_experiment: false
      conda_env: name_of_env # name of the conda env used for the model 
    parameters:
      - name: max_depth
        type: int
        bounds:
          min: 1
          max: 10
      - name: n_estimators
        type: int
        bounds:
          min: 1
          max: 10
      - name: bootstrap
        type: int
        grid:
          - 0
          - 1
      - name: max_features
        type: categorical
        categorical_values:
          - 'sqrt'
          - 'log2'
      - name: criterion
        type: categorical
        categorical_values:
          - 'gini'
          - 'entropy'
    metrics:
      - name: accuracy
        objective: maximize
        strategy: optimize
    sbatch_options:
      partition: "dev_gpu_4"
    sigopt_options:
      dev_run: false # If the dev api of sigopt should be used to get the suggestions
      project_id: "test_project"
      experiment_name: "tutorial_project"
      client_id: 11949
    ```

- SigOpt Token file
  - content of the file:
    ```
    SIGOPT_TOKEN=**********************************************
    SIGOPT_DEV_TOKEN=**********************************************
    ```
    You can find your SIGOPT_TOKEN and SIGOPT_DEV_TOKEN on the 
  SIGOPT Website: https://app.sigopt.com/tokens/info under the menu points: 
    - \<Your Username\>
      - "API Tokens"
  - name of the file: arbitrary
  - arbitrary location on the bwunicluster or horeka
  - How to make it accessible to cluster-hyperopt: ```export SIGOPT_ENV_FILE=path_to_this_file```

### Section 4: Run HyperOpt
- Open shell in arbitrary folder (but on the cluster).
- activate the conda env where cluster-hyperopt is installed
  ```bash
  conda activate test_tutorial
  ```
- execute cluster hyperopt:
  ```bash
  python /.../cluster-hyperopt/sigopt_hyperopt/hyperopt.py start --config_path=/path_to_config/config.yaml
  ```
