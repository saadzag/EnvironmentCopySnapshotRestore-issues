<img src="icon.png" width="64" align="right" />

# Environment Copy – Snapshot & Restore

XrmToolBox tool for Dataverse / Dynamics 365 environment refreshes.

Copying production over a sandbox (UAT, pre-production) wipes everything specific to that sandbox: flows are turned off, environment variables point to production values, plugin steps get their production state, users and teams get their production roles, configuration tables are replaced.

This tool takes a **snapshot** of the sandbox before the copy, then **restores only the differences** after it.

## What is saved and restored

- Cloud flows, business rules, classic workflows, actions and business process flows: on or off
- Plugin steps: enabled or disabled
- Environment variables (current values)
- Users: roles, teams, column security profiles; application users
- Teams: security roles (a team missing from the copy is reported, never recreated)
- Theme
- SharePoint sites: address of the existing site declarations
- Configuration tables of your choice: whole records
- Environment settings (plugin trace log, audit, session timeouts…): compared only, differences are listed
- Solution versions compared with production

Values holding a Power Automate trigger URL are flagged with ⚠ and left unticked: the URL changes with the copy.

Components can be filtered by **publisher prefix**, so that Microsoft flows, variables and plugin steps are left out.

## How it works

1. **Before the copy** – connect to the sandbox, open the *Snapshot* tab, tick the content to save and take the snapshot (read-only). A timestamped folder is created, with one JSON file per component type; the saved components are listed type by type in the tool.
2. **After the copy** – open the *Restore* tab, pick the snapshot and run the analysis: it is a dry run that lists every difference, with its current and saved value. Tick what to restore, then apply. What cannot be restored automatically is listed as items to do manually. A log is written next to the snapshot.

Roles that come from production are removed, so that users and teams get back exactly their sandbox roles (never for the account running the restore). Extra rows and steps are only reported. Secrets (secure configurations, keys) are never read.

Online Dataverse only (OAuth, client secret or certificate connections).

## Install

In XrmToolBox, open the **Tool Library**, search for *Environment Copy*, then install.

## Feedback

Bugs and ideas are welcome in the [issues](../../issues). Please never post secrets, connection strings or customer data: environment names and error messages are enough.

This repository holds the documentation and the issues of the tool; the source code is not published.

Author: Saad ZAGHLOUL.
