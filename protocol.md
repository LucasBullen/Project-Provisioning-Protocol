# Protocol
 * [Message Overview](#message-overview)
 * [Base Protocol](#base-protocol)
 * [Project Provisioning Protocol](#project-provisioning-protocol-outline)
 * [Protocol Objects](#protocol-objects)
 * [Example Interactions](#example-interactions)

## <a name="message-overview"></a>Message Overview
Create the connection between an interface and the provisioner

 * :arrow_right: [Initialize](#initialize)
 * :arrow_left: [Result](#initialize-result)

Validate a potential input for project provisioning and to notify the user of required changes

 * :arrow_right: [Validation](#validation)
 * :arrow_left: [Result](#validation-result)

Preview to the user what will happen if provisioning is requested

 * :arrow_right: [Preview](#preview)
 * :arrow_left: [Result](#preview-result)

Provision the project and notify the client of the new project location

 * :arrow_right: [Provision](#provision)
 * :arrow_left: [Result](#provision-result)

Instruct the client what actions will be required to provision the project, used when the server is unable to do so itself

 * :arrow_right: [ProvisionInstructions](#provision-instructions)
 * :arrow_left: [Result](#provision-instructions-result)

## <a name="base-protocol"></a>Base Protocol
The communication between the interface and the provisioner uses [JSON RCP v2.0](http://www.jsonrpc.org/specification). The descirption of the base protocol is that same as the one for the Language Server Protocol and can be found in [the "Base Protocol" section of the Language Server Protocol specification](https://github.com/Microsoft/language-server-protocol/blob/gh-pages/specification.md#base-protocol).

## <a name="project-provisioning-protocol-outline"></a>Project Provisioning Protocol
### <a name="initialize"></a>Initialize
> (Client -> Server) A request to begin the project provisioner parameter generation process, this opens and keeps open a connection to a server for future messages

Method: `projectProvisioning/initialize`
```typescript
Initialize {
	supportMarkdown: Boolean,
	allowFileCreation: Boolean
}
```
 - `supportMarkdown`: Informs the server if the client is capable of displaying markdown strings or if text that is intended to be displayed should be limited to raw Strings
 - `allowFileCreation`: Informs the server if it is able to generate the project itself and only inform the client where it has generated the project or if the files necessary for creation and their contents should be sent to the client on provisioning

#### <a name="initialize-result"></a>Initialize Result
> (Server -> Client) A response to inform the client what the server's capabilities are and what the default parameter values are
```typescript
InitializeResult {
    versionRequired: Boolean,
    validationSupported: Boolean,
    previewSupported: Boolean,
    templates: Template[],
    componentVersions: ComponentVersion[],
    defaultProvisioningParameters: ProvisioningParameters | null
}

```
 - `versionRequired`: Whether the server need the user to define a project version before creation
 - `validationSupported`: Whether the server is capable of responding to validation requests, informing the client if a set of parameters are valid
 - `previewSupported`: Whether the server is capable of previewing what will be done to provision the project
 - `templates`: A list of [Template](#template) objects defining the template projects available from this provisioner
 - `componentVersions`: A list of [ComponentVersion](#component-version) objects listing all the components required for this project creation and the versions available
 - `defaultProvisioningParameters`: Either a [ProvisioningParameters](#provisioning-parameters) object or null if there are no default values for the interface

### <a name="validation"></a>Validation
> (Client -> Server) A request to validate a set of parameters for project provisioning

Method: `projectProvisioning/validation`

The message contents is the [Provisioning-parameters](#provisioning-parameters) object.

#### <a name="validation-result"></a>Validation Result
> (Server -> Client) Whether the parameters are valid or the errors with the parameters
```typescript
ValidationResult {
	errorMessage: String | null,
	erroneousParameters: ErroneousParameter[]
}
```
`errorMessage`: If there is an error with the parameters, this is either an overarching error message or an error message that does not relate to any one specific parameter

`erroneousParameters`: If there is an error with the parameters, A list of [ErroneousParameter](#erroneous-parameter) objects listing parameter specific errors

If `errorMessage` is `null` and `erroneousParameters` is empty, then the given parameters are valid for provisioning

### <a name="preview"></a>Preview
> (Client -> Server) A request to preview the effects a set of parameters would have if used for project provisioning

Method: `projectProvisioning/preview`

The message contents is the [ProvisioningParameters](#provisioning-parameters) object.

#### <a name="preview-result"></a>Preview Result
> (Server -> Client) A preview of the effects a set of parameters would have if used for project provisioning or the errors stopping the preview from being generated
```typescript
PreviewResult {
	errorMessage: String | null,
	erroneousParameters: ErroneousParameter[],
	message: String | null
}
```
`errorMessage`: If there is an error with the parameters, this is either an overarching error message or an error message that does not relate to any one specific parameter

`erroneousParameters`: If there is an error with the parameters, A list of [ErroneousParameter](#erroneous-parameter) objects listing parameter specific errors

`message`: If `errorMessage` is `null` and `erroneousParameters` is empty, then a message explaining what will happen when the project is provisioned. If `supportMarkdown` in the [Initialize](#initialize) message was true then the message will be interpreted as markdown by the client

### <a name="provision"></a>Provision
> (Client -> Server) A request to provision a project with a set of given parameters.

Method: `projectProvisioning/provision`

The message contents is the [ProvisioningParameters](#provisioning-parameters) object.

#### <a name="provision-result"></a>Provision Result
> (Server -> Client) The resulting effects of provisioning the project or the errors stopping the provisioning
```typescript
ProvisionResult: {
	errorMessage: String | null,
	erroneousParameters: ErroneousParameter[],
	location: String,
	openFiles: String[]
}
```
 - `errorMessage`: If there is an error with the parameters, this is either an overarching error message or an error message that does not relate to any one specific parameter
 - `erroneousParameters`: If there is an error with the parameters, A list of [ErroneousParameter](#erroneous-parameter) objects listing parameter specific errors
 - `location`: The path to the location that the project was made
 - `openFiles`: A list of location relative paths to files that should be displayed to the user through the client, to show that the project has been provisioned

### <a name="provision-instructions"></a>Provision Instructions
> (Client -> Server)A request to generate a list of files and their contents to provision a project from a set of given parameters

Method: `projectProvisioning/provisionInstructions`

The message contents is the [ProvisioningParameters](#provisioning-parameters) object.

#### <a name="provision-instructions-result"></a>Provision Instructions Result
> (Server -> Client) A list of files and their contents to provision a project from a set of given parameters or the errors stopping this list from being generated
```typescript
ProvisionInstructionsResult: {
	errorMessage: String | null,
	erroneousParameters: ErroneousParameter[],
	message: String | null,
	name: String,
	newFiles: Instruction[],
	openFiles: String[]
}
```
 - `errorMessage`: If there is an error with the parameters, this is either an overarching error message or an error message that does not relate to any one specific parameter
 - `erroneousParameters`: If there is an error with the parameters, A list of [ErroneousParameter](#erroneous-parameter) objects listing parameter specific errors
 - `message`: An optional message explaining any extra steps the user will have to take to initializes the project from the files given. Is interpreted as markdown if `Initialize.supportMarkdown == True`
 - `name`: The name of the project to be created to be used in generaing the new porject's directory if needed
 - `newFiles`: A list of [Instruction](#instruction) objects that instruct the client on the files that need to be created for project provisioning
 - `openFiles`: A list of relative paths to files that should be displayed to the user through the client, to show that the project has been provisioned

## <a name="protocol-objects"></a>Protocol Objects

### <a name="template"></a>Template
```typescript
Template {
	id: String,
	title: String,
	caption: String | null,
	componentVersions: ComponentVersion[]
}
```
 - `id`: ID of the template project
 - `title`: UI name of the template project
 - `caption`: Short description of the template to be used by the UI, such as a tooltip
 - `componentVersions`: A list of [ComponentVersion](#component-version) objects listing all the components required for this template creation and the versions available

Used by:
 - [InitializeResult](#initialize-result)

### <a name="component-version"></a>Component Version
```typescript
ComponentVersion {
	id: String,
	title: String,
	caption: String | null,
	versions: Version[]
}
```
 - `id`: ID of the component
 - `title`: UI name of the component that requires a version selection
 - `caption`: Short description of the component to be used by the UI, such as a tooltip
 - `versions`: A list of [Version](#version) objects listing all the available versions of this component

Used by:
 - [Template](#template)
 - [InitializeResult](#initialize-result)

### <a name="version"></a>Version
```typescript
Version {
	id: String,
	title: String,
	caption: String | null
}
```
 - `id`: ID of the version
 - `title`: UI name of the version
 - `caption`: Short description of the version to be used by the UI, such as a tooltip

Used by:
 - [ComponentVersion](#component-version)

### <a name="provisioning-parameters"></a>Provisioning Parameters
```typescript
ProvisioningParameters {
	name: String,
	location: String | null, //Is always null if InitializeRequest->allowFileCreation == false
	version: String | null,
	templateSelection: TemplateSelection | null,
	componentVersionSelections: ComponentVersionSelection[ ],
}
```
 - `name`: Name of the project to be created
 - `location`: Relative path to the location to provision the project
 - `version`: Version to define the project at
 - `templateSelection`: A [TemplateSelection](#template-selection) object to use during provisioning
 - `componentVersionSelections`: A list of [ComponentVersionSelection](#component-version-selection) objects to define which versions of each component will be used during provisioning

Used by:
 - [InitializeResult](#initialize-result)
 - [Validation](#validation)
 - [Preview](#preview)
 - [Provision](#provision)
 - [ProvisionInstructions](#provision-instructions)

### <a name="component-version-selection"></a>Component Version Selection
```typescript
ComponentVersionSelection {
	id: String,
	versionId: String | null
}
```
 - `id`: ID of the component requiring a version selection
 - `versionId`: ID of the component's version

Used by:
 - [ProvisioningParameters](#provisioning-parameters)

### <a name="template-selection"></a>Template Selection
```typescript
TemplateSelection {
	id: String,
	componentVersions: ComponentVersion[]
}
```
 - `id`: ID of the template
 - `versionId`: A list of [ComponentVersion](#component-version) objects listing all the component versions selected for this template

Used by:
 - [ProvisioningParameters](#provisioning-parameters)

### <a name="erroneous-parameter"></a>Erroneous Parameter
```typescript
ErroneousParameter {
	parameterType: String, // (name, location, version, template, templateComponentVersion, componentVersion)
	message: String,
	componentVersionId: String | null
}
```
 - `parameterType`: Name of the parameter which is causing the error
 - `message`: UI message describing the error
 - `componentVersionId`: ID of the component version causing the error, only necessary if it is a component version causing the error

Used by:
 - [ValidationResult](#validation-result)
 - [PreviewResult](#preview-result)
 - [ProvisionResult](#provision-result)
 - [ProvisionInstructionsResult](#provision-instructions-result)

### <a name="instruction"></a>Instruction
```typescript
Instruction: {
	path: String,
	content: String
}
```
 - `path`: A relative path including the file name for the file to be created
 - `content`: The content of the file to be created

Used by:
 - [ProvisionInstructionsResult](#provision-instructions-result)

## <a name="example-interactions"></a>Example Interactions

### Rust Project in an IDE
The user opens the IDE and wishes to create a new Rust project. They then open the New Project Wizard.

**Method**: `projectProvisioning/initialize`
**Direction**: `Client -> Server`
**Message**:
```typescript
{
	supportMarkdown: true,
	allowFileCreation: true
}
```

**Method**: `projectProvisioning/initialize`
**Direction**: `Server -> Client`
**Message**:
```typescript
{
    versionRequired: false,
    validationSupported: true,
    previewSupported: true,
    templates: [{
		id: 'hello_world',
		title: 'Hello World Project',
		caption: 'A basic Rust project that outputs \'Hello World\' to the console',
		componentVersions: []
	},{
		id: 'crates',
		title: 'External Crate Example Project',
		caption: 'Cargo project that depends on the `rand` external crate',
		componentVersions: [{
			id: 'rand_version',
			title: 'rand Version',
			caption: 'The version of the `rand` crate that will be used',
			versions: [{
				id: '0.5.0',
				title: '0.5.0',
				caption: null
			}{
				id: '0.4.2',
				title: '0.4.2',
				caption: null
			}]
		}]
	}],
    componentVersions: [],
    defaultProvisioningParameters: {
		name: 'new_rust_project',
		location: '/new_rust_project',
		version: null,
		templateSelection: {
			id: 'hello_world',
			componentVersions: []
		},
		componentVersionSelections: []
	}
}
```
The returned information is then used by the client to build the wizard for the user to fill in. As the user fills in the various fields, the inputed parameters are validated.

**Method**: `projectProvisioning/validation`
**Direction**: `Client -> Server`
**Message**:
```typescript
{
	name: 'my_rust_project',
	location: 'invalid/path/my_rust_project
	version: null,
	templateSelection: {
		id: 'hello_world',
		componentVersions: []
	},
	componentVersionSelections: []
}
```

**Method**: `projectProvisioning/validation`
**Direction**: `Server -> Client`
**Message**:
```typescript
{
	errorMessage: 'Unable to create a new folder in the given location',
	erroneousParameters: [{
		parameterType: 'location'
		message: 'Unable to create a new folder in the given location',
		componentVersionId: null
	}]
}
```
After all the errors are addressed, the user wishes to review their inputs before creating the project, so they press "Next" in the wizard.

**Method**: `projectProvisioning/preview`
**Direction**: `Client -> Server`
**Message**:
```typescript
{
	name: 'my_rust_project',
	location: 'path/to/workspace/rust_projects/my_rust_project
	version: null,
	templateSelection: {
		id: 'hello_world',
		componentVersions: []
	},
	componentVersionSelections: []
}
```

**Method**: `projectProvisioning/preview`
**Direction**: `Server -> Client`
**Message**:
```typescript
{
	errorMessage: null,
	erroneousParameters: [],
	message: '# Steps that will be take to create the project
	
	 - A new directory named `my_rust_project` will be created in `path/to/workspace/rust_projects/`
		- `mkdir path/to/workspace/rust_projects/my_rust_project`
	 - This directory will contain a new directory called `src` which will contain `main.rs`
	 	- `touch path/to/workspace/rust_projects/my_rust_project/main.rs`
	 - `main.rs` will contain the required Rust code to output "Hello World"
		- `echo "fn main() { println!(\"Hello, world!\"); }" > path/to/workspace/rust_projects/my_rust_project/main.rs`'
}
```

The user is now confident in what will happen when they confirm the provisioning and press "Finish".

**Method**: `projectProvisioning/provision`
**Direction**: `Client -> Server`
**Message**:
```typescript
{
	name: 'my_rust_project',
	location: 'path/to/workspace/rust_projects/my_rust_project
	version: null,
	templateSelection: {
		id: 'hello_world',
		componentVersions: []
	},
	componentVersionSelections: []
}
```

**Method**: `projectProvisioning/provision`
**Direction**: `Server -> Client`
**Message**:
```typescript
{
	errorMessage: null,
	erroneousParameters: [],
	newFiles: ['path/to/workspace/rust_projects/my_rust_project/main.rs'],
	openFiles: ['path/to/workspace/rust_projects/my_rust_project/main.rs']
}
```

The user is now presented with the `main.rs` file and can begin working.


