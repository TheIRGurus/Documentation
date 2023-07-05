# Creating an MSSP install
Welcome.  Here we will set up the MSSP variant of the IBM SOAR product.

## Setting the Time Zone
Always a good idea to set the time zone early in an install

1. Enter the following command in the SSH client to list the available time zones:
`timedatectl list-timezones`
    - Determine the time zone that you want to use, such as America/New_York

2. Enter the following command to change the time zone.

    - Command Template:
    ```bash
    sudo timedatectl set-timezone <time_zone>
    ```
    >Example: `sudo timedatectl set-timezone America/New_York`

3. Restart the resilient-messaging.service with the following command
    - Command:
    ```bash
    sudo systemctl restart resilient-messaging.service
    ```


## Creating a Configuration Organization
>Official KB Link: https://www.ibm.com/docs/en/sqsp/49?topic=organizations-creating-configuration-organization

1. Important resutil options

    If needed, you may want to customize the install further using the available options in the ***neworg*** command. We can set the **Configuration Type, Incident Prefix Code**, and the **Incident Sequence Index**

    - **configtype** - The configuration type of the new organization.  Valid values are standard, global_dashboard, configuration, child.
The default is standard.
Standard orgs are the regular install and do not participate in MSSP configurations.

    - **incseqcodeprefix** - The prefix for the Incident Sequence Code.  Customization is valid only for Child and Standard organizations.  A maximum of 10 characters is allowed.

    - **incseqcodestartindex** - The starting index for the Incident Sequence Code.  This option is only set on organization creation and can not be set after that.  This customization is only valid for Child and Standard organizations.


2. Create new org with preferred options from above.
    - Command:
    ```bash
    sudo resutil neworg -name <config org name> -configtype configuration
    ```
    >Example: `sudo resutil neworg -name MSSP_Config -configtype configuration`

---

# Setting up IBM SOAR MSSP

## Creating a Global Dashboard
>Official KB Link: https://www.ibm.com/docs/en/sqsp/49?topic=organizations-creating-global-dashboard

- Command:
    ```bash
    sudo resutil neworg -name <dashboard org name> -configtype global_dashboard -parentorg <name of config org>
    ```
    >Example: `sudo resutil neworg -name MSSP_Dashboard -configtype global_dashboard -parentorg MSSP_Config`


## Creating the child organization(s)
>Official KB Link: https://www.ibm.com/docs/en/sqsp/49?topic=organizations-creating-child-organization

- Command:
    ```bash
    sudo resutil neworg -name <name> -configtype child -parentorg <global_dashboard_orgname>
    ```
    >Example: 
    >
    >`sudo resutil neworg -name Child_A -configtype child -parentorg MSSP_Dashboard`
    >
    >`sudo resutil neworg -name Child_B -configtype child -parentorg MSSP_Dashboard`
    >
    >`sudo resutil neworg -name Child_C -configtype child -parentorg MSSP_Dashboard`


## Setting up Master Admin access to the Configuration Org
- Command:
    ```bash
    sudo resutil newuser -email <email> -first <first_name> -last <last_name> -org <global_dashboard_orgname>
    ```

    >Example:
    >
    >`sudo resutil newuser -email cfin@irgurus.com -first Craig -last Finley -org MSSP_Config`
    >
    >`sudo resutil newuser -email nick@irgurus.com -first Nick -last Mumaw -org MSSP_Config`

---

## Setting up MSSP users

### Step 1 - Configure users and groups for each child org:
We can avoid issues with group memberships that might not be available until a configuration push has been completed by Inviting the users to group memberships first, followed by sending email invitations to the child org users.

We define the following groups in this tutorial:

### Step 2 – Performing the Configuration Push
A good practice is to set up the groups and perform a ***'Configuration Push'*** down to the child orgs and then send the email invitations.  Wise to do this to ensure they have proper access before they log in for the first time.

All administration and customization settings, except user accounts and API key accounts, are propagated to the global dashboard and child organizations.

Once the user accounts are prepared, the next step is to push the configuration.
Select ***Administrator Settings --> Configuration Push*** to propagate the configuration to the child orgs.
The install will prompt for a reason for this configuration push.
The results can be shown in a table for the orgs.


### Step 3 - Send email invites to users

The next step is to create users by sending them an email invitation from the configuration organization.

When the invitation is sent from the configuration organization, users are invited to the configuration organization by default and any other organizations you specify.

---

## Useful Links
MSSPs add-on deployment overview: https://www.ibm.com/docs/en/sqsp/49?topic=overview-mssps-add-deployment
