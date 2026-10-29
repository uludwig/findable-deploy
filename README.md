# Deploy Findable to Azure

Findable is an enterprise AI platform that runs inside your own Azure subscription
and Microsoft Entra tenant: your people sign in with their work accounts, and your
documents stay in your tenant. This repository holds the deployment template and
the released application packages.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fuludwig%2Ffindable-deploy%2Fmain%2FmainTemplate.json/createUIDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Fuludwig%2Ffindable-deploy%2Fmain%2FcreateUiDefinition.json)

## What the template creates

One resource group with everything Findable needs, created new, with managed
identities only (no account keys, no connection strings):

- App Service (Linux, Node 24) running the Findable package
- Storage account, Cosmos DB (serverless), Azure AI Search
- Azure AI Foundry (one account, which also serves document extraction) with a chat and an embedding model deployment
- Log Analytics with Findable's log table, data collection endpoint and rule, and a workbook

Two size presets: **Trial** (the smallest set that runs end to end) and
**Production**. Azure bills these resources on your subscription; the Findable
license is separate.

## Steps

1. Click **Deploy to Azure** and sign in to the Azure portal with an account that can create resources in the subscription. The person who runs the Connect tenant script afterwards must be able to create app registrations and groups in the Entra tenant (Application Administrator and Groups Administrator, or Global Administrator) and hold Contributor on the web app.
2. Pick the subscription, a resource group, a region where the Azure AI Foundry models you want are offered (an existing resource group pins the region to its own; create a new group to deploy elsewhere), and a deployment name. Basics shows an error when the subscription cannot create the Foundry account or a Cosmos DB account in that region.
3. Choose the size and the models, then create. The wizard checks the names and the region against Azure as you go: a taken name or a model the region does not offer is refused on the field, a missing service or App Service tier shows an error box, and a quota shortfall shows a warning. The deployment takes about ten minutes; role assignments can take up to ten more to apply.
4. Open the deployment's `webAppUrl` output (Outputs on the deployment page: `https://<deployment name>-<7 characters>.azurewebsites.net`; the suffix keeps the name unique across Azure). The site shows its Connect tenant page: copy the script, run it in Azure Cloud Shell, and wait. The script creates the two app registrations and the administrators group, writes them onto the web app, and the site opens signed in. Nothing is pasted back unless the script says it could not write the app settings.

## If the deployment fails

Delete the resource group of the failed attempt and start again from the
**Deploy to Azure** button in a new tab, into a new resource group. The
portal's **Redeploy** reruns the stored parameters into the same group and
fails the same way, and a Cosmos DB account whose creation failed stays
behind in a Failed state, so a retry into the same group is refused with
"DatabaseAccount ... is in a failed provisioning state ... Please delete the
previous instance". A new resource group is also how you change the region.

When the failure was Cosmos DB refusing the region (a capacity or "high
demand" message), pick another region, or ask Microsoft support for Cosmos DB
capacity in that one (Help + support, Service and subscription limits
(quotas), Azure Cosmos DB) and deploy once it is granted.

## Requirements

- An Azure subscription in the Azure public cloud (Azure Government and Azure China are not supported) with Azure AI Foundry model quota in the chosen region.
- A Microsoft Entra tenant; the Connect tenant script creates two app registrations and an administrators group in it.
- A Findable license key for use beyond the 30-day trial: see https://www.findable.net/pricing.

## Support

support@findable.net

## Releases

Every release carries `mainTemplate.json`, `createUiDefinition.json`, the
application package `findable-<version>.zip` and the Partner Center upload
`findable-marketplace-<version>.zip`. The `mainTemplate.json` and
`createUiDefinition.json` at the root of this repository are the latest release's
copies (the Azure portal reads the template from this repository, not from the
release assets), so the button above always deploys the latest release.
