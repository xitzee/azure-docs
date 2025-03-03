---
title: 'CLI (v2) command component YAML schema'
titleSuffix: Azure Machine Learning
description: Reference documentation for the CLI (v2) command component YAML schema.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference

author: lostmygithubaccount
ms.author: copeters
ms.date: 10/21/2021
ms.reviewer: laobri
---

# CLI (v2) command component YAML schema

The source JSON schema can be found at https://azuremlschemas.azureedge.net/latest/commandComponent.schema.json.

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

[!INCLUDE [schema note](../../includes/machine-learning-preview-old-json-schema-note.md)]

## YAML syntax

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `$schema` | string | The YAML schema. If you use the Azure Machine Learning VS Code extension to author the YAML file, including `$schema` at the top of your file enables you to invoke schema and resource completions. | | |
| `type` | const | The type of component. | `command` | `command` |
| `name` | string | **Required.** Name of the component. | | |
| `version` | string | Version of the component. If omitted, Azure ML will autogenerate a version. | | |
| `display_name` | string | Display name of the component in the studio UI. Can be non-unique within the workspace. | | |
| `description` | string | Description of the component. | | |
| `tags` | object | Dictionary of tags for the component. | | |
| `command` | string | **Required.** The command to execute. | | |
| `code.local_path` | string | Local path to the source code directory to be uploaded and used for the component. | | |
| `environment` | string or object | **Required.** The environment to use for the component. This value can be either a reference to an existing versioned environment in the workspace or an inline environment specification. <br><br> To reference an existing environment, use the `azureml:<environment-name>:<environment-version>` syntax. <br><br> To define an environment inline, follow the [Environment schema](reference-yaml-environment.md#yaml-syntax). Exclude the `name` and `version` properties as they are not supported for inline environments. | | |
| `distribution` | object | The distribution configuration for distributed training scenarios. One of [MpiConfiguration](#mpiconfiguration), [PyTorchConfiguration](#pytorchconfiguration), or [TensorFlowConfiguration](#tensorflowconfiguration). | | |
| `resources.instance_count` | integer | The number of nodes to use for the job. | | `1` |
| `inputs` | object | Dictionary of component inputs. The key is a name for the input within the context of the component and the value is the component input definition. <br><br> Inputs can be referenced in the `command` using the `${{ inputs.<input_name> }}` expression. | | |
| `inputs.<input_name>` | object | The component input definition. See [Component input](#component-input) for the set of configurable properties. | | |
| `outputs` | object | Dictionary of component outputs. The key is a name for the output within the context of the component and the value is the component output definition. <br><br> Outputs can be referenced in the `command` using the `${{ outputs.<output_name> }}` expression. | |
| `outputs.<output_name>` | object | The component output definition. See [Component output](#component-output) for the set of configurable properties. | |

### Distribution configurations

#### MpiConfiguration

| Key | Type | Description | Allowed values |
| --- | ---- | ----------- | -------------- |
| `type` | const | **Required.** Distribution type.  | `mpi` |
| `process_count_per_instance` | integer | **Required.** The number of processes per node to launch for the job.  | |

#### PyTorchConfiguration

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `type` | const | **Required.** Distribution type.  | `pytorch` | |
| `process_count_per_instance` | integer | The number of processes per node to launch for the job. | |  `1` |

#### TensorFlowConfiguration

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `type` | const | **Required.** Distribution type.  | `tensorflow` |
| `worker_count` | integer | The number of workers to launch for the job. | | Defaults to `resources.instance_count`. |
| `parameter_server_count` | integer | The number of parameter servers to launch for the job. | | `0` |

### Component input

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `type` | string | **Required.** The type of component input. <br><br> Use `type: path` if you want the runtime job input value to be a data URI or Azure ML dataset when the component is run. | `number`, `integer`, `boolean`, `string`, `path` | |
| `description` | string | Description of the input. | | |
| `default` | number, integer, boolean, or string | The default value for the input. | | |
| `optional` | boolean | Whether the input is required. | | `false` |
| `min` | integer or number | The minimum accepted value for the input. This field can only be specified if `type` field is `number` or `integer`. | |
| `max` | integer or number | The maximum accepted value for the input. This field can only be specified if `type` field is `number` or `integer`. | |
| `enum` | array | The list of allowed values for the input. Not applicable if `type` field is `boolean`.  | |

### Component output

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------------- |
| `type` | string | **Required.** The type of component output. | `path` | |
| `description` | string | Description of the output. | | |

## Remarks

The `az ml component` commands can be used for managing Azure Machine Learning components.

## Examples

Command component examples are available in the examples GitHub repository. Select examples for are shown below.

Examples are available in the [examples GitHub repository](https://github.com/Azure/azureml-examples/tree/main/cli/jobs/pipelines-with-components). Several are shown below.

## Hello world command component

:::code language="yaml" source="~/azureml-examples-main/cli/jobs/pipelines-with-components/basics/2a_basic_component/component.yml":::

## Next steps

- [Install and use the CLI (v2)](how-to-configure-cli.md)
