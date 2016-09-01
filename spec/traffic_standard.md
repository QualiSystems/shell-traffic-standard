# Traffic Shell Standard

#### Version 1.0.0


## Introduction
The Traffic Shell Standard is a project used to define a standard for all Traffic Shells that integrate with CloudShell, both for virtual and physical traffic generators.
The standard defines the Shell’s data model, commands and a set of guidelines that should be followed in traffic Shells development and customization.


## Revision History

Version | Date | Notes
--- | --- | ---
1.0.0 | 2016-09-01 | First release of the Traffic Shell Standard


## Definitions
### Granularity
The Traffic Shell Standard defines two separate shells - a Shell for the Traffic Generator and a Shell for the Traffic Controller. The Traffic Generator Shell supports all traffic generator devices of the same Vendor, for example "Ixia Traffic Generator Shell". The Traffic Controller Shell supports traffic controller of a specific vendor and application, for example "Ixia IxLoad Traffic Controller Shell" or "Spirent Test Center Controller Shell".

### Specific Versions Certification
Each released Shell should have a list of certified versions. Version certification can be done only by Quali’s engineering, and it includes validatation that all the Shell’s capabilities are working for a specific version.
The Shell should also work for non-certified versions unless specified differently in the Shell's release notes, and in case some gaps are found a new version of the Shell will be released with the gaps addressed and the version certified.

### Generic Data Model
All traffic generator and traffic controller Shells share the same generic data model, except the model of the root level which is different per each Shell. The data model shouldn’t be modified.
The attributes associated with those generic models are also shared by all traffic generator Shells and their values are populated by the driver. It is allowed to add custom attributes only to the root level model, and it isn’t allowed to remove attributes from any level.

