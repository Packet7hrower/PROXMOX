
**Updating Proxmox is critical to ensure proper security and operation of your Hypervisor**

Version Control: This guide is built assuming you're running Proxmox VE 8.3.

Housekeeping: This guide is for the GUI Method, not SSH.

Let's start by browsing to your Host or Cluster. Once you're logged in, lets ensure the proper Repositories are configured for you enviroment. If you're in a clustered enviorment, you'll need to ensure each node is configured.

- Select your Node > Updates > Repositories. Take note of the APT Repo you're working in. The core updates are for /etc/apt/sources.list. If you have installed CEPH, you may have additional files to update.
  
  - In the Column "Suites", ensure bookwork, bookworm-updates, bookworm-security are all listed. (This should already be there)
    
  - You should also have a fourth "Origin" item listed for Proxmox, or "pve". If it'snot listed, click "Add" at the top left, and select pve-no-subscription, and add it.
    
    - ![image](https://github.com/user-attachments/assets/f248413f-28c9-441d-868f-b4758ead833d)
      
- Once you've ensured the above four sources are added, click on Updates, Refresh, then Upgrade. The console will open. Confirm the download size, and hit enter. (Note - Proxmox will warn about the License if you're using the no-subscription. This is normal)
  
    - ![image](https://github.com/user-attachments/assets/dc8d9378-e904-4a43-a42d-78500cb310ea)
      
- Now that you have the latest updates installed, it's always best practice to reboot the Node.
  
  - If you only have one node, do a clean shutdown on your VMs & Containers, and reboot your Node. If you're in a Clustered Enviroment, you can either live-migrate One/Some of your VMs / Containers to maintain uptime

- Once your Node reboots, verify your VMs / Containers start back. If you migrated VMs / Containers to another node, feel free to move them back, and proceed with updating the other Node(s) in your enviroment.

- You can verify all updates have been installed, by going tack to Updates, clicking Refresh, and ensure no results No updates are available.

- Congradulations - you've now oficially updated Proxmox!
