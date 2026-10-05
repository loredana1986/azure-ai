# Remove tier upgrade policy for MS AI services

## 1. Received MS email about automatic tier upgrade  

Important update: Eligibility for Quota Tier upgrade  
We are pleased to inform you that, based on your recent usage and account standing, your subscription 3a9c6d7e-2025-4d61-8b58-44fde013e0be is now eligible for an upgrade from your current Tier (Free Tier) to the next Tier (Tier 1) within our AI Services platform.  
What does this mean for you?  
You will gain access to the enhanced benefits and features associated with the next Tier.  
The upgrade will be processed automatically unless you choose to remain at your current Tier.  
What do you need to do?
If you wish to remain at your current Tier (Free Tier), please follow the [link](https://learn.microsoft.com/en-us/azure/foundry/openai/quotas-limits?tabs=bash%2Ctier1) within the next 3 days.    
If no action is taken, your subscription 3a9... will be automatically upgraded to Tier 1.  
If you have any questions or would like more information about the Tier upgrade process, please contact our support team. Thank you for being a valued customer.  

## 2. Opt out of auto upgrades  

Open Bash CLI from Azure Portal.  

```bash
SUBSCRIPTION_ID="3a9..."

az rest \
  --method PATCH \
  --url "https://management.azure.com/subscriptions/$SUBSCRIPTION_ID/providers/Microsoft.CognitiveServices/quotaTiers/default?api-version=2025-10-01-preview" \
  --body '{
    "properties": {
      "tierUpgradePolicy": "NoAutoUpgrade"
    }
  }'

# result
{
  "id": "/subscriptions/3a9.../providers/Microsoft.CognitiveServices/quotaTiers/default",
  "name": "default",
  "properties": {
    "assignmentDate": "2026-10-04T06:33:47.6456342Z",
    "currentTierName": "Free Tier",
    "tierUpgradePolicy": "NoAutoUpgrade"
  },
  "type": "Microsoft.CognitiveServices/quotaTiers"
}
```

Wait 5 minutes and check whether the change took place. It's important to wait since changes will not be visible within the first few minutes.  

```bash
az rest \
  --method GET \
  --url "https://management.azure.com/subscriptions/3a9c6d7e-2025-4d61-8b58-44fde013e0be/providers/Microsoft.CognitiveServices/quotaTiers/default?api-version=2025-10-01-preview"

# result
{
  "id": "/subscriptions/3a9.../providers/Microsoft.CognitiveServices/quotaTiers/default",
  "name": "default",
  "properties": {
    "assignmentDate": "2026-10-04T06:33:47.6456342Z",
    "currentTierName": "Free Tier",
    "tierUpgradePolicy": "NoAutoUpgrade"
  },
  "type": "Microsoft.CognitiveServices/quotaTiers"
}
```
