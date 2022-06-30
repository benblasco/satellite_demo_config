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

Note: This functionality is currently broken in Satellite 6.10 due to a bug.  You will see an error message saying: "This action uses katello-agent, which is currently disabled. Use remote execution instead."
Bug link: [https://bugzilla.redhat.com/show_bug.cgi?id=2029192](https://bugzilla.redhat.com/show_bug.cgi?id=2029192)
This bug has been added to advisory RHBA-2022:96560 by Patrick Creech (pcreech@redhat.com)

1. Go to Content -> Errata
2. Select only RHEL 7 server repos
3. Explain Applicable and Installable
4. Click applicable
5. Select 100 per page
6. Click on the applicable checkbox
7. Find the first installable errata for all 3 hosts: RHSA-2020:5566 Important: openssl security update
8. Note the date is Dec 16
9. Click the Apply Errata button to install it and fix the issues on all impacted hosts
10. Note that it uses Ansible
11. Click and show the log for one of the hosts

Optional tasks:

1. Go back and click on RHSA-2021:2575 Moderate: lz4 security update
2. Run it and fix the issues on just the one host
3. Go to content views to explain why the content is not accessible to the prod host

# Installing/Updating a package

1. Go to the Hosts -> Host Collections -> All (RHEL) hosts collection
2. Click "Package Installation, Removal, and Update"
3. Enter “nano” package name (editor) (or screen, vim-enhanced, bash-completion)
4. Install -> via remote execution
5. Point out that you could do an “update all packages” but it would take a long time, so we won’t be doing that!
6. Click Done
7. Watch the install

# Check if a reboot is required

1. Go to Hosts -> All Hosts
2. Add filter “trace_status=reboot_needed” (without the quotation marks), and press "Search".

Refer to [https://access.redhat.com/discussions/3175851](https://access.redhat.com/discussions/3175851)

# Run a remote job

1. Go to hosts -> All Hosts
2. Select action -> schedule remote job
3. Fill in text: hostname; uptime; whoami
4. Submit
5. Wait for success
6. Click on a host and check the output

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
11. Click on the timesync_ntp_servers parameter. 
12. Click the "Override" check box.

Paste the line below exactly as is into the "Default Value", and then click "Submit"  
```
[{"hostname":"0.au.pool.ntp.org","iburst":"yes"},{"hostname":"1.au.pool.ntp.org","iburst":"yes"}]
```  
You will now see that the value has a flag next to it to tell us that it has been overridden.
13. Go back to hosts -> all hosts
14. Alt tab to CLI of node1
15. Run the following commands on the host  
```
more /etc/chrony.conf
chronyc sources
```
16. Alt tab Back to GUI
17. Ensure the hosts are selected
18. Action -> Run all Ansible roles
19. Watch it execute on a specific node
20. Go back and check all completed successfully
21. Check on the CLI again  
```
more /etc/chrony.conf
chronyc sources
```
22. Summarise how you have applied a configuration at scale across a group of hosts

# Show SCAP compliance

1. Go to Hosts -> Policies
2. Click on Dashboard
3. Explain that this is better for a truly disconnected environment… but ask how disconnected they need to be.
4. Can switch to console.redhat.com as needed and explain that there you can run remediation here if you have a smart management subscription

# Demonstrate Insights

Demonstrate this via cloud.redhat.com because our lab hosts are not connected to Insights.

# Other capabilities to discuss

- Tailoring SCAP policies
- Generating reports (Monitor -> Report Templates)
    [https://www.redhat.com/en/blog/getting-started-satellite-65-reporting-engine](https://www.redhat.com/en/blog/getting-started-satellite-65-reporting-engine)
- Remote execution jobs


