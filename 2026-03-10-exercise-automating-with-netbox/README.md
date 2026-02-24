# Exercise - Automating with Netbox

Netbox is the go-to solution for modeling and documenting network infrastructure. As a successor to legacy IPAM and DCIM applications, NetBox provides a cohesive, extensive, and accessible data model for all things networked. By providing a single robust user interface and programmable APIs for everything from cable maps to device configurations, NetBox serves as the central source of truth for the modern network.

Netbox can be used as the **source of truth** for the Ansible inventory (a *dynamic* inventory), but its REST-API can also be used to create objects like VMs, racks, cables, ... with Ansible. With Ansible Netbox modules an existing infrastructure can be *fact-gathered* and the results written to Netbox (e.g. for initial filling).

> [!IMPORTANT]
> **Have fun!**  
> If you have any questions, let us know!

**Open up your development environment and start hacking!**  
You'll (most likely) need to install some additional packages and collections. If you run into any issues with dependencies not being found (especially when you are working in a *devcontainer*), **start a new terminal after installing your dependencies!**

## 1. Access Netbox

We will be using [https://demo.netbox.dev/](https://demo.netbox.dev/) as our Netbox instance. Create a new account for you by [accessing this website](https://demo.netbox.dev/plugins/demo/login/), choose a **username** and **password**.  

**Achieve the following:**

✅ Netbox Login Credentials created

## 2. Create API Token

Once you are logged in to Netbox, click your username in the top-right corner and choose **API-Token**.  
Create a new Token with the green button on the right.  Use **Version `V1`** and click the checkbox for a **Write-enabled** Token.  
**Before** hitting save, **copy the API-Token**.

**Achieve the following:**

✅ Netbox API Token created

## 3. Dynamic inventory

The Netbox is already filled with some data, take a look at [all **Virtual Machines**](https://demo.netbox.dev/virtualization/virtual-machines/), we want to retrieve these machines and *automate* against all hosts from *cluster* **DO-FRA1** (Servers deployed on *Digital Ocean* in the *Frankfurt* DC).

**Create an inventory file which will retrieve all VMs from Netbox**, an [inventory plugin](https://docs.ansible.com/projects/ansible/latest/collections/index_inventory.html) is available (**find the correct one**).

> [!TIP]
> It makes sense to first **filter** the **Query** for all hosts in the **EMEA** region, this is done in a [cluster group](https://demo.netbox.dev/virtualization/cluster-groups/).

Now, you can **group** the hosts for Ansible **by** certain properties like *tenant*, *platform*, etc. We want to automate all hosts from the **DO-FRA1** [*cluster*](https://demo.netbox.dev/virtualization/clusters/). **That group can be used in the playbook `playbook_use_hosts_from_netbox.yml`**, you'll need to adjust the `hosts` value with the correct value.

Use the `ansible-inventory` utility to check what will be returned by Netbox (use the `--graph` parameter for better readability).

```console
ansible-inventory -i <inventory-file> --graph
```

<p>
<details>
<summary><b>Help needed?</b></summary>

Create a new file `netbox_inventory.yml`.  

Provide the `plugin` key with the value `netbox.netbox.nb_inventory`. Provide the `api_endpoint` key with the value `https://demo.netbox.dev/`.

Filtering for the `emea` cluster group can be done in the `query_filters` list, this will return all hosts without any additional grouping.  

Use the `group_by` key to (automatically) create additional groups in regards to the `cluster`.

</details>
</p>

If you identified a useful group returned by the Netbox inventory, **use** the group in the playbook and *run* the playbook.

**Achieve the following:**

✅ Inventory file for Netbox created  
✅ Netbox collection (and Python dependencies) are installed and usable  
✅ VMs of `DO-FRA1` are retrieved, checked with `ansible-inventory`  
✅ Inventory-Group is used in `playbook_use_hosts_from_netbox.yml`

### Bonus

You'll need to provide the API Token in your inventory file, which would end up in Git.  

Use an environment variable to provide the token.

## 4. Write content to Netbox

Create a new VM object in Netbox, it should represent your *workstation* (your Ansible control node).  
Find an appropriate module and add a task to the existing `playbook_write_content_to_netbox.yml`. The playbook already contains some variables which you **can** use for authentication to Netbox, but you can overwrite them or not use them if you don't need it.  

> [!TIP]
> **You'll need a *write-enabled* token, if your token does not have write permissions, adjust it in the [Netbox UI](https://demo.netbox.dev/user/api-tokens/).**

The *data* of the VM object should include some **facts** of your local machine:

| Parameter   | Ansible fact variable or String                                                  | Description                                                                                                                       |
| ----------- | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **Name**    | <nobr>`(ansible_facts['hostname'] ~ ansible_facts['user_id']) \| to_uuid`</nobr> | A combination of hostname and username which is converted to a UUID to prevent exposing personal information to the public Netbox |
| **VCPUs**   | `ansible_facts['processor_vcpus']`                                               | Automatically gathered fact by Ansible                                                                                            |
| **Memory**  | `ansible_facts['memtotal_mb']`                                                   | Automatically gathered fact by Ansible                                                                                            |
| **VM Role** | `Application Server`                                                             | A (pre-existing) *role* or *label* for the VM                                                                                     |
| **Cluster** | `ACC Workstations`                                                               | A (pre-existing) *cluster* name to easy filter for all of our VM objects                                                          |

Use this ad-hoc command to view all facts used for the VM creation in Netbox:

```console
ansible -i localhost, all -m setup -a 'filter=ansible_hostname,ansible_user_id,ansible_processor_vcpus,ansible_memtotal_mb'
```

The parameters above should be configured by the Ansible module, add these with the appropriate module parameters.

Once you are finished, run the playbook:

```console
ansible-playbook playbook_write_content_to_netbox.yml
```

> [!TIP]
> **Run the playbook with `-v` to see all values of your variables and returned content from the Ansible-Netbox module.**

**Achieve the following:**

✅ Module identified and dependencies installed  
✅ Task added to `playbook_write_content_to_netbox.yml` with all necessary parameters  
✅ VM object is created by playbook and visible in Netbox UI  

### Success

🎉 **Congrats, you did it! Hope you had fun!**
