# **IN VERY EARLY STEPS OF CREATION, COME BACK LATER**

| User | <-> | Client | <-> | Server |
|---|---|---|---|---|
|<td colspan=3 align=center><b>Open a Client</b> ||
|Wants to start a new project|Requires to know which PPs it has|Gets all the servers|Asks what PPs each has|Generates list of PPs|
|*|Opens the list|Lists all the PPs|Returns list PPs|⏎|
|<td colspan=3 align=center><b>Open a Project Provisioner</b> ||
|Chooses one of the PPs to open|→|Requires to know how to generate the PP|Starts a connection|Generates the descriptors|
|*|Opens the interface|Converts the descriptors into an interface for the user, populated with default values and default errors|Returns list of descriptors|⏎|
|<td colspan=3 align=center><b>Editing the descriptor values</b> ||
|Edits one of the fields|Does this edit update the validation? |Collects the user inputs for each descriptor|Requests to validate the descriptors|Generates the validation|
|*|Displays the effects of the edit|Updates the view with the new errors|Returns errors|⏎|
|<td colspan=3 align=center><b>After enough edits are made to remove all errors</b>||

