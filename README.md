# Deploy Findable to Azure

Findable is an enterprise AI platform that runs inside your own Azure subscription
and Microsoft Entra tenant: your people sign in with their work accounts, and your
documents stay in your tenant. This repository holds the deployment template and
the released application packages.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fgithub.com%2Fuludwig%2Ffindable-deploy%2Freleases%2Flatest%2Fdownload%2FmainTemplate.json/createUIDefinitionUri/https%3A%2F%2Fgithub.com%2Fuludwig%2Ffindable-deploy%2Freleases%2Flatest%2Fdownload%2FcreateUiDefinition.json)

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
2. Pick the subscription, a resource group and a region where Azure OpenAI models are offered, and a deployment name.
3. Choose the size and the models, then create. The deployment takes about ten minutes; role assignments can take up to ten more to apply.
4. Open `https://<deployment name>.azurewebsites.net/#/start` and follow the first-run page to connect your Entra tenant and finish the setup.

## Requirements

- An Azure subscription with quota for Azure OpenAI in the chosen region.
- A Microsoft Entra tenant; the first-run page creates two app registrations and an administrators group.
- A Findable license key for use beyond the 30-day trial: see https://www.findable.net/pricing.

## Support

support@findable.net

## Releases

Every release carries `mainTemplate.json`, `createUiDefinition.json`, the
application package `findable-<version>.zip` and the Partner Center upload
`findable-marketplace-<version>.zip`. The button above always deploys the latest.
