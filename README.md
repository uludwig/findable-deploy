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
- Storage account, Cosmos DB (serverless, with the database and the containers the app starts on), Azure AI Search
- Key Vault with the key that seals stored sign-in tokens
- Azure AI Foundry (one account, which also serves document extraction) with a chat and an embedding model deployment
- Log Analytics with Findable's log table, data collection endpoint and rule, and a workbook

Two size presets: **Trial** (the smallest set that runs end to end) and
**Production**. Azure bills these resources on your subscription; the Findable
license is separate.

## Steps

1. Click **Deploy to Azure** and sign in to the Azure portal with an account that can create resources in the subscription. The person who runs the Connect tenant script afterwards must be able to create app registrations and groups in the Entra tenant (Application Administrator and Groups Administrator, or Global Administrator) and hold Contributor on the web app.
2. Pick the subscription, a resource group, a region and a deployment name (an existing resource group pins the region to its own; create a new group to deploy elsewhere). No single region offers everything to every subscription, so Basics checks the region as you go and shows an error when the subscription cannot create the Azure AI Foundry account or the Cosmos DB account there, or the region does not offer the models (Findable deploys `text-embedding-3-small` and the newest chat model your subscription can deploy, `gpt-5.6-terra` down to `gpt-4.1`; there is nothing to choose, and the Size tab names the model when it is not `gpt-5.6-terra`). Under each error is a checkbox that creates that one account in another region of your choice while everything else stays in the region you picked: tick it and choose a region the check accepts. Basics also shows an error when a Key Vault from an earlier install into a resource group of the same name is still soft-deleted (see below). Do not change the region once the Summary page has validated: the portal created the resource group then, the deployment goes into the group's region, and the Summary page keeps what it validated; use the checkboxes instead, or start again in a new tab with a new resource group name (Basics shows an error when the group you named already exists in another region).
3. Choose the size, then create. The Size tab can put the search service alone in another region, for a region where Azure AI Search has no capacity left (see below), shows an error when the subscription has no App Service VM quota in the region or no Azure OpenAI quota for any chat model Findable can deploy (see below), names the chat model the install deploys when it is not `gpt-5.6-terra`, and warns when the embedding model's quota is short. The wizard checks the names and the regions against Azure as you go: a missing service, model or App Service tier shows an error box, and a quota shortfall shows a warning. The deployment takes about ten minutes; role assignments can take up to ten more to apply.
4. Open the deployment's `webAppUrl` output (Outputs on the deployment page: `https://<deployment name>-<7 characters>.azurewebsites.net`; the suffix keeps the name unique across Azure). The site opens on its Connect tenant page (every other prerequisite was met by the template): copy the script, run it in Azure Cloud Shell, and wait. The script creates the two app registrations and the administrators group, writes them onto the web app, and the site opens signed in. Nothing is pasted back unless the script says it could not write the app settings.

## If the deployment fails

Delete the resource group of the failed attempt and start again from the
**Deploy to Azure** button in a new tab, into a new resource group. The
portal's **Redeploy** reruns the stored parameters into the same group and
fails the same way, and a Cosmos DB account whose creation failed stays
behind in a Failed state, so a retry into the same group is refused with
"DatabaseAccount ... is in a failed provisioning state ... Please delete the
previous instance". A resource group with a new name is also how you change
the region: the portal creates a new group the first time the Summary page
validates, even when that validation fails, and the deployment goes into the
group's region, so a region changed afterwards needs a new group name too
(Basics shows an error when the group you named exists in another region).

When the failure was Cosmos DB refusing the region (a capacity or "high
demand" message), deploy again into a new resource group and tick "Create the
Cosmos DB account in another region" on Basics, picking a region its check
accepts: only the Cosmos DB account moves. Or ask Microsoft support for Cosmos
DB capacity in that one (Help + support, Service and subscription limits
(quotas), Azure Cosmos DB) and deploy once it is granted.

When the failure was the Foundry account or a model deployment refusing the
region (the Basics check names the model the region lacks; an install from
before 2026-11-03 could fail on the Summary page with "The specified SKU
'Standard' for model 'text-embedding-3-small 1' is not supported in this
region", which the embedding deployment's `GlobalStandard` SKU has since
removed), deploy again into a new resource group and tick "Create the Azure AI
Foundry account in another region" on Basics, picking a region the checks
accept: only the Foundry account and its two model deployments move.

When the Summary page failed with `SpecialFeatureOrQuotaIdRequired` ("The
current subscription does not have access to this model"), the subscription
has no Azure OpenAI quota for that model in the Foundry account's region:
GPT-5.4 and later are limited-access models Azure opens per subscription on
application, and a brand-new subscription can hold zero quota for every model.
The wizard deploys the newest model your subscription has quota for and names
it on the Size tab; when it has quota for none, the Size tab shows an error.
Request quota for a model in Azure AI Foundry (Quotas, Azure OpenAI) or apply
for GPT-5.4 and later access, then deploy again into a new resource group. A
newer model can be deployed on the Foundry account later and chosen on
Findable's AI model endpoints page.

When the Summary page failed with `SubscriptionIsOverQuotaForSku` ("Current
Limit (Total VMs): 0"), the subscription has no App Service VM quota in the
region: a new or a trial subscription starts at zero. The Size tab shows an
error when that is the case. Request the quota (Quotas, App Service, the
region, the tier's family and the regional total; a trial subscription may
need an upgrade to pay-as-you-go first) and deploy again, or deploy into
another region with a new resource group.

When the failure was the search service refusing the region with
`InsufficientResourcesAvailable` ("The region '...' is currently out of the
resources required to provision new services"), Azure AI Search has no
capacity left there for new services, and nothing can read that before the
deployment creates the service. Deploy again into a new resource group and
tick "Create the search service in another region" on the Size tab: only the
search service moves, and everything else stays in the region you chose.

Deleting a resource group leaves its Key Vault soft-deleted under its name for
seven days, and the vault's name derives from the resource group's, so a
reinstall under the same resource group name within that week is refused with
a conflict on the vault. Basics shows an error when that is the case: give the
group a new name, or purge the vault first (Key vaults, Manage deleted vaults).

## Requirements

- An Azure subscription in the Azure public cloud (Azure Government and Azure China are not supported) with App Service VM quota and Azure OpenAI quota for at least one GPT model in the chosen region. A brand-new or trial subscription may hold neither: request both under Quotas before installing, and apply for GPT-5.4 and later access to get the default model.
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
