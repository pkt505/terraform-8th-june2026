##Below detail should be in terraform.tf file for connectivity
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "4.76.0"
    }
  }
}

provider "azurerm" {
  subscription_id = "c5bf1a3e-246a-4960-a860-9d42699c8a9e"
    tenant_id       = "00494a3e-560c-4a22-a096-792081e260e8"
    client_id       = "417e929a-11d9-4ce6-9cee-6b786d417fa3"
    client_secret   = ""
    features {
      
    }
    }
    -------------------------------------------------------------------
    Where code will be writtern main.tf
    resource "azurerm_resource_group" "rg" {
name     = local.resource_group_name
  location = local.resource_location
} 

resource "azurerm_network_security_group" "nsg" {
  name                = "nsgterraform"
  location            = local.resource_location
  resource_group_name = local.resource_group_name
  depends_on = [ azurerm_resource_group.rg ]
}

resource "azurerm_network_security_group" "nsg1" {
  name                = "nsgterraform1"
  location            = local.resource_location
  resource_group_name = local.resource_group_name
  depends_on = [ azurerm_resource_group.rg ]
}

resource "azurerm_virtual_network" "vnet" {
  name                = "vnetterraform"
  location            = local.resource_location
  resource_group_name = local.resource_group_name
  address_space       = ["10.0.0.0/16"]
  dns_servers         = ["10.0.0.4", "10.0.0.5"]
  depends_on = [ azurerm_resource_group.rg ]
}

resource "azurerm_subnet" "subnet1" {
  name                 = "subnetterraform1"
  resource_group_name  = local.resource_group_name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = [local.subnet_address_prefixes[0]]
  depends_on = [ azurerm_virtual_network.vnet ]
}

resource "azurerm_subnet" "subnet2" {
  name                 = "subnetterraform2"
  resource_group_name  = local.resource_group_name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = [local.subnet_address_prefixes[1]]
  depends_on = [ azurerm_virtual_network.vnet ]
}

resource "azurerm_subnet_network_security_group_association" "subnet1_assoc" {
  subnet_id                 = azurerm_subnet.subnet1.id
  network_security_group_id = azurerm_network_security_group.nsg.id
}

resource "azurerm_subnet_network_security_group_association" "subnet2_assoc" {
  subnet_id                 = azurerm_subnet.subnet2.id
  network_security_group_id = azurerm_network_security_group.nsg1.id
}

resource "azurerm_network_interface" "nic" {
  name                = "nicterraform"
  resource_group_name = local.resource_group_name
  location            = local.resource_location

  ip_configuration {
    name                          = "ipconfig1"
    subnet_id                     = azurerm_subnet.subnet2.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_virtual_machine" "vm" {
  name                  = "vmterraform"
  location              = local.resource_location
  resource_group_name   = local.resource_group_name
  network_interface_ids = [azurerm_network_interface.nic.id]
  vm_size               = "Standard_D2s_v3"
  depends_on = [ azurerm_network_interface.nic ]

 
  
  storage_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2019-Datacenter"
    version   = "latest"
  }

  storage_os_disk {
    name              = "osdiskterraform"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }

  os_profile {
    computer_name  = "hostname"
    admin_username = "adminuser"
    admin_password = "P@ssw0rd1234!"
  }

  os_profile_windows_config {
    provision_vm_agent        = true
    enable_automatic_upgrades = true
  }
}
----------------------------------
##To import manual created azure resources. Write code manually

resource "azurerm_resource_group" "rgnew" {
  name     = "SALES"  
  location = "Central India"
  tags = {
    environment = "production"
  } 
}
#then run following command 
#terraform import azurerm_resource_group.rgnew /subscriptions/c5bf1a3e-246a-4960-a860-9d42699c8a9e/resourceGroups/SALES
#it will import code and we can check in state file or modify again make plan/apply command
---------------------------------------------
how to move terraform tfstate file from local machine to azure storage/container
terraform {
  backend "azurerm" {
    resource_group_name  = "it"
    storage_account_name = "storagetf0505"
    container_name       = "containertf0505"
    key                  = "terraform.tfstate"
  }
}
 # run terrafor init command
 Migration Steps (moving state from local to Azure Storage)
Create the storage account + container (you already have those).

Add the backend block above to your Terraform config.

Run:

powershell
terraform init
Terraform will detect the local state and ask if you want to migrate it to the new backend.

Confirm, and your state file will be uploaded to Azure Blob Storage.
