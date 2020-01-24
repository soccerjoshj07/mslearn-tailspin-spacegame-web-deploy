
## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Environment Setup

```bash
# Create a random number to append to url
webappsuffix=$RANDOM

# Create rg
az group create --name tailspin-space-game-rg

# Create the asp's
az appservice plan create \
  --name tailspin-space-game-test-asp \
  --resource-group tailspin-space-game-rg \
  --sku B1

az appservice plan create \
  --name tailspin-space-game-prod-asp \
  --resource-group tailspin-space-game-rg \
  --sku P1V2

# Create the webapps
az webapp create \
  --name tailspin-space-game-web-dev-$webappsuffix \
  --resource-group tailspin-space-game-rg \
  --plan tailspin-space-game-test-asp

az webapp create \
  --name tailspin-space-game-web-test-$webappsuffix \
  --resource-group tailspin-space-game-rg \
  --plan tailspin-space-game-test-asp

az webapp create \
  --name tailspin-space-game-web-staging-$webappsuffix \
  --resource-group tailspin-space-game-rg \
  --plan tailspin-space-game-prod-asp

# List the webapps
az webapp list \
  --resource-group tailspin-space-game-rg \
  --query "[].{hostName: defaultHostName, state: state}" \
  --output table

#Get the staging name to use for deployment slots
staging=$(az webapp list \
  --resource-group tailspin-space-game-rg \
  --query "[?contains(@.name, 'tailspin-space-game-web-staging')].{name: name}" \
  --output tsv)

# Create deployment slot
az webapp deployment slot create \
  --name $staging \
  --resource-group tailspin-space-game-rg \
  --slot swap

# List deployment slot's hostname (just adds -swap)
az webapp deployment slot list \
    --name $staging \
    --resource-group tailspin-space-game-rg \
    --query [].{hostNames} \
    --output tsv

# Cleanup
az group delete --name tailspin-space-game-rg
```
