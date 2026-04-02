# Mara Triangle Tourism Report Workflow

## Introduction

This workflow generates a monthly tourism report for the Mara Triangle conservancy. It fetches guest entry and vehicle entry events from EarthRanger, processes guest categories and gate-level breakdowns, and produces charts, summary tables, a CSV export, and a DOCX report.

**What this workflow does:**
- Fetches **guest entry** and **vehicle entry** events from EarthRanger
- Categorizes guests into 12 groups (Non-Resident, Citizen, Resident, and Narok Adults/Children/Students)
- Creates a pivot breakdown of paying vs. NCG guests by gate
- Generates 3 bar charts and a summary table
- Exports guest summary data as CSV
- Produces a formatted DOCX report with all charts and tables

**Who should use this:**
- Conservation managers preparing monthly tourism reports for Mara Triangle
- Finance or operations teams tracking visitor counts by category and gate
- Anyone needing a consolidated view of tourism activity across the conservancy

## Prerequisites

Before using this workflow, you need:

1. **Ecoscope Desktop** installed on your computer
   - If you haven't installed it yet, please follow the installation instructions for Ecoscope Desktop

2. **EarthRanger Data Source** configured in Ecoscope Desktop
   - You must have already set up a connection to your EarthRanger server
   - Your data source should be configured with proper authentication credentials
   - You'll need to know the name of your configured data source (e.g., `"mara"`)

3. **Tourism Event Types** configured in EarthRanger
   - Your EarthRanger system must have `guest_entry` and `vehicle_entry` event types
   - Events should include guest details (guest group, gate, visitor counts by category)
   - You can verify event types at `https://<your-site>.pamdas.org/admin/activity/eventtype/`

## Installation

1. Select "Workflow Templates" tab
2. Click "+ Add Template"
3. Copy and paste this URL `https://github.com/wildlife-dynamics/mt-tourism` and wait for the workflow template to be downloaded and initialized
4. The template will now appear in your available template list

## Configuration Guide

### Basic Configuration

#### 1. Workflow Details
Give your workflow run a name and optional description.

- **Name** (required): A descriptive name for this run
  - Example: `"MT TOURISM"`
- **Description** (optional): Additional context about this run
  - Example: `"January 2026 monthly tourism report"`

#### 2. Time Range
Set the date range for the tourism data you want to analyze.

- **Since** (required): Start date and time
  - Example: `2026-01-01T00:00:00`
- **Until** (required): End date and time
  - Example: `2026-01-31T23:59:59`
- **Timezone** (required): Your local timezone
  - Example: `Africa/Nairobi (UTC+03:00)`
  - Note: Choose the timezone that matches your reporting region

#### 3. Data Source
Select the EarthRanger connection for fetching tourism events.

- **Data Source** (required): The name of your configured EarthRanger connection
  - Example: `"mara"`
  - Note: This must match the data source name configured in Ecoscope Desktop

#### 4. Create Tourism Report
Configure the DOCX report template.

- **Template Path** (required): URL or path to the Word document template
  - Default: `"https://raw.githubusercontent.com/wildlife-dynamics/mt-tourism/main/resources/templates/mt_tourism_report_template.docx"`
  - Note: The default template is provided with this workflow. Only change this if you have a custom template.

## Running the Workflow

Once you've configured all the settings:

1. **Review your configuration**
   - Double-check your time range and data source name
   - Ensure the time range covers the reporting period you need (typically one calendar month)

2. **Save and run**
   - Click the "Submit" button and the workflow will show up in the "My Workflows" table
   - Click on "Run" and the workflow will begin processing

3. **Monitor progress and wait for completion**
   - You'll see status updates as the workflow runs
   - Processing time depends on:
     - The size of your date range
     - The number of guest entry and vehicle entry events in the system
   - The workflow completes with status "Success" or "Failed"

## Understanding Your Results

After the workflow completes successfully, you'll find your outputs in the designated output folder.

### Data Outputs

#### Guest Entry Summary (CSV)
- **File format**: CSV
- **Opens in**: Microsoft Excel, Google Sheets
- **Contents**: Guest counts aggregated by guest group, with columns for each visitor category:
  - `guest_group`: The guest group name (e.g., Paying Guests, Angama, Serena Tickets)
  - `Non Resident Adults`, `Non Resident Children`, `Non Resident Students`
  - `Citizen Adults`, `Citizen Children`, `Citizen Students`
  - `Resident Adults`, `Resident Children`, `Resident Students`
  - `Narok Adults`, `Narok Children`, `Narok Students`

#### DOCX Report
- **File format**: Word document (.docx)
- **Opens in**: Microsoft Word, Google Docs
- **Contents**: A formatted monthly report containing:
  - Report date header
  - Guest Entry by Group bar chart
  - Guest Entry by Gate bar chart
  - Vehicle Entry by Gate bar chart
  - Guest summary table

### Visual Outputs (Dashboard)

