# Deploying the Environment


## RHPDS

1. Log in to the [Red Hat Demo System](https://demo.redhat.com)
2. In the catalog, search for "Ansible Automation Platform 2 & Satellite Workshop"
3. Select Size "Training" so you only get one instances, and fill out the other values before submitting
4. Wait for the workshop to deploy
5. Click the link provided in the resulting email e.g. [http://<GUID>.example.opentlc.com](http://<GUID>.example.opentlc.com)
6. Enter your name and an email address (note that these aren't recorded anywhere).
7. Note the workbench SSH user, likely to be student1, and the workbench password
8. Note the workbench SSH control node DNS access command, e.g. `ssh student1@student1.<GUID>.example.opentlc.com`

# Prepare Satellite and the RHEL nodes using AAP Controller

Note: Launch each template once it has been created.

1. Complete all setup tasks for Exercise 0: Setup here:
[https://aap2.demoredhat.com/exercises/ansible_smart_mgmt/0-setup/](https://aap2.demoredhat.com/exercises/ansible_smart_mgmt/0-setup/)

    - EXCEPTION: When running the "Server / RHEL7 Register" template, do not run it as per the instructions above.  Instead,run it 3 times, assigning the hosts to their appropriate environments as follows:
       - node1 -> Dev
       - node2 -> QA
       - node3 -> Prod
    - Note: You can skip CentOS-related exercises if not demonstrating convert2rhel

2. Go to Settings -> Job settings and set the following parameters:

    - Enable Role Download: On
    - Enable Collection(s) Download: On
    
3. Create a new project with the following parameters:

    - Name: DEMO Satellite Demo Config
    - SCM type: GIT
    - SCM URL: [https://github.com/benblasco/satellite_demo_config](https://github.com/benblasco/satellite_demo_config)
    
    - Branch: main
    - Options: Clean; Update Revision on Launch

4. Configure Satellite Remote Execution by creating a template with the following parameters and then launching it:

    - Name: DEMO Satellite Remote Execution
    - Inventory: Workshop Inventory
    - Project: DEMO Satellite Demo Config
    - Execution Environment: smart_mgmt workshop execution environment
    - Playbook: satellite_config_rex.yml
    - Credential type: Satellite_Collection
    - Credential name: Satellite Credential
    - Privilege escalation: yes (even if possibly redundant as it's in the playbook)

5. Install RHEL System Roles in Satellite by creating a template with the following parameters and then launching it.  
    Note 1: This requires your RHN username and password.  It will register the system, install the roles, and then immediately unregister the system.  
    Note 2: The template below includes surveys.  If you want to bypass this just add the variables in the survey as extra_vars when creating the template, with the only caveat being that your RHN password will be seen as plain text.

    - Name: DEMO Satellite Install System Roles
    - Inventory: Workshop Inventory
    - Project: DEMO Satellite Demo Config
    - Execution Environment: smart_mgmt workshop execution environment
    - Playbook: satellite_install_system_roles.yml
    - Credential type: Satellite_Collection
    - Credential name: Satellite Credential
    - Credential type: Machine
    - Credential name: Workshop Credential
    - Privilege escalation: yes (even if possibly redundant as it's in the playbook)
    - Save
    - Survey -> Add
    - Question: RHN Username
    - Answer variable name: rhn_username
    - Answer type: Text
    - Required: Yes
    - Save
    - Survey -> Add
    - Question: RHN Password
    - Answer variable name: rhn_password
    - Answer type: Password
    - Required: Yes
    - Save
    - Enable Survey via the slider

6. Enable RHEL Remote Execution by creating a template with the following parameters and then launching it:

    - Name: DEMO RHEL Remote Execution
    - Inventory: Workshop Inventory
    - Project: DEMO Satellite Demo Config
    - Execution Environment: smart_mgmt workshop execution environment
    - Playbook: rhel_configure_rex.yml
    - Credential type: Machine
    - Credential name: Workshop Credential
    - Limit: rhel (possibly redundant as it's in the playbook)
    - Privilege escalation: yes (even if possibly redundant as it's in the playbook)

7. (Partially complete) Configure RHEL host groups and collections by creating a template with the following parameters and then launching it:

    - Name: DEMO Satellite Configure RHEL hosts
    - Inventory: Workshop Inventory
    - Project: DEMO Satellite Demo Config
    - Execution Environment: smart_mgmt workshop execution environment
    - Playbook: satellite_config_hosts.yml
    - Credential type: Satellite_Collection
    - Credential name: Satellite Credential
    - Privilege escalation: yes (even if possibly redundant as it's in the playbook)

    Note: You will need to manually add the relevant hosts to the created host groups and host collections, as this has not yet been automated.

8. Continue with any other configuration you want to perform as per the workshop instructions.

## Config in Satellite

### Synchronise the repositories

1. Navigate to Content -> Sync Status -> Red Hat Enterprise Linux Server -> 7Server -> x86_64
2. Tick "Red Hat Enterprise Linux 7 Server RPMs x86_64 7Server" and "Red Hat Enterprise Linux 7 Server - Extras RPMs x86_64"
3. Click the "Synchronise Now" button to pull the latest content.

### Create new version of the Content View (CV) for each lifecycle environment (LCE)

1. Create a filter of type "RPM" that includes all packages without errata (ie the base versions of the RHEL packages)
    Note ticking of "include all RPMs with no errata"
2. Create a filter of type "Errata " to only include errata up to 2023-04-30
3. Publish the Content View
    Now you can see what version 2.0 looks like, as it has significantly less packages available and less errata.
4. Edit and rename the filter to include errata up to 2023-05-30
5. Publish the Content View
6. Edit and rename the filter to include errata up to 2023-06-22
5. Publish the Content View
6. Promote v2.0 to prod
7. Promote v3.0 to QA
8. Promote v4.0 to Dev

### ~~TO BE RETESTED Register the servers using AAP template:~~

Next, run the SERVER / RHEL7 - Register job template by clicking the "launch" button to launch.

Node1 -> prod
Node2 -> dev
Node1 -> qa



Node1 updates
Install    1 Package  (+2 Dependent packages)
Upgrade  116 Packages

Node2 updates
Install    1 Package  (+2 Dependent packages)
Upgrade  117 Packages

Node3 updates
Install    1 Package  (+2 Dependent packages)
Upgrade  118 Packages

### ~~PARTLY AUTOMATED Create host collection~~

Create a hosts collection called "All RHEL hosts collection" and add all 3 RHEL hosts to it:

1. Hosts -> Host Collections -> Create Host Collection
2. Name "All RHEL hosts collection"
3. Save
4. Go to Hosts tab
5. Click Add
6. Add the RHEL hosts only


### PARTLY AUTOMATED Create host group

1. Configure -> host groups
2. Name: "All RHEL hosts group"
3. Click "Submit" button
4. Go to Hosts -> All Hosts
5. Select all RHEL hosts
6. Click Actions -> Change group
7. Select "All RHEL hosts group" from the drop down
8. Click "Submit" button
9. Go back and check that all RHEL hosts belong to the correct group (see Host Group) column

# Make sure you can show SCAP compliance

Import the foreman-scap-client role as per instructions here

[https://www.redhat.com/en/blog/deploying-openscap-satellite-using-ansible](https://www.redhat.com/en/blog/deploying-openscap-satellite-using-ansible)

# Fix subscription manifest if it goes stale

Content -> subscriptions

Manifest refresh

# Pull updates on demand?

Check if you want to pull updates on demand for each repo, or whether you would prefer to keep a downloaded copy for speed and reliability.
