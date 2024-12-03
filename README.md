# ThingsBoard Gateway Management Extension Distributions

The project offers distribution files for the [ThingsBoard Gateway Management Extension v1.0][1], designed to streamline gateway configuration and connector management using a modular, component-based approach.
## Steps to Run the Extension

### 1. Access Files
- Download the necessary files from this repository.

### 2. Upload the Resource
- In ThingsBoard, upload the `gateway-management-extension.js` file as a resource with the following details:
  - **Type**: JS Module
  - **Title**: Gateway
  - **Resource File**: Upload the downloaded `gateway-management-extension.js` file.

### 3. Add/Update Selected Widgets
- Upload or update the following files in the Widget Library:
  - **Gateway Configuration**: `gateway_configuration.json`
  - **Gateway Configuration for Single Device**: `gateway_configuration__single_device_.json`
  - **Gateway Connectors**: `gateway_connectors.json`
  - **Gateway Custom Statistics**: `gateway_custom_statistics.json`
  - **Gateway General Chart Statistics**: `gateway_general_chart_statistics.json`
  - **Gateway General Configuration**: `gateway_general_configuration.json`
  - **Gateway Logs**: `gateway_logs.json`
  - **Service RPC**: `service_rpc.json`

### 4. Update the Gateways Dashboard
- Upload or update the Gateways Dashboard using the `gateways_dashboard.json` dashboard file.

[1]: https://github.com/thingsboard/gateway-management-extensions
