
# llama3.1-sft-chatbot

## License
This template is licensed under Apache 2.0 and contains the following components: 
* Llama3.1 [License](https://ai.meta.com/llama/license/)
* mlflow [Apache License 2.0](https://github.com/mlflow/mlflow/blob/master/LICENSE.txt)
* accelerate [Apache License 2.0](https://github.com/huggingface/accelerate/blob/main/LICENSE)
* bitsandbytes [MIT License](https://github.com/TimDettmers/bitsandbytes/blob/main/LICENSE)
* pytorch [Caffe 2](https://github.com/pytorch/pytorch/blob/main/LICENSE)
* datasets [Apache License 2.0](https://github.com/huggingface/datasets/blob/main/LICENSE)
* peft [Apache License 2.0](https://github.com/huggingface/peft/blob/main/LICENSE)
* streamlit [Apache License 2.0](https://github.com/streamlit/streamlit/blob/develop/LICENSE)
* transformers [Apache License 2.0](https://github.com/huggingface/transformers/blob/main/LICENSE)
* trl [Apache License 2.0](https://github.com/huggingface/trl/blob/main/LICENSE)


## About this project
This project shows how to fine tune Llama3.1-8b using supervised fine tuning. We use the `SFTTrainer` that's available in the `trl` library from Huggingface. In order to fine tune the model, we will load it in 4bit or 8bit, train a LoRA adapter and merge it back to the base `llama3.1-8b` model.

Fine-tuning a pre-trained LLM is a commonly used technique for solving NLP problems with machine learning. This is a typical transfer learning task where the final model is realised through a number of training phases:

1. The process typically begins with a pre-trained model, which is not task specific. This model is trained on a large corpora of unlabelled data 

2. The model undergoes a process of domain specific adaptive fine-tuning, which produces a new model with narrower focus or better alignment. This new model is better prepared to address domain-specific challenges as it is now closer to the expected distribution of the target data or responses the user expects. 

In this demo project we use the [mlabonne/guanaco-llama2-1k](https://huggingface.co/datasets/mlabonne/guanaco-llama2-1k) dataset, which provides 1000 samples of the excellent [timdettmers/openassistant-guanaco](https://huggingface.co/datasets/timdettmers/openassistant-guanaco) dataset, in two distinct notebooks processed to match Llama 3.1 propmt format. This dataset is used in conjuction with [Meta-Llama-3.1-8B](https://huggingface.co/NousResearch/Meta-Llama-3.1-8B), which we fine-tune for the purpose of building a conversational assistant.

The assets available in this project are:

*llama3_1_guanco.ipynb* - A notebook, illustrating the process of finetuning [Meta-Llama-3.1-8B](https://huggingface.co/NousResearch/Meta-Llama-3.1-8B) on the [mlabonne/guanaco-llama2-1k] (https://huggingface.co/datasets/mlabonne/guanaco-llama2-1k) dataset


*`model.py`* - A python file which is used to deploy the fine-tuned model as a Domino model API .

*`app.py`* - A Python file that loads the model, the LoRA adapter and allows for users to interact with the model as a Streamlit app .

*`app.sh`* - This script has launch instructions for the accompanying Streamlit app.

**Note : The first load of the app and model API takes time as it loads the fine tuned model into memory so please set the `Override request timeout` in the model API settings to an appropriate number**

## Prerequisites

Github Personal Access Token (PAT) Authentication in Domino [Domino PAT Setup](https://docs.dominodatalab.com/en/latest/user_guide/314004/import-git-repositories/)


This project requires the following [compute environments](https://docs.dominodatalab.com/en/latest/user_guide/f51038/environments/) to be present. Please ensure the "Automatically make compatible with Domino" checkbox is selected while creating the environment.

### Hardware Requirements 

This project will run on any Nvidia GPU with >=24GB of VRAM. Also ensure that the [`Workspace and Jobs Volume Size`] (https://docs.dominodatalab.com/en/latest/user_guide/0ea71e/change-the-workspace-volume-size/) setting of the workspace is set to 100GB as the model files and datasets can get large.

### Environment Requirements

**Environment Base**
***base image :*** `nvcr.io/nvidia/pytorch:23.10-py3`

### Model API Instructions 
The model.py file is used to deploy the API. The function to invoke is generate. 


When setting up the Model endpoint, the following request can be used as a sample test: 

{
  "data": {
    "prompt": "Why is the sky blue."
    "max_new_tokens": 250
  }
}

## Dockerfile
***Dockerfile instructions*** (not needed if using the Domino AI Hub template)
```
RUN pip install "transformers>=4.43.2" "peft>=0.7.1,!=0.11.0" "trl>=0.7.9,<0.9.0" \
"bitsandbytes==0.43.2" "accelerate>=0.26.1" "streamlit==1.37.0" \
"mlflow==2.12.0" "uWSGI==2.0.26" \
"Flask==3.0.3" "Flask-Compress==1.15" "Flask-Cors==4.0.1" "jsonify==0.5"

RUN pip uninstall --yes transformer-engine

```
***Pluggable Workspace Tools** 
```
jupyterlab:
  title: "JupyterLab"
  iconUrl: "/assets/images/workspace-logos/jupyterlab.svg"
  start: ["/opt/domino/bin/jupyterlab-start.sh"]
  httpProxy:
    internalPath: "/{{ownerUsername}}/{{projectName}}/{{sessionPathComponent}}/{{runId}}/{{#if pathToOpen}}tree/{{pathToOpen}}{{/if}}"
    port: 8888
    rewrite: false
    requireSubdomain: false
vscode:
 title: "vscode"
 iconUrl: "/assets/images/workspace-logos/vscode.svg"
 start: [ "/opt/domino/bin/vscode-start.sh" ]
 httpProxy:
    port: 8888
    requireSubdomain: false
```
Please change the value in `start` according to your Domino version. This repo has been tested with Domino 5.10.0 and 5.11.0 .
