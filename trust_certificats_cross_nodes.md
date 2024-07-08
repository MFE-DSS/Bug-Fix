# Issue: Using a Personal API Key for Deployer Node in Dataiku DSS for Creating Projects on Automation Node

## Summary
As we know, when deploying a design instance project to automation, its necessary to manually create the project in the deployer in order to be able to send the bundle. We're going to manage this in a python script to perform these operations remotely and automate them.
Dataiku recommends using a personal API key for the Deployer Node from a user with permissions to create projects on the Automation Node. Initially, global API keys were configured and project owner in deployment isnt existing in other instance, leading to potential issues.

### User Permissions
- The user must exist on the Automation Node to create a project from the Deployer.
- When using deployment hooks, an authorized user must be assigned to launch a kernel on the Deployer.

### SSL Configuration
- Look at Doc for : Custom SSL certificate verification.
- Look at doc for : Nodes are configured to trust all certificates.

### Deployment Script for Governance Policy Project

The following script is executed via a Dataiku scenario and is responsible for creating the project and deploying a specific bundle on the Automation Node.

> **Note**: This script is conditioned to run only for the first deployment of the project on the Automation Node.

### Script

```python
import dataiku
import dataikuapi
from dataiku.scenario import Scenario

# Get the handle of the currently running scenario
sc = Scenario()

# Get all variables (global, project-specific, and scenario-specific)
variables = sc.get_all_variables()

# Initialize the Dataiku client for the Design Node
design_client = dataiku.api_client()
design_client._session.verify = False

# Retrieve deployment information from global variables
host_deployer = design_client.get_global_variables()["your_host_deployer"]
keymac_deployer = design_client.get_global_variables()["your_keymac_deployer"]

# Configure the Dataiku client for the Deployer Node
dataiku.set_remote_dss(host_deployer, keymac_deployer, no_check_certificate=True)
deployer_client = dataiku.api_client()
deployer_client._session.verify = False

# Retrieve scenario trigger parameters
scenario_trigger_params = variables.get('scenarioTriggerParams', {})
project_key = scenario_trigger_params.get('project_key', 'default_value')
bundle_id = scenario_trigger_params.get('bundle_id', 'default_value')

project_deployer = deployer_client.get_projectdeployer()

# Create the deployment
infra = "automation-int"
deployment_id = f'{project_key}-{bundle_id}-on-{infra}'
deployment = project_deployer.create_deployment(deployment_id, project_key, infra, bundle_id)

# Start the deployment update
update = deployment.start_update()
update.wait_for_result()
```
### Script Steps
Connect to the Design Node: Initialize the API client to connect to the Design Node.
Retrieve Global Variables: Use global variables to get the information about the Deployer Node.
Configure Deployer Client: Initialize the API client to connect to the Deployer Node.
Retrieve Scenario Parameters: Extract the necessary parameters from the current scenario.
Create Deployment: Create a specific deployment on the Automation Node using the provided bundle.
Update Deployment: Start and wait for the completion of the deployment update.
Points of Attention : Ensure the API key used in the Deployer Node configuration is a personal API key from a user with sufficient permissions on the Automation Node. It does not need to belong to the user performing the deployment but must have the required permissions to create projects.
