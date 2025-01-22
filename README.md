# README

This repository is part of my master's dissertation titled **"Leveraging AI for Kubernetes System Administration."** It includes three Jupyter notebooks along with all test results and cluster dumps.

## Notebooks Overview

### Prompt Engineering

The `prompt_engineering.ipynb` notebook contains the code used to request various LLMs to generate Kubernetes Infrastructure as Code (IaC) manifests for deploying WordPress with a MySQL backend. The following prompt engineering techniques were tested, each with both basic and detailed baseline prompts:

- **Zero-shot**
- **Zero-shot role-based**
- **Zero-shot Chain-of-Thought**
  - Using a human-designed reasoning prompt: *"Let's think step by step."*
  - Using an APE-discovered reasoning prompt: *"Let's work this out in a step-by-step way to be sure we have the right answer."*
- **Meta prompting**
- **Meta prompting for prompting tasks**
- **Simple Tree-of-Thoughts implementation** in the form of ToT-style prompts

### Testing Script

The `testing.ipynb` notebook contains the testing script used to evaluate the generated manifests, along with output logs from all executed tests. 

#### Key Functions:

- **`create_yaml`**: Generates a YAML file from the LLM's response and verifies if the response contains valid YAML code.

- **`kubeconform`**: Validates the manifest against Kubernetes OpenAPI specs and resource schemas.

- **`encode_secrets`**: Encodes secrets that are not base64 encoded, as required by Kubernetes.

- **`add_probes_to_container`**: Adds liveness and readiness probes to the WordPress and MySQL containers to assess the functionality of the generated manifest.

```python
liveness_probe_wordpress = {
	    'tcpSocket': {
	        'port': 80
	    },
	    'initialDelaySeconds': 30,
	    'timeoutSeconds': 5,
	    'periodSeconds': 10,
	    'failureThreshold': 3
}

readiness_probe_wordpress = {
    'httpGet': {
        'path': "/wp-admin/install.php",
        'port': 80
    },
    'initialDelaySeconds': 30,
    'timeoutSeconds': 5,
    'periodSeconds': 5
}

liveness_probe_mysql = {
    'tcpSocket': {
        'port': 3306
    },
    'initialDelaySeconds': 20,
    'timeoutSeconds': 5,
    'periodSeconds': 10,
    'failureThreshold': 3
}

readiness_probe_mysql = {
    'exec': {
        'command': ["mysqladmin", "ping", "-h", "localhost"]
    },
    'initialDelaySeconds': 20,
    'timeoutSeconds': 5,
    'periodSeconds': 5

}
```

- **`polaris_audit`**: Conducts a Polaris audit on the generated manifest, scoring it based on Kubernetes best practices in three areas: Security, Efficiency, and Reliability.

- **`create_cluster` and `delete_cluster`**: Functions to create and delete a fresh kind cluster using the configuration file `kind-cluster-config-mirror.yaml`. The cluster is set up with a Docker Hub mirror registry (`mirror_registry.yaml`) to avoid Docker Hub's pull rate limits. Contexts are utilized to run multiple kind clusters simultaneously.

- **`deploy_manifest`**: Applies the generated manifest to the kind cluster using the `create_from_yaml` function from the Kubernetes Python client library's utils module.

- **`cluster_dump`**: Waits for five minutes to allow the cluster to reconcile its desired state, then takes a detailed cluster dump.

- **`k8sgpt_analyze`**: Scans the cluster using the k8sGPT tool to diagnose and triage issues in simple English.

## Results

Test results are organized in the following directory structure:

```
ape_prompt/
├── response-0/
│   ├── dump/
│   │   ├── default/
│   │   ├── kube-system/
│   │   ├── (custom namespace)
│   │   └── nodes.json
│   ├── conform.json
│   ├── deploy_manifest.yaml
│   ├── k8sgpt.json
│   ├── manifest.yaml
│   ├── polaris.json
│   └── testing.json
├── response-1/
├── response-2/
├── .../
├── intermediate/
│   ├── response-0.yaml
│   ├── response-1.yaml
│   ├── response-2.yaml
│   └── ...
├── response-0.yaml
├── response-1.yaml
├── response-2.yaml
└── ...
```

- **`response-X.yaml`**: Contains the original response from the LLM.
- **`intermediate/`**: [Conditional] Contains intermediate responses from the LLM.
- **`response-X/`**: This directory includes all results and processed files created during the testing procedure.
  - **`dump/`**: Contains the cluster dump.
  - **`conform.json`**: Results from Kubeconform.
  - **`deploy_manifest.yaml`**: The manifest deployed to the kind cluster during the deployment phase.
  - **`k8sgpt.json`**: Results from k8sGPT analysis.
  - **`manifest.yaml`**: Extracted manifest from the extraction phase.
  - **`polaris.json`**: Results from the Polaris audit.
  - **`testing.json`**: Summary of the testing procedure.
