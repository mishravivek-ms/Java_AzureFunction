
Terraform script for Azure Function
============================================


Steps
---------------

*   Create a Resource group name 'terraformRG'
*   Store the file into folder 


        
        provider "azurerm" {
        features {}
        }
        
        data "azurerm_resource_group" "existing" {
        name = "terraformRG"
        }
        
        resource "azurerm_storage_account" "example" {
        name                     = "tfazurefunctionstorage"
        resource_group_name      = data.azurerm_resource_group.existing.name
        location                 = data.azurerm_resource_group.existing.location
        account_tier             = "Standard"
        account_replication_type = "LRS"
        }
        
        resource "azurerm_app_service_plan" "example" {
        name                = "tfazurefunction-plan"
        location            = data.azurerm_resource_group.existing.location
        resource_group_name = data.azurerm_resource_group.existing.name
        kind                = "FunctionApp"
        
            sku {
                tier = "Dynamic"
                size = "Y1"
            }
        }
        
        resource "azurerm_function_app" "example" {
        name                      = "tfazurefunction"
        location                  = data.azurerm_resource_group.existing.location
        resource_group_name       = data.azurerm_resource_group.existing.name
        app_service_plan_id       = azurerm_app_service_plan.example.id
        storage_account_name      = azurerm_storage_account.example.name
        storage_account_access_key = azurerm_storage_account.example.primary_access_key
        version = "~4"
        https_only = true
        
            site_config {
            java_version = "11"
        }
        }


*   Execute command to create a Azure function_app

        terraform init
        terraform validate
        terraform plan
        terraform apply
