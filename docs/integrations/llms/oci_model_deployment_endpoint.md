---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/llms/oci_model_deployment_endpoint.ipynb
---
# OCI Data Science Model Deployment Endpoint

[OCI Data Science](https://docs.oracle.com/en-us/iaas/data-science/using/home.htm) is a fully managed and serverless platform for data science teams to build, train, and manage machine learning models in the Oracle Cloud Infrastructure.

This notebooks goes over how to use an LLM hosted on a [OCI Data Science Model Deployment](https://docs.oracle.com/en-us/iaas/data-science/using/model-dep-about.htm).

To authenticate, [oracle-ads](https://accelerated-data-science.readthedocs.io/en/latest/user_guide/cli/authentication.html) has been used to automatically load credentials for invoking endpoint.


```python
!pip3 install oracle-ads
```

## Prerequisite

### Deploy model
Check [Oracle GitHub samples repository](https://github.com/oracle-samples/oci-data-science-ai-samples/tree/main/model-deployment/containers/llama2) on how to deploy your llm on OCI Data Science Model deployment.

### Policies
Make sure to have the required [policies](https://docs.oracle.com/en-us/iaas/data-science/using/model-dep-policies-auth.htm#model_dep_policies_auth__predict-endpoint) to access the OCI Data Science Model Deployment endpoint.

## Set up

### vLLM
After having deployed model, you have to set up following required parameters of the `OCIModelDeploymentVLLM` call:

- **`endpoint`**: The model HTTP endpoint from the deployed model, e.g. `https://<MD_OCID>/predict`. 
- **`model`**: The location of the model.

### Text generation inference (TGI)
You have to set up following required parameters of the `OCIModelDeploymentTGI` call:

- **`endpoint`**: The model HTTP endpoint from the deployed model, e.g. `https://<MD_OCID>/predict`. 

### Authentication

You can set authentication through either ads or environment variables. When you are working in OCI Data Science Notebook Session, you can leverage resource principal to access other OCI resources. Check out [here](https://accelerated-data-science.readthedocs.io/en/latest/user_guide/cli/authentication.html) to see more options. 

## Example


```python
import ads
from langchain_community.llms import OCIModelDeploymentVLLM

# Set authentication through ads
# Use resource principal are operating within a
# OCI service that has resource principal based
# authentication configured
ads.set_auth("resource_principal")

# Create an instance of OCI Model Deployment Endpoint
# Replace the endpoint uri and model name with your own
llm = OCIModelDeploymentVLLM(endpoint="https://<MD_OCID>/predict", model="model_name")

# Run the LLM
llm.invoke("Who is the first president of United States?")
```


```python
import os

from langchain_community.llms import OCIModelDeploymentTGI

# Set authentication through environment variables
# Use API Key setup when you are working from a local
# workstation or on platform which does not support
# resource principals.
os.environ["OCI_IAM_TYPE"] = "api_key"
os.environ["OCI_CONFIG_PROFILE"] = "default"
os.environ["OCI_CONFIG_LOCATION"] = "~/.oci"

# Set endpoint through environment variables
# Replace the endpoint uri with your own
os.environ["OCI_LLM_ENDPOINT"] = "https://<MD_OCID>/predict"

# Create an instance of OCI Model Deployment Endpoint
llm = OCIModelDeploymentTGI()

# Run the LLM
llm.invoke("Who is the first president of United States?")
```


## Related

- LLM [conceptual guide](/docs/concepts/#llms)
- LLM [how-to guides](/docs/how_to/#llms)
