# Azure VM and Networking Training Documentation

## 1. Overview
This document summarizes the hands-on training performed on Azure, including creating a virtual machine (VM) and networking setup using Azure CLI. All steps are designed for **DevOps practice** and reference for future projects.

---

## 2. Azure Environment Setup

### 2.1 Create a Resource Group
Resource Group acts as a container for all resources (VMs, VNet, Subnets, Public IPs, etc.)

az group create --name mycli --location westeurope


* **Check the Resource Group**:

az group show --name mycli

---

### 2.2 Create Virtual Network (VNet) and Subnet

az network vnet create \
  --resource-group mycli \
  --name vnetName \
  --address-prefix 10.0.0.0/16 \
  --subnet-name subnetName \
  --subnet-prefix 10.0.1.0/24

* **Verify VNet and Subnet**:

az network vnet show --resource-group mycli --name vnetName
az network vnet subnet show --resource-group mycli --vnet-name vnetName --name subnetName

---

## 3. Create a VM (Free Tier, Zone 2)

az vm create \
  --resource-group mycli \
  --name vmName \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --zone 2 \
  --vnet-name vnetName \
  --subnet subnetName \
  --generate-ssh-keys \
  --output json

* Notes:

  * **Size B1s** → Free Tier
  * **Zone 2** → Availability Zone
  * SSH key generated for secure access

* **Check VM Status and IPs**:

az vm list -d -o table

* **Check VM size and zone**:

az vm show --name vmName --resource-group mycli --query "{name:name,size:hardwareProfile.vmSize,zone:zones}" -o table

* **Connect to VM via SSH**:

ssh azureuser@<Public-IP>

> For production-like setups, connect via **Bastion host** to avoid public IP exposure.

---

## 4. Firewall Rule Example (Nginx Access)

* Create a Network Security Group (NSG) rule to allow specific IP to access Nginx:

az network nsg rule create \
  --resource-group mycli \
  --nsg-name myNsg \
  --name nginxRule \
  --protocol tcp \
  --priority 100 \
  --destination-port-ranges 80 \
  --source-address-prefixes <Your-Public-IP>

---

## 5. Verification Commands

* **List all resources in the Resource Group**:

az resource list --resource-group mycli -o table

* **Check VM status**:

az vm get-instance-view --name vmName --resource-group mycli -o table

---

## 6. Cleanup / Delete Resources

* **Delete a VM**:

az vm delete --resource-group mycli --name vmName --yes

* **Delete associated NIC**:

az network nic delete --resource-group mycli --name vmNameNic

* **Delete Public IP**:

az network public-ip delete --resource-group mycli --name vmNamePublicIP

* **Delete OS Disk**:

az disk delete --resource-group mycli --name vmName_OsDisk --yes

* **Delete entire Resource Group (all resources)**:

az group delete --name mycli --yes --no-wait

* **Verify deletion**:

az group show --name mycli

> Should return "Resource group not found" if deletion succeeded.

---

## 7. Notes & Best Practices

1. **Always specify VM size explicitly** to avoid future changes in CLI defaults.
2. **Use Free Tier VM sizes** (B1s) during training to avoid charges.
3. **Delete resources immediately after testing** to save Azure credits.
4. **Organize resources in Resource Groups** for easier management.
5. **Use SSH keys** for secure VM access rather than passwords.
6. **For production environments**, use Bastion host instead of public IP for security.

---

## 8. Optional: Additional Commands

* **List available VM sizes in a region**:

az vm list-sizes --location westeurope -o table

* **Check network interfaces**:

az network nic list --resource-group mycli -o table

* **List disks**:

az disk list --resource-group mycli -o table

* **List NSG rules**:

az network nsg rule list --resource-group mycli --nsg-name myNsg -o table

---

**End of Documentation**
