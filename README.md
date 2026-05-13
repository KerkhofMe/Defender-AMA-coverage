***

# AMA vs Defender Coverage Workbook

#### ⚠️ This workbook assumes Microsoft Defender XDR data is ingested into Sentinel. Without ingestion, device name normalization and correlation may be inconsistent. To workaround that, copy the KQL query from the Github page and run it in Advanced Hunting in the Defender Portal (https://security.microsoft.com). 

When running the KQL query, the **AMA presence** in the first table is inferred from the `Heartbeat` table within the selected time window — not from the actual extension state. The reason is that the real installation state is only available via an Azure Resource Graph (ARG) call. As a result, a device may show as `No AMA or No Heartbeat` / `MDE Only (no AMA heartbeat)` even when the AMA extension is installed but not reporting (for example: VM powered off, network blocked, AMA service stopped, or no DCR associated).

To make this explicit, the query and workbook expose two separate columns:

- `HeartbeatSeen` — `Yes` / `No`, based purely on the `Heartbeat` table
- `AMAStatus` — `Heartbeat seen` or `No AMA or No Heartbeat`

The **merged view** at the bottom of the workbook (`Merge - MDEvsAMA + DCR`) cross-checks this with `hasAMAExt` / `amaExtVersion` from Azure Resource Graph and is the authoritative source for whether the AMA extension is actually installed.

## Overview

This Microsoft Sentinel Workbook provides visibility into Microsoft Defender for Endpoint (MDE)–managed devices and their telemetry coverage within Sentinel. It helps security and operations teams verify that devices are properly configured for comprehensive monitoring by checking:

*   **Azure Monitor Agent (AMA)** installation status
*   **SecurityEvent** log ingestion into Sentinel (Windows)
*   **Syslog** log ingestion into Sentinel (Linux)
*   **Last heartbeat and log timestamps** for freshness

By correlating data from **DeviceInfo**, **Heartbeat**, and **SecurityEvent/Syslog** tables, the workbook identifies configuration gaps and supports remediation efforts.

***

## Key Features

*   **Coverage Analysis**
    Detect devices that:
    *   Are onboarded to MDE but missing an AMA heartbeat (potentially missing AMA, or installed but not reporting)
    *   Are not sending SecurityEvent/Syslog logs despite being onboarded

    > **Note:** AMA presence in the first table is determined by the `Heartbeat` table only. See the [Important Notes](#important-notes) section for how to interpret `No AMA or No Heartbeat`.

*   **Filtering Options**
    Filter by:
    *   Workspace
    *   Time range (default: 7 days)
    *   OS platform
    *   AMA status (All, Yes, No) — based on whether an AMA heartbeat was seen in the time window
    *   Exclude Workstations (default: Yes)
    *   Exclude Compliant Machines

*   **Summary Tiles**
    Quick overview of device counts based on AMA status

*   **Detailed Breakdown**
    Categorizes devices as:
    *   **MDE + AMA**
    *   **MDE Only (no AMA heartbeat)**
    *   **AMA Only**

*   **DCR Association**
    Displays Data Collection Rules (DCRs) linked to machines for AMA configuration

*   **Merged View**
    Combines AMA-enabled and/or Defender devices with associated DCRs for full visibility

***

## Important Notes

*   **AMA presence is heartbeat-based in the first table**
    The first table and the executive-summary tiles classify AMA presence using the `Heartbeat` table. A `No` / `No AMA or No Heartbeat` result does **not** prove that the AMA extension is uninstalled — it only means no heartbeat was received in the selected time window. Common causes for a missing heartbeat while the extension is installed:
    - VM is powered off or deallocated
    - Network connectivity to AMA endpoints is blocked
    - AMA service is stopped or misconfigured
    - No Data Collection Rule (DCR) is associated with the machine

    The **merged view** at the bottom of the workbook joins this with Azure Resource Graph (`hasAMAExt`, `amaExtVersion`, `amaExtState`) and is the authoritative source for the actual extension installation state.

*   **Windows and Linux Support**
    This workbook supports both **Windows** and **Linux** endpoints.
    - Windows devices are validated using the **SecurityEvent** table
    - Linux devices are validated using the **Syslog** table

*   **Log Ingestion Check**
    Queries validate security log ingestion into Sentinel using **SecurityEvent** (Windows) and **Syslog** (Linux) tables.

*   **Device Type Filtering**
    By default, workstations and mobile devices are excluded to focus on server infrastructure. This can be toggled via the **Exclude Workstations** filter.

*   **OS Name Limitation**
    Some AMA versions do not report the full OS name (e.g., only `Windows` instead of `Windows Server 2025`).
    This can make filtering by server OS more challenging. Consider using additional metadata or naming conventions for accurate filtering.

***

## How It Works

1.  Collects data from:
    *   **DeviceInfo** (Defender onboarding status)
    *   **Heartbeat** (AMA presence and last seen timestamp)
    *   **SecurityEvent** (Windows security log ingestion)
    *   **Syslog** (Linux security log ingestion)
2.  Joins and correlates AMA presence, Defender onboarding, and log ingestion.
3.  Applies filters for AMA status and OS platform.
4.  Outputs:
    *   Interactive tiles for quick insights
    *   Detailed tables for device-level analysis
    *   Export options for Excel

***

## Use Cases

*   Validate AMA deployment across Defender-managed endpoints
*   Ensure SecurityEvent or Syslog logs are flowing into Sentinel
*   Identify gaps in telemetry for compliance and security posture
*   Correlate AMA coverage with DCR assignments for troubleshooting

***

## Prerequisites

*   Microsoft Sentinel workspace
*   Defender for Endpoint integration enabled
*   AMA deployed on target machines
*   Relevant tables in Log Analytics:
    *   `DeviceInfo`
    *   `Heartbeat`
    *   `SecurityEvent`
    *   `Syslog`

***

## Deployment

1.  Open **Microsoft Sentinel Workbooks**
2.  Click **Add Workbook → Advanced Editor**
3.  Paste the JSON from this repository
4.  Save and customize filters as needed

**Advisory:**
- Default time range is 7 days (adjustable)
- Workstations are excluded by default (toggle with **Exclude Workstations** filter)
- By default, the **Exclude Compliant** filter is set to `MDE + AMA`, which excludes compliant machines so you can focus on remediation. Adjust this filter to include compliant devices if needed.

***
