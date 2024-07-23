# Lab 4 - Create a  managed HSM (25 mins)

## Task 1: Set the CLI context

In this task we will setup a new Azure Keyvault Managed HSM.

1. Navigate to the Azure Portal https://portal.azure.com
1. Open the cloud shell by clicking on the icon in to toolbar
1. Set the correct subscription by using the following command

``` CLI
     az account set -s "Visual Studio Enterprise Subscription"
```

## Task 2: Create the Managed HSM in Cloud Shell

1. Run the following command to assign your user object id to a variable

``` CLI
    $oid=$(az ad signed-in-user show --query id -o tsv)
```
1. Now you can create the Managed HSM with the CLI. Replace `<alias>` with your alias.

``` CLI
    az keyvault create --hsm-name "<alias>-MHSM" --resource-group "<alias>-rg" --location "eastus" --administrators $oid --retention-days 7
```

1. Wait for the keyvault to be provisioned

## Task 2: Activate the Managed HSM

In this task you will setup the keys to be used by the Managed HSM to setup a Security Domain.

1. Run the following commands to generate 3 certificates and store them in the cloud shell storage

``` CLI
    openssl req -newkey rsa:2048 -nodes -subj '/C=IL' -keyout cert_0.key -x509 -days 365 -out cert_0.cer
    openssl req -newkey rsa:2048 -nodes -subj '/C=IL' -keyout cert_1.key -x509 -days 365 -out cert_1.cer
    openssl req -newkey rsa:2048 -nodes -subj '/C=IL' -keyout cert_2.key -x509 -days 365 -out cert_2.cer
```

__Note__ In real life you would do this on a trusted workstation and not in the cloud.

1. Ceate the security domain. Replace `<alias>` with your alias.

```CLI
    az keyvault security-domain download --hsm-name <alias>-MHSM --sd-wrapping-keys ./cert_0.cer ./cert_1.cer ./cert_2.cer --sd-quorum 2 --security-domain-file <alias>-MHSM-SD.json
```

## Task 3: Add yourself a crpto role

1. Using the portal, navigate to the newly created MHSM
1. Click on the **Local RBAC** under the **Settings** section. 
1. Click **Add**
1. **Role:** **Managed HSM Crypto User**
1. **Scope:** **All Keys ("/ or /keys")"
1. **Security Principal:** Select your user
1. Click **Create**

![Virtual machine basic details](img/vm-basics.png)

## Task 4: Explore creating new keys

** Navigate to the **Keys** blade under the **Settings** sectionin the portal and explore key creation options.

## Task 5: Delete the keyvault