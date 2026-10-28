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
- Azure AI Foundry with a chat and an embedding model deployment, Azure AI Services, Document Intelligence
- Log Analytics with Findable's log table, data collection endpoint and rule, and a workbook

Two size presets: **Trial** (the smallest set that runs end to end) and
**Production**. Azure bills these resources on your subscription; the Findable
license is separate.

## Steps

1. Click **Deploy to Azure** and sign in to the Azure portal with an account that can create resources in the subscription. The person who finishes the setup afterwards must be able to create app registrations in the Entra tenant (Application Administrator or Global Administrator).
2. Pick the subscription, a resource group and a region where the Azure AI Foundry models you want are offered, and a deployment name.
3. Choose the size and the models, then create. The wizard checks the names and the region against Azure as you go: a taken name or a model the region does not offer is refused on the field, a missing service or App Service tier shows an error box, and a quota shortfall shows a warning. The deployment takes about ten minutes; role assignments can take up to ten more to apply.
4. Open the deployment's `webAppUrl` output followed by `/#/start` (`https://<deployment name>-<7 characters>.azurewebsites.net/#/start`; the suffix keeps the name unique across Azure) and follow the first-run page to connect your Entra tenant and finish the setup.

## Requirements

- An Azure subscription in the Azure public cloud (Azure Government and Azure China are not supported) with Azure AI Foundry model quota in the chosen region.
- A Microsoft Entra tenant; the first-run page creates two app registrations and an administrators group.
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
