# Project Provisioning Protocol
The Project Provisioning Protocol is used between a project creation tool (the server) and an interface (the client) where a developer wishes to generate a new project. Messages are sent between the client and the server to inform the project creation tool how the user wishes to generate the project, and inform the user what options are available to them. The following diagram illustrates the communications between a tool and an interface through the process of creating a new project.

//TODO create diagrams
## Reason For This Protocol
Coders are do-ers, so when they wish to learn a new language or technology jumping into a new project is one of the first things they do. This allows coders to be exposed to simple ideas, syntaxes, and confirms that all the required technologies they will need are properly configured.

However, without a common protocol for creating projects not all technologies offer such a feature to users, leaving them behind in the orientation process of that technology. This is caused by valid reasons. Generating the support for new project creation across multiple interfaces (such as IDEs, web sites, or CLIs) is costly to developers and if they do create project provisioners, it is not ensured that they will be made in such a way that helps new users, also updates to all the different interfaces would be costly, leading to outdated tools only confusing new users more.

With a common protocol for project creation (or provisioning), technology developers only create one provisioning tool and any interface would be able to implement it, and these provisioning tools will be held to a higher standard and as a community we can decided what are the features that will most help with onboarding developers to new technologies.

## Contributing
The Project Provisioning Protocol is in its beginning stages, if this project sparks ideas that you think would add value to the project, [create an issue](https://github.com/LucasBullen/Project-Provisioning-Protocol/issues) so we can discuss it.

## Message overview
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

## Base Protocol
The communication between the interface and the provisioner uses [JSON RCP v2.0](http://www.jsonrpc.org/specification)

// TODO: explain the messaging process, error messages, json structure

## Project Provisioning Protocol
### <a name="initialize"></a>Initialize
> (Client -> Server) A request to begin the project provisioner parameter generation process, this opens and keeps open a connection to a server for future messages
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
 - `componentVersions`: A list of [ComponentVersion](#component-versions) objects listing all the components required for this project creation and the versions available
 - `defaultProvisioningParameters`: Either a [ProvisioningParameters](#provisioning-parameters) object or null if there are no default values for the interface

### <a name="validation"></a>Validation
> (Client -> Server) A request to validate a set of parameters for project provisioning

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

The message contents is the [ProvisioningParameters](#provisioning-parameters) object.

#### <a name="provision-result"></a>Provision Result
> (Server -> Client) The resulting effects of provisioning the project or the errors stopping the provisioning
```typescript
ProvisionResult: {
	errorMessage: String | null,
	erroneousParameters: ErroneousParameter[],
	newFiles: String[],
	openFiles: String[]
}
```
 - `errorMessage`: If there is an error with the parameters, this is either an overarching error message or an error message that does not relate to any one specific parameter
 - `erroneousParameters`: If there is an error with the parameters, A list of [ErroneousParameter](#erroneous-parameter) objects listing parameter specific errors
 - `newFiles`: A list of absolute paths to files that have been created by the project provisioning
 - `openFiles`: A list of absolute paths to files that should be displayed to the user through the client, to show that the project has been provisioned

### <a name="provision-instructions"></a>Provision Instructions
> (Client -> Server)A request to generate a list of files and their contents to provision a project from a set of given parameters

The message contents is the [ProvisioningParameters](#provisioning-parameters) object.

#### <a name="provision-instructions-result"></a>Provision Instructions Result
> (Server -> Client) A list of files and their contents to provision a project from a set of given parameters or the errors stopping this list from being generated
```typescript
ProvisionResult: {
	errorMessage: String | null,
	erroneousParameters: ErroneousParameter[],
	newFiles: Instruction[],
	openFiles: String[]
}
```
 - `errorMessage`: If there is an error with the parameters, this is either an overarching error message or an error message that does not relate to any one specific parameter
 - `erroneousParameters`: If there is an error with the parameters, A list of [ErroneousParameter](#erroneous-parameter) objects listing parameter specific errors
 - `newFiles`: A list of [Instruction](#instruction) objects that instruct the client on the files that need to be created for project provisioning
 - `openFiles`: A list of relative paths to files that should be displayed to the user through the client, to show that the project has been provisioned

## Protocol Objects
//TODO: add descriptors for each parameter in the objects

### <a name="template"></a>Template
```typescript
Template {
	id: String,
	title: String,
	caption: String | null,
	componentVersions: ComponentVersion[]
}
```
Uses:
 - [ComponentVersion](#component-version)

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
Uses:
 - [Version](#version)

Used by:
 - [Template](#template)

### <a name="version"></a>Version
```typescript
Version {
	id: String,
	title: String,
	caption: String | null
}
```
Uses:

Used by:
 - [ComponentVersion](#component-version)

### <a name="provisioning-parameters"></a>Provisioning Parameters
```typescript
ProvisioningParameters {
	name: String,
	location: String | null, //Is always null if InitializeRequest->allowFileCreation == false
	version: String | null,
	TemplateSelection: TemplateSelection | null,
	componentVersionSelections: ComponentVersionSelection[ ],
}
```
Uses:
 - [TemplateSelection](#template-selection)
 - [ComponentVersionSelection](#component-version-selection)

Used by:
 - [InitializeResult](#initializeResult)
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
Uses:

Used by:
 - [ProvisioningParameters](#provisioning-parameters)

### <a name="template-selection"></a>Template Selection
```typescript
TemplateSelection {
	id: String,
	componentVersions: ComponentVersion[]
}
```
Uses:

Used by:
 - [ProvisioningParameters](#provisioning-parameters)

### <a name="erroneous-parameter"></a>Erroneous Parameter
```typescript
ErroneousParameter { 
	parameterType: String, // (name, location, version, template, templateComponentVersion, componentVersion)
	message: String,
	componentVersionId: String | null // Only necessary if it is a component version causing the error
}
```
Uses:

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
Uses:

Used by:
 - [ProvisionInstructionsResult](#provision-instructions-result)

## Example Interactions

### Rust Project in an IDE
// TODO
### .NET Template Project Provisioning Web Site
// TODO
### DSL Project Creation From a CLI
// TODO
## License
[Eclipse Public License 2.0](https://www.eclipse.org/org/documents/epl-2.0/EPL-2.0.html)