### Versioning
The traffic generator and traffic controller Shell version follows Semantic Versioning Guidelines (see details in http://semver.org). In short, the version structure is Major.Minor.Patch, for example “1.0.2”. A Patch version is promoted when making backward-compatibile bug fixes, a Minor version is promoted when adding functionality in a backwards-compatible manner and a  Major version is promoted when making any backwards incompatible change. The standard itself follows the same guidelines.

### Dependencies
In case the traffic generator or traffic controller Shell is written in Python and has dependencies to Python packages (that follow Semantic Versioning Guidelines) the dependency should be to a range of Patch versions, for example to “cloudshell-traffic-ixia 2.1.X”.
The dependency to cloudshell-automation-api should be to the latest Patch version (cloudshell-automation-api package version is of the format “CloudShell_Version.X”, for example 7.0.X”).



## Data Model
### Families & Models
The traffic generator Shell standard supports both physical and virtual traffic generators and controllers and includes modelling for both a traffic application (controller) and a traffic generator device (chassis).

The traffic generator is modeled per vendor; for example a traffic generator server will be modeled as "Ixia Traffic Generator" or "Spirent Traffic Generator". A specific traffic generator can be associated with multiple traffic controllers and vice versa; for example an "Ixia Traffic Generator" can be associated with multiple "IxNetwork" and "IxLoad" traffic controllers, and an "IxNetwork" traffic controller can be associated with multiple "Ixia Traffic Geneartor" traffic generator devices. The association between a traffic generator and a traffic controller is done via an attribute named "Controller Group", and multiple traffic controllers are part of the same controller group in case those controllers can be used to access the same traffic generator devices.

### User flows and Shell configuration

The expected user flow is adding traffic ports to a blueprint (via the inventory or abstract template) and reserving. An orchestration script can then query the sandbox, select the traffic controller to use out of the those defined in the controller group associated with the ports' root server, add that controller to the sandbox (as Shared) and launch commands such as load test configuration, run/stop test etc. using the Traffic Controller shell. All the information required for the controller's shell to work should be available in the controller and traffic generator attributes.

The expected administrator flow is to load both the traffic generators and controllers into CloudShell using the shell template and Autoload command. The traffic client used by the Controller is expected to be installed on the Execution Server(s) as a pre-requisited for the controller shell and the path to its installation should be defined in the attribute "Client Install Path" on the Traffic Controller resource. In case there is an external server (such as Spirent Test Center Lab Server or IxNetwork API Server) the address of the Traffic Controller should point to this server and the server's TCP Port (if not default) should be defined in the "TCP Port" attribute of the Traffic Controller. Else the address of the Traffic Controller should be defined as "NA".

#### Traffic Generator Data Model
- Server
 - Module
    - Port
    - Port Group
      - Port
 - Power Port
 - Port
 - Port Group
   - Port

#### Traffic Controller Data Model
- Traffic Controller
 - (no sub resources)

#### Virtual Traffic Generator Data Model
- Generic App Family
 - Module
    - Port
 - Power Port
 - Port

#### Virtual Traffic Controller Data Model
- Generic App Family
 - (no sub resources)

##### Example:
Physical:
- Family: Server, Model: Ixia Traffic Generator
 - Family: Module, Model: Generic Traffic Module
    - Family: Port, Model: Generic Traffic Port
    - Family: Port Group, Model: Generic Port Group
      - Family: Port, Model: Generic Traffic Port
 - Family: Power Port, Model: Generic Power Port
 - Family: Port , Model: Generic Traffic Port
 - Family: Port Group, Model: Generic Port Group
   - Family: Port, Model: Generic Traffic Port

- Family: Controller, Model: Ixia IxNetwork Traffic Controller

Virtual:
- Family: Generic App Family, Model: BreakingPoint Traffic Generator
 - Family: Module, Model: Generic Traffic Module
    - Family: Port, Model: Generic Virtual Traffic Port
  - Family: Port , Model: Generic Virtual Traffic Port

- Family: Generic App Family, Model: BreakingPoint Traffic Controller

#### Family Rules

Family | Rules
--- | ---
Server | Searchable
Traffic Controller | Searchable
Module | Searchable
Port | Searchable, Connectable, Locked By Default
Power Port | Searchable, Connectable, Locked By Default
Port Group | Searchable, Acts As Group

#### Resource Name and Address
Family | Model | Resource Name | Resource Address
--- | --- | --- | ---
Server | [Vendor] Traffic Generator | (user defined) | (user defined - IP)
Traffic Controller | [Vendor/Application] Traffic Controller | (user defined) | (user defined - IP or "NA")
Generic App Family | [Vendor] Traffic Generator | (user defined) | (user defined - IP)
Generic App Family | [Vendor/Application] Traffic Controller | (user defined) | (user defined - IP)
Module | Generic Traffic Module | Module[ID] | M[ID]
Port | Generic Traffic Port | The name of the port as appears in the device. Any “/” character is replaced with “-“, spaces trimmed. | P[ID]
Port | Generic Virtual Traffic Port | The name of the port as appears in the device. Any “/” character is replaced with “-“, spaces trimmed. | P[ID]
Power Port | Generic Power Port | PP[ContainerID][ID] | PP[ContainerID][ID]
Port Group | Generic Traffic Grouped Port | PG[ID] | PG[ID]

Notes:
 1. The [ID] for each sub-resource is taken from the device itself (corresponds to the names defined in the device). For the Port Group the [ID] is automatically generated.
 2. The address of a physical Traffic Controller should be the IP of the server in case an external server is configured (can be relevant for Ixia IxNetwork, Spirent Test Center Lab Server and Ixia IxN2X). In case there is no external server the address should be "NA" which means there is no external server but only a client application installed on the Execution Servers.
 3. Grouped ports should be modeled under a "Port Group". Once modeled as such reserving one of the ports under a specific port group will add all other ports under the same port group to the same Sandbox.
 4. The "Generic App Family" used to model virtual traffic generators and controllers is defined in the Deployed App Shell Standard - https://github.com/QualiSystems/shell-deployedapp-standard/blob/master/spec/deployed_app_standard.md


### Attributes
#### Guidelines
- Attributes which aren’t relevant to a device won’t be populated by the driver.
- All attributes which aren't user-input should have the rule "Admin only" enabled.
- The attribute rules are as follows - all attributes which are user input should have the rule "Configuration" enabled, all attributes which aren't user input should have the rules "Settings" and "Available For Abstract Resources" enabled.
- It is possible to customize the attribute rules selection after importing the Shell to CloudShell.
- Attributes shouldn’t be removed.
- Custom attributes should be added only to the root level model.
- All attributes are of type String unless mentioned otherwise

##### [Vendor] Traffic Generator (virtual or physical)

Attribute Name | Data Type | User input? | Description
--- | --- | --- | ---
Version | String | No | The firmware version of the resource.
Model | String | No | The device model. This information is typically used for abstract resource filtering.
Power Management | Boolean | Yes | Used by the power management orchestration, if enabled, to determine whether to automatically manage the device power status. Enabled by default.
Server Description | String | No | The full description of the server. Usually includes the OS, exact firmware version and additional characteritics of the device.
Controller Group | String | Yes | The name of the controller group that the traffic generator is associated with or the controller application is part of.
Supported Applications | String | Yes | Comma-separated list of traffic applications supported by this traffic generator. For example "IxLoad,IxNetwork".

##### Generic Traffic Controller (physical)

Attribute Name | Data Type | User input? | Description
--- | --- | --- | ---
Controller Group | String | Yes | The name of the controller group that the traffic generator is associated with or the controller application is part of.
Client Install Path | String | Yes | The path in which the traffic client is installed on the Execution Server. For example "C:/Program Files (x86)/Ixia/IxLoad/5.10-GA".
TCP Port | Numeric | Yes | The TCP port of the traffic server. Relevant only in case an external server is configured. Default TCP port should be used if kept empty.

##### Generic Virtual Traffic Controller (virtual)

Attribute Name | Data Type | User input? | Description
--- | --- | --- | ---
Controller Group | String | Yes | The name of the controller group that the traffic generator is associated with or the controller application is part of.

#####  Generic Traffic Module

Attribute Name | Data Type | User input? | Description
--- | --- | --- | ---
Model | String | No | The device model. This information is typically used for abstract resource filtering.

##### Generic Traffic Port

Attribute Name | Data Type | User input? | Description
--- | --- | --- | ---
Model | String | No | The device model. This information is typically used for abstract resource filtering.
Media Type | String | No | Interface media type. Possible values are Fiber and/or Copper (comma-separated).
Supported Speed | String | No | Speed supported by the interface, comma-separated.
Logical Name | String | Yes | The port's logical name in the test configuration. If kept emtpy automatic allocation will apply.

##### Generic Virtual Traffic Port

Attribute Name | Data Type | User input? | Description
--- | --- | --- | ---
Model | String | No | The device model. This information is typically used for abstract resource filtering.
Logical Name | String | Yes | The port's logical name in the test configuration. If kept emtpy automatic allocation will apply.
Requested vNIC Name | String | Yes | The name or number of the network interface represented by this port. If kept empty an available inetrafce will be selected on connection creation.

#####  Generic Power Port

Attribute Name | Data Type | User input? | Description
--- | --- | --- | ---
Model | String | No | The device model. This information is typically used for abstract resource filtering.
Serial Number | String | No |
Version | String | No | The firmware version of the resource.
Port Description | String | No | The description of the port as configured in the device.

#####  Generic Port Group
No attributes.

## Commands
Below is a list of all the commands that will be part of the standard Shell, their names and interfaces. Each traffic generator Shell that will be released by Quali’s engineering will include implementation for all those commands.

When creating a new shell according to the standard it is OK not to implement all commands and/or implement additional command, but a command with a functionality that fits one of the predefined list commands should be implemented according to the standard.

Command outputs: On failure an exception containing the error will be thrown and the command will be shown as failed. A failure is defined as any scenario in which the command didn’t complete its expected behavior, regardless if the issue originates from the command’s input, device or the command infrastructure itself. On success the command will just return as passed with no output. The “Autoload” command has a special output on success that CloudShell reads when building the resource hierarchy.

### Traffic Generator Commands
Below is a list of all the commands associated with the Traffic Generator root resource (Family = Server).

#### Autoload
```python
def get_inventory (self, context)
```

###### Description
This function queries the device, discovers it's specification and structure and autoload it into CloudShell. When new resource is created Cloudshell asks the user to specify some user input (i.e address and credentials if needed) and then it calls the get_inventory function.

###### Display Name
System command, it has no display name

###### Parameters
Input / Output | Parameter | Alias | Data Type | Required | Description
--- | --- | --- | --- | --- | ---
Input | context | - | object | system parameter | object of type AutoLoadCommandContext which includes API connectivity details and the details of the resource including attributes that the user entered during the resource creation. | - | -
Output | AutoLoadDetails | - | object | Yes | object of type AutoLoadDetails with the discovered resource structure and attributes. | - | -

### Traffic Controller Commands
The Traffic Controller doesn't have commands associated with it as part of this traffic standard. It is possible to extend the Traffic Controller Shell (which consists only of data model) with custom commands such as load_configuration, run_test, stop_test, send_arp, preview_configuration, get_statistics or any other custom commands according to the use case.

It is suggested to launch the commands associated with the Traffic Controller via orchestration scripts. The orchestration scripts will query the Sandbox, find the available traffic ports, decide on the right controller according to the information available on the ports and their root server, and launch the Traffic Controller's commands such as load test configuration and run traffic using the traffic ports available in the Sandbox.

Note that the Shell's commands should be launched in Execution Servers that have the relevant traffic client (such as "IxLoad" or "Spirent Test Center") installed. The path to the traffic client can be found in the "Client Install Path" attribute on the Traffic Controller resource. In case a traffic server exists (relevant in Spirent Test Center Lab Server, Ixia IxNetwork API Server or Ixia IxN2X) than the address of the Traffic Controller will hold the address of the server and the attribute "TCP Port" on the Traffic Controller will hold the server's TCP port.

Examples for common traffic controller commands:
#### Load Configuration
```python
def load_configuration(test_location)
```

#### Run Test
```python
def run_test()
```

#### Stop Test
```python
def stop_test()
```

#### Preview Configuration
```python
def preview_configuration()
```

#### Send Arp
```python
def send_arp()
```

#### Get Statistics
```python
def get_statistics()
```
