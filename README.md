<img src="icon.png" width="64" align="right" />

# Environment Copy – Snapshot & Restore

XrmToolBox tool for Dataverse / Dynamics 365 environment refreshes.

Copying production over a sandbox (UAT, pre-production) wipes everything specific to that sandbox: flows are turned off, environment variables point to production values, plugin steps get their production state, users and teams get their production roles, configuration tables are replaced.

This tool takes a **snapshot** of the sandbox before the copy, then **compares** it with the sandbox after the copy and **restores only the differences you tick**.

- [Requirements](#requirements)
- [Install](#install)
- [1. Before the copy: take a snapshot](#1-before-the-copy-take-a-snapshot)
- [2. After the copy: compare and restore](#2-after-the-copy-compare-and-restore)
- [What is restored, component by component](#what-is-restored-component-by-component)
- [Power Automate trigger URLs](#power-automate-trigger-urls)
- [Not covered](#not-covered)
- [Snapshot folder](#snapshot-folder)
- [Good practices](#good-practices)
- [Feedback](#feedback)

## Requirements

- XrmToolBox 1.2025.10.74 or later.
- Online Dataverse, with an OAuth, client secret or certificate connection.
- System Administrator role in the sandbox (snapshot and restore).

## Install

In XrmToolBox, open the **Tool Library**, search for *Environment Copy*, then install.

## 1. Before the copy: take a snapshot

Connect XrmToolBox to the **sandbox** (not to production), then open the tab **1. Snapshot (before the copy)**.

| Setting | Use |
|---|---|
| Snapshots folder | Where snapshots are written. Each snapshot is a folder named `<environment>_<date>_<time>`. |
| Customer publisher prefixes | One or more prefixes (e.g. `abc, xyz`). Only the components of these publishers are saved: Microsoft flows, variables and plugin steps are left out. **Detect…** lists the publishers of the environment. |
| Configuration tables | Tables saved row by row (e.g. a parameter table). **Choose…** lists the tables of the environment. |
| Compare solutions with production | Optional: connect to production to record the solution versions of both environments. |
| Content to save | Component types to save. *Select all* ticks or unticks everything. |

Click **Take snapshot**. It is read-only: nothing is changed in the environment. The saved components are then listed type by type in **Snapshot content**, so that you can check them before starting the copy.

Settings are remembered per environment.

## 2. After the copy: compare and restore

Once the copy is done, connect XrmToolBox to the **sandbox** again and open the tab **2. Restore (after the copy)**. The latest snapshot of the environment is picked; **Browse…** picks another one.

1. Tick the sections to compare, then click **1. Compare with this environment**. Nothing is changed.
2. The differences are grouped by component type on the left. For each one the grid shows the value **now** (after the copy), the value in the **snapshot**, and what restoring it does.
3. Tick what to restore (*Select all* works on the type shown), then click **2. Apply selected actions** and confirm.
4. Each line gets its status: done, failed or skipped. Then handle the **To do manually** list.
5. Click **Compare** again: only what was not restored should remain.

Rows highlighted in yellow need a check before being applied (see [trigger URLs](#power-automate-trigger-urls)); they are unticked.

A log of what was applied (`restore_<date>_log.json`) is written into the snapshot folder.

## What is restored, component by component

| Component | Saved | After the copy |
|---|---|---|
| Users | Roles, teams and column security profiles of active users. System accounts and Microsoft application users are left out. | Missing roles, teams and profiles are added; roles that come from production are removed, so that users get back exactly their sandbox roles. The account running the restore never loses a role. Missing application users are recreated; missing Entra users are listed with the `pac admin assign-user` command to add them. |
| Teams | Security roles of owner and Entra group teams. | Missing roles are added, roles from production are removed. A team missing from the copy is reported, never recreated. |
| Environment settings | Organization settings, organization setting and app setting values. | **Compared only**: each difference is listed in *To do manually*, nothing is changed. |
| Theme | Custom themes and the published theme. | Colors restored, theme recreated if missing, then published. |
| Environment variables | Current value of the customer's variables. | Sandbox value restored; if the sandbox used the default value, the copied value is deleted. |
| Configuration tables | Whole rows of the chosen tables. | Rows compared column by column (lookups and state included), then updated or recreated. Rows only in the copy are kept and reported. |
| Plugin steps | On/off state of the steps shipped by the customer's solutions (Microsoft and ActivityFeeds assemblies left out). | On/off state restored. |
| SharePoint sites | Site declarations and their addresses. | Address of the existing site declarations restored, so that documents go to the sandbox SharePoint. A missing declaration is reported. |
| Flows, business rules, workflows | On/off state (flows owned by application users left out). | On/off state restored: classic processes first, then flows, then parent flows. A flow that cannot be turned on because of a connection is reported with its connectors and connection references. |
| Solutions | Versions, compared with production if connected. | Information only. |

## Power Automate trigger URLs

A flow with an HTTP trigger gets a **new URL** with the copy. A value that holds such a URL (environment variable, configuration table, …) is flagged with **⚠**: neither the saved value nor the copied one works. Its line is unticked and an item reminds you to turn the flow on, copy its new URL and paste it.

## Not covered

- Connection references, service endpoints (webhooks, Service Bus), mailboxes and queues: not saved for now.
- Secrets (secure configurations, keys, passwords) are never read.
- The content of flows is not saved, only their on/off state. Values that differ between environments belong in environment variables.

## Snapshot folder

One JSON file per component type, readable in any editor:

| File | Content |
|---|---|
| `00_manifeste.json` | Environment, date, prefixes, tables, saved sections |
| `01_processus.json` | Flows, business rules and processes |
| `02_variables-environnement.json` | Environment variables |
| `04_plugins.json` | Plugin steps |
| `05_utilisateurs.json` | Users, roles, teams |
| `06_organisation.json` | Environment settings |
| `07_themes.json` | Themes |
| `09_sharepoint.json` | SharePoint sites |
| `11_solutions.json` | Solution versions |
| `tables/<table>.json` | Rows of each configuration table |

The folder holds configuration and user names: store it like any other environment data.

## Good practices

- Take a manual backup of the sandbox in the Power Platform admin center before applying.
- Test the tool on a non-critical environment first.
- After the copy, the sandbox is in administration mode with background operations off: restore, handle the items to do manually, then turn administration mode off.

## Feedback

Bugs and ideas are welcome in the [issues](../../issues). Please never post secrets, connection strings or customer data: environment names and error messages are enough.

This repository holds the documentation and the issues of the tool; the source code is not published.

Author: Saad ZAGHLOUL.
