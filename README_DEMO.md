# Intro

Explain we won’t be talking about provisioning today, but how we can do it on VMWare, public cloud, bare metal of course etc.
1. Go to hosts -> content hosts and show all the hosts
2. Go to content -> Lifecycle environments and explain paths, and how hosts are associated with them
3. Explain the lifecycle environments.  Explain that we have versions for these: dev/qa/prod, but they could have 
    a. Database servers
    b. Web servers
    c. etc.
4. Go to content -> content views.  Show how the software is versioned and promoted through the lifecycle environments

# Applying a patch (errata)

## Setup (manual)

Note 1: This is based on the knowledge shared in this comment within a KB article [https://access.redhat.com/discussions/2913231#comment-1148661](https://access.redhat.com/discussions/2913231#comment-1148661).  Please make it a priority to read this article before proceeding.  

### Publish the Content Views

Note: Steps below are carried out in Satellite.

1. Go to Content -> Content Views
2. Select the RHEL7 CV
3. Click "Yum Content" -> "Repositories" and note the repos that are included in the CV
4. Click "Yum Content" -> "Filters"
5. Click the "New Filter" button and enter the following parameters:
    - Name: Base packages with no errata
    - Content Type: Package
    - Inclusion type: Include
6. Click Create Filter
7. Check the "Include all RPMs with no errata" slider. It will turn from grey to blue
8. Click "Yum Content" -> "Filters"
9. Click the "New Filter" button and enter the following parameters:
    - Name: Errata to 2024-02-28
    - Content Type: Errata - by date range
    - Inclusion type: Include filter
10. Click Create Filter
11. Leave all the "Errata Type" boxes ticked (Security, Enhancement, Bugfix)
12. Select Date Type "Updated On"
13. Set the End Date to 2024-02-28
14. Click Edit Rule to save the changes
15. Click "Publish New Version"
16. Wait for the CV to publish.  It will take ~20 minutes. Be Patient!
17. Edit the Errata filter, change the end date to 2024-04-30, and publish again
18. Edit the Errata filter, change the end date to 2024-07-31, and publish again
19. You will then have multiple versions of the CV, and each newer version should contain more errata than the previous version.

### Promote the content views

Note: Steps below are carried out in Satellite.

1. Go back to Content Views -> RHEL 7
2. Click "Versions"
3. Promote the versions as follows:
    - Most recent version (N) to RHEL7_Dev
    - Next most recent version (N-1) to RHEL7_QA
    - Next most recent version (N-2) to RHEL7_Prod

### Move your hosts to the correct Lifecycle Environments

Note: Steps below are carried out in AAP Automation Controller

1. Move the hosts into different lifecycle environments using the "Server / RHEL 7 - Register" template:
    - node1 to "Prod"
    - node2 to "QA"
    - node3 stays in "Dev" (ie. do nothing)
2. Run the "EC2 / Set instance tags based on Satellite facts" template
3. Run the "EC2 / Set instance tag - RHEL" template
4. Run the "CONTROLLER / Update inventories via dynamic sources" template for each environment: Dev, QA, Prod

Note: Steps 2-4 above are required to update the inventory in AAP Controller in case that is used elsewhere.

## Demo

1. Go to Content -> Errata
3. Explain Applicable and Installable
5. Select 100 per page
6. Click on the "Applicable" checkbox
7. Find the first installable errata for all 3 hosts: RHBA-2024:0856 subscription-manager bug fix update 
8. Note the date is Feb 19 2024, before the first filter end date created
9. Click the Apply Errata button to install it and fix the issues on all impacted hosts
10. Note that it uses Ansible
11. Click and show the log for one of the hosts

Optional tasks:

1. Go back to the Errata view, click the applicable checkbox, and select  RHSA-2024:3669 Important: less security update 
2. Run it and fix the issues on just the one host
3. Go to content views to explain why the content is not installable on the Prod and QA hosts

# Installing/Updating a package

1. Go to the Hosts -> Content Hosts
2. Select all 3 RHEL hosts
3. Click the drop-down next to the "Select Action" button, and select "Manage Packages"
4. Enter “nano” package name (editor) (or screen, vim-enhanced, bash-completion)
5. Click the drop down next to "Install" and select "via remote execution"
6. Point out that you could do an “update all packages” but it would take a long time, so we won’t be doing that!
7. Click Done
8. Watch the install, which completes quite quickly

# Check if a reboot is required

1. Go to Hosts -> All Hosts
2. Add filter `trace_status=reboot_needed` via the search box, and press "Search".

Refer to [https://access.redhat.com/discussions/3175851](https://access.redhat.com/discussions/3175851)

# Run a remote job

1. Go to Hosts -> All Hosts
2. Select all 3 hosts
3. Select action -> schedule remote job
4. Fill in text: `hostname; uptime; whoami`
5. Click "Submit"
6. Wait for success
7. Click on a host and check the output

# Demonstrate system roles

Using instructions from: 
[https://www.redhat.com/en/blog/satellite-host-configuration-rhel-system-roles-powered-ansible](https://www.redhat.com/en/blog/satellite-host-configuration-rhel-system-roles-powered-ansible)
and
[https://www.redhat.com/en/blog/advanced-ansible-variables-satellite](https://www.redhat.com/en/blog/advanced-ansible-variables-satellite)

Preparation steps:
1. Configure -> Ansible Roles
2. Press the button "Import from satellite.example.com"
3. Check the "Select all" box and click "Submit"

Execution steps:
1. Open a terminal CLI session on node1
2. Start with Hosts -> all hosts view
3. Explain that all three hosts have been associated with the same host group
4. Click the "All hosts host group" you can see in the "Host Group" column
5. Click Ansible roles
6. Add rhel-system-roles.timesync if not already added
7. Submit, and explain that now we need to check the configuration
8. Click on the "Variables" button under the actions for the role
9. Click on the timesync_ntp_provider parameter. 
10. Click the "Override" check box, and paste the line below exactly as is into the "Default Value"  
`chrony`  
11. Click "Submit".  You will now see that the value has a flag next to it to tell us that it has been overridden.
12. Click on the timesync_ntp_servers parameter. 
13. Click the "Override" check box, and paste the line below exactly as is into the "Default Value"  
`[{"hostname":"0.au.pool.ntp.org","iburst":"yes"},{"hostname":"1.au.pool.ntp.org","iburst":"yes"}]`  
14. Click "Submit".  You will now see that the value has a flag next to it to tell us that it has been overridden.
15. Go back to hosts -> all hosts
16. Alt tab to CLI of node1
17. Run the following commands on the host  
```
more /etc/chrony.conf
chronyc sources
```
15. Alt tab Back to GUI
16. Ensure the hosts are selected
17. Action -> Run all Ansible roles
18. Watch it execute on a specific node
19. Go back and check all completed successfully
20. Check on the CLI again  
```
more /etc/chrony.conf
chronyc sources
```
21. Summarise how you have applied a configuration at scale across a group of hosts

# Show SCAP compliance

1. Go to Hosts -> Policies
2. Click on Dashboard
3. Explain that this is better for a truly disconnected environment… but ask how disconnected they need to be.
4. Can switch to console.redhat.com as needed and explain that there you can run remediation here if you have a smart management subscription

Alternatively, follow the detailed instructions located in the exercise here:
https://github.com/ansible/workshops/tree/devel/exercises/rhdp_auto_satellite/1-compliance

# Demonstrate Insights

Demonstrate this via console.redhat.com because our lab hosts are not connected to Insights.

# Other capabilities to discuss

- Tailoring SCAP policies
- Generating reports (Monitor -> Report Templates)
    [https://www.redhat.com/en/blog/getting-started-satellite-65-reporting-engine](https://www.redhat.com/en/blog/getting-started-satellite-65-reporting-engine)
- Remote execution jobs


