# Blobfuse Installation and Configuration Script for Terraform

This ***example script*** automates the installation and configuration of Blobfuse2 (or Blobfuse) on various Linux distributions, including Ubuntu, RHEL, and SLES. It also handles the creation of necessary directories, configuration files, and fstab entries for persistent mounts. Using Terraform template file.

## Features

* **Automatic Distribution and Version Detection:** Identifies the Linux distribution and version to ensure compatibility.
* **Linux Software Repository for Microsoft Products Installation:** Adds the Microsoft repository for the detected distribution and version.
* **Blobfuse2/Blobfuse Installation:** Installs Blobfuse2 if available, or falls back to Blobfuse if Blobfuse2 is not found.
* **Mount Point and Temporary Directory Creation:** Creates the mount point and temporary directories. The allow_other is set to true.
* **Configuration File Generation:** Generates the Blobfuse configuration file using provided storage account details.
* **Fstab Entry Creation:** Creates an fstab entry for persistent Blobfuse mounts.
* **Input Variables:** Accepts storage account name, storage account key, container name, mount path, and temporary path as input variables.
* **Logging:** Provides detailed logging with UTC timestamps.
* **Error Handling:** Includes robust error handling and informative error messages.
* **Case-Insensitive Distribution Check:** Ensures compatibility with various Ubuntu-like distributions.

## Usage

1. **Move the deploy-blobfuse.sh.tftpl to your desired path**

    **Example:**

    ```text
    /scripts/deploy-blobfuse.sh.tftpl
    ```

2. **Define your variables to mount Azure blob**

    * `<storage_account_name>`: Azure storage account name.
    * `<storage_account_key>`: Azure storage account key. **(Secure this key!)**
    * `<container_name>`: Azure blob container name.
    * `<mount_path>`: Local directory to mount Blobfuse (e.g., `/mnt/blobfuse`).

    **Note:** the `<tmp_path>`: Local directory will be `<mount_pathtmp>` (e.g., `/mnt/blobfusetmp`) by default. Extra configuration not needed.

    **Example:**

    ```Terrafrom
    locals{
      storage_account_name = azurerm_storage_account.this.name
      storage_account_key  = azurerm_storage_account.this.primary_access_key
      container_name       = azurerm_storage_container.this.name
      mount_path           = "/mnt/blobfuse"
    }
    ```

    **Note:** check documentation regarding configiration details. [Links](#links)

3. **Create your Terraform azurerm_linux_virtual_machine resource:**

    Use the [custom_data](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/linux_virtual_machine#custom_data-1) argument and [templatefile function](https://developer.hashicorp.com/terraform/language/functions/templatefile).

    **Example:**

    ```Terrafrom
    resource "azurerm_linux_virtual_machine" "this" {
    # required arguments
      custom_data = base64encode(templatefile("${path.module}/scripts/deploy-blobfuse.sh.tftpl", {
        storage_account_name = local.storage_account_name
        storage_account_key  = local.storage_account_key
        container_name       = local.container_name
        mount_path           = local.mount_path
      }))
    }
    ```

4. **Deploy your infrastructue with Terraform**

   **Note:** Blobfuse deployment could tike some time.

5. **Review logs and check your mount point:**
    * Check `/var/log/blobfuse-init.log` log file for installation and configuration details.

## Links

[What is BlobFuse?](https://learn.microsoft.com/en-us/azure/storage/blobs/blobfuse2-what-is)

[Azure-storage-fuse wiki](https://github.com/Azure/azure-storage-fuse/wiki)

[Linux Software Repository for Microsoft Products](https://learn.microsoft.com/en-us/linux/packages)

## Important Security Notes

* **Storage Account Key:** The storage account key is sensitive information. Avoid printing it to the console or storing it in plain text. Consider using environment variables, secrets management tools, or other secure methods to handle the key.

* **Sudo Privileges:** The script requires sudo privileges to install packages and create directories.

## Supported Distributions

* Ubuntu (20.04, 22.04, 24.04 and other debian based distros)
* RHEL (7, 8, 9)
* SLES (12, 15)

## Dependencies

* wget
* dpkg (for Ubuntu)
* apt-get (for Ubuntu)
* rpm (for RHEL and SLES)
* yum (for RHEL and SLES)
* lsb_release (if available)
* /etc/os-release
* /etc/redhat-release (for RHEL)
* /etc/SuSE-release (for SLES)

## Logging

All output is logged to `/var/log/blobfuse-init.log` with UTC timestamps.
