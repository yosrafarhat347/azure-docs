---
title: Tutorial - Create and use an Azure file shares on Windows VMs
description: This tutorial covers how to create and use an Azure files shares in the Azure portal. Connect it to a Windows VM, connect to the file share, and upload a file to the file share.
author: roygara
ms.service: storage
ms.topic: tutorial
ms.date: 02/14/2022
ms.author: rogarana
ms.subservice: files
ms.custom: mode-ui
#Customer intent: As an IT admin new to Azure Files, I want to try out Azure file share so I can determine whether I want to subscribe to the service.
---

# Tutorial: Create and manage Azure file shares with Windows virtual machines via the Azure portal

Azure Files offers fully managed file shares in the cloud that are accessible via the industry standard [Server Message Block (SMB) protocol](/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) or [Network File System (NFS) protocol](https://en.wikipedia.org/wiki/Network_File_System). In this tutorial, you will learn a few ways you can use an Azure file share in a Windows virtual machine (VM).

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

> [!div class="checklist"]
> * Create a storage account
> * Create a file share
> * Deploy a VM
> * Connect to a VM
> * Mount an Azure file share to your VM
> * Create and delete a share snapshot

## Applies to
| File share type | SMB | NFS |
|-|:-:|:-:|
| Standard file shares (GPv2), LRS/ZRS | ![Yes](../media/icons/yes-icon.png) | ![No](../media/icons/no-icon.png) |
| Standard file shares (GPv2), GRS/GZRS | ![Yes](../media/icons/yes-icon.png) | ![No](../media/icons/no-icon.png) |
| Premium file shares (FileStorage), LRS/ZRS | ![Yes](../media/icons/yes-icon.png) | ![No](../media/icons/no-icon.png) |

## Getting started

### Create a storage account

Before you can work with an Azure file share, you have to create an Azure storage account.

1. Sign in to the [Azure portal](https://portal.azure.com).
1. On the Azure portal menu, select **All services**. In the list of resources, type **Storage Accounts**. As you begin typing, the list filters based on your input. Select **Storage Accounts**.
1. On the **Storage Accounts** window that appears, choose **+ New**.
1. On the **Basics** tab, select the subscription in which to create the storage account.
1. Under the **Resource group** field, select your desired resource group, or create a new resource group.
1. Next, enter a name for your storage account. The name you choose must be unique across Azure. The name also must be between 3 and 24 characters in length, and may include only numbers and lowercase letters.
1. Select a region for your storage account, or use the default region.
1. Select a performance tier. The default tier is *Standard*.
1. Specify how the storage account will be replicated. The default redundancy option is *Geo-redundant storage (GRS)*.
1. Select **Review + Create** to review your storage account settings and create the account.
1. Select **Create**.

The following image shows the settings on the **Basics** tab for a new storage account:

:::image type="content" source="media/storage-files-quick-create-use-windows/account-create-portal.png" alt-text="Screenshot showing how to create a storage account in the Azure portal." lightbox="media/storage-files-quick-create-use-windows/account-create-portal.png":::

### Create an Azure file share

Next, create a file share.

1. When the Azure storage account deployment is complete, select **Go to resource**.
1. Select **File shares** from the storage account pane.

    ![Screenshot, File shares selected.](./media/storage-files-quick-create-use-windows/click-files.png)

1. Select **+ File Share**.

    ![Screenshot, + file share selected to create a new file share.](./media/storage-files-quick-create-use-windows/create-file-share.png)

1. Name the new file share *qsfileshare*, enter "1" for the **Quota**, leave **Transaction optimized** selected, and select **Create**. The quota can be a maximum of 5 TiB (100 TiB, with large file shares enabled), but you only need 1 GiB for this.
1. Create a new txt file called *qsTestFile* on your local machine.
1. Select the new file share, then on the file share location, select **Upload**.

    ![Screenshot of file upload.](./media/storage-files-quick-create-use-windows/create-file-share-portal5.png)

1. Browse to the location where you created your .txt file > select *qsTestFile.txt* > select **Upload**.

### Deploy a VM

So far, you've created an Azure storage account and a file share with one file in it. Next, create an Azure VM with Windows Server 2016 Datacenter to represent the on-premises server.

1. Expand the menu on the left side of the portal and select **Create a resource** in the upper left-hand corner of the Azure portal.
1. Under **Popular services** select **Virtual machine**.
1. In the **Basics** tab, under **Project details**, select the resource group you created earlier.

   ![Screenshot of Basic tab, basic VM information filled out.](./media/storage-files-quick-create-use-windows/vm-resource-group-and-subscription.png)

1. Under **Instance details**, name the VM *qsVM*.
1. For **Image** select **Windows Server 2016 Datacenter - Gen2**.
1. Leave the default settings for **Region**, **Availability options**, and **Size**.
1. Under **Administrator account**, add a **Username** and enter a **Password** for the VM.
1. Under **Inbound port rules**, choose **Allow selected ports** and then select **RDP (3389)** and **HTTP** from the drop-down.
1. Select **Review + create**.
1. Select **Create**. Creating a new VM will take a few minutes to complete.

1. Once your VM deployment is complete, select **Go to resource**.

### Connect to your VM

Now that you've created the VM, connect to it so you can mount your file share.

1. Select **Connect** on the virtual machine properties page.

   ![Screenshot of VM tab, +Connect highlighted.](./media/storage-files-quick-create-use-windows/connect-vm.png)

1. In the **Connect to virtual machine** page, keep the default options to connect by **IP address** over **port number** *3389* and select **Download RDP file**.
1. Open the downloaded RDP file and select **Connect** when prompted.
1. In the **Windows Security** window, select **More choices** and then **Use a different account**. Type the username as *localhost\username*, where &lt;username&gt; is the VM admin username you created for the virtual machine. Enter the password you created for the virtual machine, and then select **OK**.

   ![Screenshot of VM login prompt, More choices highlighted.](./media/storage-files-quick-create-use-windows/local-host2.png)

1. You may receive a certificate warning during the sign-in process. Select **Yes** or **Continue** to create the connection.

### Map the Azure file share to a Windows drive

1. In the Azure portal, navigate to the *qsfileshare* fileshare and select **Connect**.
1. Select a drive letter then copy the contents of the second box and paste it in **Notepad**.

   :::image type="content" source="media/storage-how-to-use-files-windows/files-portal-mounting-cmdlet-resize.png" alt-text="Screenshot that shows the contents of the box that you should copy and paste in Notepad." lightbox="media/storage-how-to-use-files-windows/files-portal-mounting-cmdlet-resize.png":::

1. In the VM, open **PowerShell** and paste in the contents of the **Notepad**, then press enter to run the command. It should map the drive.

## Create a share snapshot

Now that you've mapped the drive, create a snapshot.

1. In the portal, navigate to your file share, select **Snapshots**, then select **+ Add snapshot**.

   ![Screenshot of storage account snapshots tab.](./media/storage-files-quick-create-use-windows/create-snapshot.png)

1. In the VM, open the *qstestfile.txt* and type "this file has been modified". Save and close the file.
1. Create another snapshot.

## Browse a share snapshot

1. On your file share, select **Snapshots**.
1. On the **Snapshots** tab, select the first snapshot in the list.

   ![Snapshots tab, first snapshot highlighted.](./media/storage-files-quick-create-use-windows/snapshot-list.png)

1. Open that snapshot, and select *qsTestFile.txt*.

## Restore from a snapshot

1. From the file share snapshot tab, right-click the *qsTestFile*, and select the **Restore** button.

    :::image type="content" source="media/storage-files-quick-create-use-windows/restore-share-snapshot.png" alt-text="Screenshot of the snapshot tab, qstestfile is selected, restore is highlighted.":::

1. Select **Overwrite original file**.

   ![Screenshot of restore pop up, overwrite original file is selected.](./media/storage-files-quick-create-use-windows/snapshot-download-restore-portal.png)

1. In the VM, open the file. The unmodified version has been restored.

## Delete a share snapshot

1. On your file share, select **Snapshots**.
1. On the **Snapshots** tab, select the last snapshot in the list and select **Delete**.

   ![Screenshot of the snapshots tab, last snapshot selected, delete button highlighted.](./media/storage-files-quick-create-use-windows/portal-snapshots-delete.png)

## Use a share snapshot in Windows

Just like with on-premises VSS snapshots, you can view the snapshots from your mounted Azure file share by using the Previous Versions tab.

1. In File Explorer, locate the mounted share.

   ![Screenshot of a mounted share in File Explorer.](./media/storage-files-quick-create-use-windows/snapshot-windows-mount.png)

1. Select *qsTestFile.txt* and > right-click and select **Properties** from the menu.

   ![Screenshot of the right-click menu for a selected directory.](./media/storage-files-quick-create-use-windows/snapshot-windows-previous-versions.png)

1. Select **Previous Versions** to see the list of share snapshots for this directory.

1. Select **Open** to open the snapshot.

   ![Screenshot of previous Versions tab.](./media/storage-files-quick-create-use-windows/snapshot-windows-list.png)

## Restore from a previous version

1. Select **Restore**. This copies the contents of the entire directory recursively to the original location at the time the share snapshot was created.

   ![Screenshot of previous versions, restore button in warning message is highlighted.](./media/storage-files-quick-create-use-windows/snapshot-windows-restore.png)
    
    > [!NOTE]
    > If your file has not changed, you will not see a previous version for that file because that file is the same version as the snapshot. This is consistent with how this works on a Windows file server.

## Clean up resources

[!INCLUDE [storage-files-clean-up-portal](../../../includes/storage-files-clean-up-portal.md)]

## Next steps

> [!div class="nextstepaction"]
> [Use an Azure file share with Windows](storage-how-to-use-files-windows.md)