
# Delinea Secret Server

External Secrets Operator integration with [Delinea Secret Server](https://docs.delinea.com/online-help/secret-server/start.htm).

### Basic Requirements

- You are required to supply a username, password and a fully qualified
   Secret Server tenant URL to authenticate i.e.
   `https://yourTenantName.secretservercloud.com`.

 - You must properly Configure Secret Server for communication with ESO.

 - You must create and properly configure a secret that is accessable by
   the account provided above.

### Configuring Secret Server
The Secret Server ESO provider module implements the Secret Server [Golang SDK](https://github.com/DelineaXPM/tss-sdk-go).<br />
`Note: You will need administrator privileges to configure Secret Server.`

Please refer to the Secret Server [SDK setup guide](https://docs.delinea.com/online-help/secret-server/api-scripting/sdk-devops/using-sdk/index.htm#SetupProcedure) documentation
for details on how to successfully configure Secret Server to work with ESO.

 - To learn more about Secret Server, refer to the  Secret Server [end user guide](https://docs.delinea.com/online-help/secret-server/guides-tutorials/end-user-guide/index.htm) documentation.

### Creating your secret
Note: ESO will be looking for a specific field in your secret named "Data".

 ***Create a secret template***

 - Navigate to Settings->Secrets->"Secret Templates"

 - Click "Create / Import Template"

 - Enter a name and click save

 - In the following template information screen click "Fields" then click "Add Field"

 - Enter "Data" in the name field, and select "Text" as the "Data Type"

 - Save the template

 ***Create a secret***

 - Create a new secret using the template you created above<br />*(the secret name field should not contain spaces or control characters)*

 - Add the JSON data you wish to work with in the "Data" field

 - Save the secret

 - Note the secret ID in the URL

### Configuring the ESO Secret Store

Both username and password can be specified directly in your `SecretStore` yaml config, or by referencing a kubernetes secret.<br />

 - Both `username` and `password` can either be specified directly via the `value` field (example below)
```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: secret-server-store
spec:
  provider:
    secretserver:
      serverURL: "https://yourtenantname.secretservercloud.com"
      username:
        value: "yourusername"
      password:
        value: "yourpassword"
```
 - Or you can reference a kubernetes secret (password example below).

```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: secret-server-store
spec:
  provider:
    secretserver:
      serverURL: "https://yourtenantname.secretservercloud.com"
      username:
        value: "yourusername"
      password:
        secretRef:
          name: <NAME_OF_K8S_SECRET>
          key: <KEY_IN_K8S_SECRET>
```

### Referencing Secrets

Secrets may be referenced by secret ID or secret name.
>If using the secret name as your remoteRef.key
the name field must not contain spaces or control characters.<br />
If multiple secrets are found, *`only the first found secret will be returned`*.

Please note: `Retrieving a specific version of a secret is not yet supported.`

All Secret Server secrets are JSON objects, therefore you must specify the `remoteRef.property`
in your ExternalSecret configuration.<br />
You can access nested values or arrays using [gjson syntax](https://github.com/tidwall/gjson/blob/master/SYNTAX.md).

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
    name: secret-server-external-secret
spec:
    refreshInterval: 15s
    secretStoreRef:
        kind: SecretStore
        name: secret-server-store
    data:
      - secretKey: SecretServerValue #<SECRET_VALUE_RETURNED_HERE>
        remoteRef:
          key: "52622" #<SECRET_ID>
          property: "array.0.value" #<GJSON_PROPERTY> * an empty property will return the entire secret
```

### Accessing your secret
You can either retrieve your entire secret or you can use a JSON formatted string
stored in your secret "Data" field located at Items[0].ItemValue to retrieve a specific value.<br />
See example JSON secret below.

### Examples
Using the json formatted secret below:

- Lookup a single top level property using secret ID.

>spec.data.remoteRef.key = 52622 (id of the secret)<br />
spec.data.remoteRef.property = "user" (Items.0.ItemValue user attribute)<br />
returns: marktwain@hannibal.com

- Lookup a nested property using secret name.

>spec.data.remoteRef.key = "external-secret-testing" (name of the secret)<br />
spec.data.remoteRef.property = "books.1" (Items.0.ItemValue books.1 attribute)<br />
returns: huckleberryFinn

- Lookup by secret ID (*secret name will work as well*) and return the entire secret.

>spec.data.remoteRef.key = "52622" (id of the secret)<br />
spec.data.remoteRef.property = "" <br />
returns: The entire secret in JSON format as displayed below


```JSON
{
  "Name": "external-secret-testing",
  "FolderID": 73,
  "ID": 52622,
  "SiteID": 1,
  "SecretTemplateID": 6098,
  "SecretPolicyID": -1,
  "PasswordTypeWebScriptID": -1,
  "LauncherConnectAsSecretID": -1,
  "CheckOutIntervalMinutes": -1,
  "Active": true,
  "CheckedOut": false,
  "CheckOutEnabled": false,
  "AutoChangeEnabled": false,
  "CheckOutChangePasswordEnabled": false,
  "DelayIndexing": false,
  "EnableInheritPermissions": true,
  "EnableInheritSecretPolicy": true,
  "ProxyEnabled": false,
  "RequiresComment": false,
  "SessionRecordingEnabled": false,
  "WebLauncherRequiresIncognitoMode": false,
  "Items": [
    {
      "ItemID": 280265,
      "FieldID": 439,
      "FileAttachmentID": 0,
      "FieldName": "Data",
      "Slug": "data",
      "FieldDescription": "json text field",
      "Filename": "",
      "ItemValue": "{ \"user\": \"marktwain@hannibal.com\", \"occupation\": \"author\",\"books\":[ \"tomSawyer\",\"huckleberryFinn\",\"Pudd'nhead Wilson\"] }",
      "IsFile": false,
      "IsNotes": false,
      "IsPassword": false
    }
  ]
}
```