The workflow creates an interactive dashboard with 4 widgets:

#### Guest Entry by Group
- **Format**: Interactive bar chart
- **Features**:
  - X-axis: Guest group (e.g., Paying Guests, Angama, Serena Tickets)
  - Y-axis: Visitor counts broken down by 12 categories (Non-Resident/Citizen/Resident/Narok Adults/Children/Students)
  - Interactive hover: Shows exact counts when you mouse over bars

#### Guest Entry by Gate
- **Format**: Interactive bar chart
- **Features**:
  - X-axis: Gate name
  - Y-axis: Total guests for Paying Guests and Narok County Government (NCG) Guests
  - Compares paying vs. NCG guest volumes at each entry gate

#### Vehicle Entry by Gate
- **Format**: Interactive bar chart
- **Features**:
  - X-axis: Gate name
  - Y-axis: Vehicle counts (Paying Vehicles, Non-Paying Vehicles, Annual Sticker)
  - Shows vehicle entry distribution across gates

#### Guest Entry Summary Table
- **Format**: Interactive sortable table
- **Features**:
  - Rows grouped by guest group
  - Columns for each of the 12 visitor categories
  - Click column headers to sort

## Common Use Cases & Examples

### Example 1: Standard Monthly Report
**Goal**: Generate the January 2026 tourism report for Mara Triangle

**Configuration**:
- **Time Range**:
  - Since: `2026-01-01T00:00:00`
  - Until: `2026-01-31T23:59:59`
  - Timezone: `Africa/Nairobi (UTC+03:00)`
- **Data Source**: `"mara"`
- **Template Path**: Use default

**Result**:
- Bar charts showing guest and vehicle counts for January
- CSV export with detailed guest breakdown by group
- DOCX report ready for distribution to stakeholders

---

### Example 2: Quarterly Review
**Goal**: Generate a tourism summary for Q1 2026 (January-March)

**Configuration**:
- **Time Range**:
  - Since: `2026-01-01T00:00:00`
  - Until: `2026-03-31T23:59:59`
  - Timezone: `Africa/Nairobi (UTC+03:00)`
- **Data Source**: `"mara"`
- **Template Path**: Use default

**Result**:
- Aggregated guest and vehicle counts across the full quarter
- Useful for identifying seasonal trends in tourism activity

---

### Example 3: Custom Report Template
**Goal**: Use a customized DOCX template with additional branding or sections

**Configuration**:
- **Time Range**: Set to your desired reporting period
- **Data Source**: `"mara"`
- **Template Path**: Path or URL to your custom `.docx` template
  - Example: `"/path/to/custom_tourism_template.docx"`

**Result**:
- Report generated using your custom template layout
- Note: Your template must contain the same Jinja2 placeholders as the default template

## Troubleshooting

### Common Issues and Solutions

#### No data returned for the time range
**Problem**: The workflow completes but charts and tables are empty.

**Solutions**:
- Verify that `guest_entry` and `vehicle_entry` events exist in EarthRanger for your selected date range
- Check the event types at `https://<your-site>.pamdas.org/admin/activity/eventtype/`
- Try a broader date range to confirm data exists

#### Authentication error
**Problem**: The workflow fails with a connection or authentication error.

**Solutions**:
- Verify your data source name matches exactly (e.g., `"mara"`)
- Re-enter your EarthRanger credentials in Ecoscope Desktop
- Ensure your EarthRanger account has permission to view events

#### DOCX report fails to generate
**Problem**: The workflow runs but the DOCX report is not created.

**Solutions**:
- Verify the template path is accessible (URL must be reachable, or local path must exist)
- If using a custom template, ensure it contains the required Jinja2 placeholders: `report_date`, `guest_entry_bar_chart`, `guest_entry_bar_gate`, `vehicle_entry_bar`, `data`
- Check that Playwright (used for chart screenshots) is installed correctly

#### Charts show unexpected categories
**Problem**: Bar charts display unknown or missing guest groups.

**Solutions**:
- The guest group mapping is built into the workflow and handles standard Mara Triangle guest categories
- If new guest groups have been added to EarthRanger, they may appear with their raw code names
- Contact your system administrator to update the guest group mapping if needed

#### Workflow runs slowly
**Problem**: The workflow takes a long time to complete.

**Solutions**:
- Reduce the date range to a shorter period
- The first run may be slower as the system initializes; subsequent runs are typically faster
- Large numbers of tourism events (thousands per month) may increase processing time

#### Missing guest categories in summary
**Problem**: Some visitor categories show zero or are missing from the summary table.

**Solutions**:
- Verify that the guest entry events in EarthRanger include the expected fields (`nra`, `nrc`, `nrs`, `ca`, `cc`, `cs`, `ra`, `rc`, `rs`, `na`, `nc`, `ns`)
- Check that the `guest_info` field in event details is populated for guest entry events
- Zero values are expected if no visitors of that category were recorded in the time range
