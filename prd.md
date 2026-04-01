---
title: 'Mara Triangle Tourism Report'
repo_name: 'mt-tourism'
workflow_id: 'mt_tourism'
created: '2026-04-01'
status: 'ready-for-dev'
data_sources: [set_er_connection]
reference_workflows: [mt-security, custom-workflows/tourism-report]
tasks_identified: [set_workflow_details, set_time_range, get_timezone_from_time_range, load_df, set_er_connection, get_events, normalize_json_column, drop_column_prefix, filter_row_values, mt_guest_entry, summarize_df, mt_guest_entry_pivot, draw_bar_chart, draw_table, persist_text, persist_df_wrapper, set_string_var, create_docx, create_plot_widget_single_view, create_table_widget_single_view, gather_dashboard]
---

# PRD: Mara Triangle Tourism Report

**Created:** 2026-04-01
**Repo:** `/Users/yunwu/MEP/wt/mt-tourism/`
**Workflow ID:** `mt_tourism`

## Overview

### Problem Statement

Mara Triangle needs a consolidated tourism reporting workflow that fetches guest entry and vehicle entry events from EarthRanger, processes guest categories and gate-level breakdowns, generates bar charts and summary tables, and produces a DOCX report. Currently this requires two separate workflows (`mt_tourism_event_download` + `mt_tourism`) in the legacy custom-workflows system.

### Solution

A single workflow that fetches tourism events from EarthRanger, splits them into guest entry and vehicle entry streams, processes guest data via custom MT tasks, generates 3 bar charts (guest by group, guest by gate, vehicle by gate), a summary table, exports CSV data, assembles a DOCX report, and provides a dashboard with chart and table widgets.

### Scope

**In Scope:**
- Fetch guest_entry and vehicle_entry events from EarthRanger
- Process guest entry data (explode guest_info, map guest groups, pivot by gate)
- Bar chart: guest entry counts by guest group (12 categories)
- Bar chart: paying vs NCG guests by gate
- Bar chart: vehicle entry (paying, non-paying, sticker) by gate
- Guest entry summary table
- CSV data export
- DOCX report with all charts and summary table
- Dashboard with 4 widgets (3 charts + 1 table)

**Out of Scope:**
- Map visualizations (no spatial display needed)
- Revenue calculations
- Historical trend analysis across months

## Data Sources & Connections

| Source | Type | Connection | Notes |
| ----- | ---- | ---------- | ----- |
| EarthRanger | Events (guest_entry, vehicle_entry) | `set_er_connection` / `mara` | Mara Triangle conservancy |

## Reference Workflows

| Workflow | Relevance | Patterns Borrowed |
| -------- | --------- | ----------------- |
| `mt-security` | Primary pattern | ER fetch → process → chart/table widgets → DOCX → dashboard structure |
| `custom-workflows/tourism-report` | Source workflow | Guest entry processing, bar chart configs, DOCX context keys |

## Task Pipeline

### Identified Tasks

| # | Task | ID | Group | Purpose |
|---|------|----|-------|---------|
| 1 | set_workflow_details | workflow_details | — | Dashboard metadata |
| 2 | set_time_range | time_range | — | User time range |
| 3 | get_timezone_from_time_range | get_timezone | — | Extract timezone |
| 4 | load_df (Phase 2) / get_events (Phase 3) | tourism_events | — | Load/fetch tourism events |
| 5 | filter_row_values | filter_guest | Guest Entry | Filter event_type = guest_entry |
| 6 | mt_guest_entry | guest_entry_data | Guest Entry | Explode guest_info, map groups |
| 7 | summarize_df | guest_summary | Guest Entry | Summarize by guest_group |
| 8 | mt_guest_entry_pivot | guest_pivot | Guest Entry | Pivot paying/NCG guests by gate |
| 9 | draw_bar_chart | guest_group_bar | Guest Entry | Guest entry by group chart |
| 10 | persist_text | persist_guest_group_bar | Guest Entry | Save chart HTML |
| 11 | draw_bar_chart | guest_gate_bar | Guest Entry | Guest entry by gate chart |
| 12 | persist_text | persist_guest_gate_bar | Guest Entry | Save chart HTML |
| 13 | filter_row_values | filter_vehicle | Vehicle Entry | Filter event_type = vehicle_entry |
| 14 | draw_bar_chart | vehicle_gate_bar | Vehicle Entry | Vehicle entry by gate chart |
| 15 | persist_text | persist_vehicle_gate_bar | Vehicle Entry | Save chart HTML |
| 16 | set_string_var | set_guest_group_title | Widgets | "Guest Entry by Group" |
| 17 | create_plot_widget_single_view | guest_group_widget | Widgets | Chart widget |
| 18 | set_string_var | set_guest_gate_title | Widgets | "Guest Entry by Gate" |
| 19 | create_plot_widget_single_view | guest_gate_widget | Widgets | Chart widget |
| 20 | set_string_var | set_vehicle_title | Widgets | "Vehicle Entry by Gate" |
| 21 | create_plot_widget_single_view | vehicle_widget | Widgets | Chart widget |
| 22 | set_string_var | set_table_title | Widgets | "Guest Entry Summary" |
| 23 | draw_table | guest_table | Widgets | Render summary table |
| 24 | persist_text | persist_guest_table | Widgets | Save table HTML |
| 25 | create_table_widget_single_view | table_widget | Widgets | Table widget |
| 26 | persist_df_wrapper | persist_guest_csv | — | Export guest summary CSV |
| 27 | create_docx | tourism_report | — | Generate DOCX report |
| 28 | gather_dashboard | dashboard | — | Dashboard with 4 widgets |

### Spec.yaml Draft

```yaml
id: mt_tourism

requirements:
  - name: ecoscope-workflows-core
    version: ">=0.22.18, <0.23.0"
    channel: https://repo.prefix.dev/ecoscope-workflows/
  - name: ecoscope-workflows-ext-ecoscope
    version: ">=0.22.18, <0.23.0"
    channel: https://repo.prefix.dev/ecoscope-workflows/
  - name: ecoscope-workflows-ext-custom
    version: ">=0.0.41, <0.1.0"
    channel: https://repo.prefix.dev/ecoscope-workflows-custom/
  - name: ecoscope-workflows-ext-mt
    version: ">=0.0.2"
    channel: https://repo.prefix.dev/ecoscope-workflows-custom/
  - name: pydeck
    version: "0.0.2"
    channel: https://repo.prefix.dev/ecoscope-workflows-custom/

rjsf-overrides:
  properties:
    # Phase 3 defaults (uncomment when switching to ER connection)
    # get_event_data.properties.event_types.default: ["guest_entry", "vehicle_entry"]
    # get_event_data.properties.event_types.description: >-
    #   Tourism event types to include in the report.
    Guest Entry.properties.guest_summary.properties.summary_params.default:
      - display_name: "Non Resident Adults"
        aggregator: sum
        column: nra
      - display_name: "Non Resident Children"
        aggregator: sum
        column: nrc
      - display_name: "Non Resident Students"
        aggregator: sum
        column: nrs
      - display_name: "Citizen Adults"
        aggregator: sum
        column: ca
      - display_name: "Citizen Children"
        aggregator: sum
        column: cc
      - display_name: "Citizen Students"
        aggregator: sum
        column: cs
      - display_name: "Resident Adults"
        aggregator: sum
        column: ra
      - display_name: "Resident Children"
        aggregator: sum
        column: rc
      - display_name: "Resident Students"
        aggregator: sum
        column: rs
      - display_name: "Narok Adults"
        aggregator: sum
        column: na
      - display_name: "Narok Children"
        aggregator: sum
        column: nc
      - display_name: "Narok Students"
        aggregator: sum
        column: ns
    Guest Entry.properties.guest_group_bar.properties.bar_chart_configs.default:
      - column: "Non Resident Adults"
        agg_func: "sum"
        label: "Non Resident Adults"
      - column: "Non Resident Children"
        agg_func: "sum"
        label: "Non Resident Children"
      - column: "Non Resident Students"
        agg_func: "sum"
        label: "Non Resident Students"
      - column: "Resident Adults"
        agg_func: "sum"
        label: "Resident Adults"
      - column: "Resident Children"
        agg_func: "sum"
        label: "Resident Children"
      - column: "Resident Students"
        agg_func: "sum"
        label: "Resident Students"
      - column: "Citizen Adults"
        agg_func: "sum"
        label: "Citizen Adults"
      - column: "Citizen Children"
        agg_func: "sum"
        label: "Citizen Children"
      - column: "Citizen Students"
        agg_func: "sum"
        label: "Citizen Students"
      - column: "Narok Adults"
        agg_func: "sum"
        label: "Narok Adults"
      - column: "Narok Children"
        agg_func: "sum"
        label: "Narok Children"
      - column: "Narok Students"
        agg_func: "sum"
        label: "Narok Students"
    Guest Entry.properties.guest_gate_bar.properties.bar_chart_configs.default:
      - column: "Paying Guests"
        agg_func: "sum"
        label: "Paying Guests"
        show_label: true
      - column: "Narok County Government(NCG) Guests"
        agg_func: "sum"
        label: "Narok County Government(NCG) Guests"
        show_label: true
    Vehicle Entry.properties.vehicle_gate_bar.properties.bar_chart_configs.default:
      - column: "paying"
        agg_func: "sum"
        label: "Paying Vehicles"
        show_label: true
      - column: "nonpaying"
        agg_func: "sum"
        label: "Non-Paying Vehicles"
        show_label: true
      - column: "sticker"
        agg_func: "sum"
        label: "Annual Sticker"
        show_label: true
    tourism_report.properties.template_path.default: "https://raw.githubusercontent.com/wildlife-dynamics/mt-tourism/main/resources/templates/mt_tourism_report_template.docx"

task-instance-defaults:
  skipif:
    conditions:
      - any_is_empty_df
      - any_dependency_skipped

workflow:
  # Setup
  - name: Workflow Details
    id: workflow_details
    task: set_workflow_details

  - name: Time Range
    id: time_range
    task: set_time_range
    partial:
      time_format: '%d %b %Y %H:%M:%S %Z'

  - name: Extract Timezone
    id: get_timezone
    task: get_timezone_from_time_range
    partial:
      time_range: ${{ workflow.time_range.return }}

  # Data Source — Phase 2: load from local file
  - name: Load Tourism Events
    id: tourism_events
    task: load_df
    partial:
      deserialize_json: true

  # Phase 3 — uncomment to restore ER connection:
  # - name: Data Source
  #   id: er_client_name
  #   task: set_er_connection
  #
  # - name: Get Tourism Events
  #   id: get_event_data
  #   task: get_events
  #   partial:
  #     client: ${{ workflow.er_client_name.return }}
  #     time_range: ${{ workflow.time_range.return }}
  #     event_columns: null
  #     raise_on_empty: false
  #     include_details: true
  #     include_updates: false
  #     include_related_events: false
  #     include_display_values: false
  #     include_null_geometry: true
  #
  # - name: Normalize Event Details
  #   id: normalize_details
  #   task: normalize_json_column
  #   partial:
  #     df: ${{ workflow.get_event_data.return }}
  #     column: "event_details"
  #     skip_if_not_exists: true
  #     sort_columns: false
  #
  # - name: Remove Event Details Prefix
  #   id: tourism_events
  #   task: drop_column_prefix
  #   partial:
  #     df: ${{ workflow.normalize_details.return }}
  #     prefix: "event_details__"
  #     duplicate_strategy: "suffix"

  # Guest Entry Pipeline
  - title: Guest Entry
    type: task-group
    description: "Process guest entry events: categorize, summarize, and chart."
    tasks:
      - name: Filter Guest Entry Events
        id: filter_guest
        task: ecoscope_workflows_ext_custom.tasks.transformation.filter_row_values
        partial:
          df: ${{ workflow.tourism_events.return }}
          column: "event_type"
          values: ["guest_entry"]

      - name: Process Guest Entry
        id: guest_entry_data
        task: ecoscope_workflows_ext_mt.tasks.mt_guest_entry
        partial:
          df: ${{ workflow.filter_guest.return }}

      - name: Summarize Guest Entry
        id: guest_summary
        task: summarize_df
        partial:
          df: ${{ workflow.guest_entry_data.return }}
          groupby_cols: ["guest_group"]
          reset_index: true

      - name: Guest Entry Pivot by Gate
        id: guest_pivot
        task: ecoscope_workflows_ext_mt.tasks.mt_guest_entry_pivot
        partial:
          df: ${{ workflow.guest_entry_data.return }}

      - name: Draw Guest Entry by Group Chart
        id: guest_group_bar
        task: draw_bar_chart
        partial:
          dataframe: ${{ workflow.guest_summary.return }}
          category: "guest_group"
          layout_kwargs: null

      - name: Persist Guest Group Chart
        id: persist_guest_group_bar
        task: persist_text
        partial:
          text: ${{ workflow.guest_group_bar.return }}
          root_path: ${{ env.ECOSCOPE_WORKFLOWS_RESULTS }}
          filename_suffix: "guest_group_bar"

      - name: Draw Guest Entry by Gate Chart
        id: guest_gate_bar
        task: draw_bar_chart
        partial:
          dataframe: ${{ workflow.guest_pivot.return }}
          category: "gate"
          layout_kwargs: null

      - name: Persist Guest Gate Chart
        id: persist_guest_gate_bar
        task: persist_text
        partial:
          text: ${{ workflow.guest_gate_bar.return }}
          root_path: ${{ env.ECOSCOPE_WORKFLOWS_RESULTS }}
          filename_suffix: "guest_gate_bar"

  # Vehicle Entry Pipeline
  - title: Vehicle Entry
    type: task-group
    description: "Process vehicle entry events and generate chart."
    tasks:
      - name: Filter Vehicle Entry Events
        id: filter_vehicle
        task: ecoscope_workflows_ext_custom.tasks.transformation.filter_row_values
        partial:
          df: ${{ workflow.tourism_events.return }}
          column: "event_type"
          values: ["vehicle_entry"]

      - name: Draw Vehicle Entry by Gate Chart
        id: vehicle_gate_bar
        task: draw_bar_chart
        partial:
          dataframe: ${{ workflow.filter_vehicle.return }}
          category: "gate"
          layout_kwargs: null

      - name: Persist Vehicle Gate Chart
        id: persist_vehicle_gate_bar
        task: persist_text
        partial:
          text: ${{ workflow.vehicle_gate_bar.return }}
          root_path: ${{ env.ECOSCOPE_WORKFLOWS_RESULTS }}
          filename_suffix: "vehicle_gate_bar"

  # Dashboard Widgets
  - title: Dashboard Widgets
    type: task-group
    description: "Create chart and table widgets for the dashboard."
    tasks:
      - name: Set Guest Group Chart Title
        id: set_guest_group_title
        task: set_string_var
        partial:
          var: "Guest Entry by Group"

      - name: Create Guest Group Chart Widget
        id: guest_group_widget
        task: create_plot_widget_single_view
        skipif:
          conditions:
            - never
        partial:
          title: ${{ workflow.set_guest_group_title.return }}
          data: ${{ workflow.persist_guest_group_bar.return }}

      - name: Set Guest Gate Chart Title
        id: set_guest_gate_title
        task: set_string_var
        partial:
          var: "Guest Entry by Gate"

      - name: Create Guest Gate Chart Widget
        id: guest_gate_widget
        task: create_plot_widget_single_view
        skipif:
          conditions:
            - never
        partial:
          title: ${{ workflow.set_guest_gate_title.return }}
          data: ${{ workflow.persist_guest_gate_bar.return }}

      - name: Set Vehicle Chart Title
        id: set_vehicle_title
        task: set_string_var
        partial:
          var: "Vehicle Entry by Gate"

      - name: Create Vehicle Chart Widget
        id: vehicle_widget
        task: create_plot_widget_single_view
        skipif:
          conditions:
            - never
        partial:
          title: ${{ workflow.set_vehicle_title.return }}
          data: ${{ workflow.persist_vehicle_gate_bar.return }}

      - name: Set Table Title
        id: set_table_title
        task: set_string_var
        partial:
          var: "Guest Entry Summary"

      - name: Draw Guest Summary Table
        id: guest_table
        task: draw_table
        partial:
          dataframe: ${{ workflow.guest_summary.return }}
          table_config:
            enable_sorting: true
          widget_id: ${{ workflow.set_table_title.return }}

      - name: Persist Guest Table
        id: persist_guest_table
        task: persist_text
        partial:
          text: ${{ workflow.guest_table.return }}
          root_path: ${{ env.ECOSCOPE_WORKFLOWS_RESULTS }}
          filename_suffix: "guest_summary_table"

      - name: Create Table Widget
        id: table_widget
        task: create_table_widget_single_view
        skipif:
          conditions:
            - never
        partial:
          title: ${{ workflow.set_table_title.return }}
          data: ${{ workflow.persist_guest_table.return }}

  # Data Export
  - name: Export Guest Summary
    id: persist_guest_csv
    task: persist_df_wrapper
    partial:
      df: ${{ workflow.guest_summary.return }}
      root_path: ${{ env.ECOSCOPE_WORKFLOWS_RESULTS }}
      filename_prefix: "guest_entry_summary"
      filetypes: ["csv"]
      sanitize: false
    skipif:
      conditions:
        - never

  # DOCX Report
  - name: Create Tourism Report
    id: tourism_report
    task: ecoscope_workflows_ext_custom.tasks.results.create_docx
    partial:
      groupers: null
      context:
        items:
          - item_type: timerange
            key: report_date
            value: ${{ workflow.time_range.return }}
            format: "%b %Y"
          - item_type: image
            key: guest_entry_bar_chart
            value: ${{ workflow.persist_guest_group_bar.return }}
            screenshot_config:
              wait_for_timeout: 0
              max_concurrent_pages: 2
          - item_type: image
            key: guest_entry_bar_gate
            value: ${{ workflow.persist_guest_gate_bar.return }}
            screenshot_config:
              wait_for_timeout: 0
              max_concurrent_pages: 2
          - item_type: image
            key: vehicle_entry_bar
            value: ${{ workflow.persist_vehicle_gate_bar.return }}
            screenshot_config:
              wait_for_timeout: 0
              max_concurrent_pages: 2
          - item_type: table
            key: data
            value: ${{ workflow.guest_summary.return }}
      output_dir: ${{ env.ECOSCOPE_WORKFLOWS_RESULTS }}
      filename_prefix: mt_tourism_report
    skipif:
      conditions:
        - never

  # Dashboard
  - name: Create Dashboard
    id: dashboard
    task: gather_dashboard
    partial:
      details: ${{ workflow.workflow_details.return }}
      widgets:
        - ${{ workflow.guest_group_widget.return }}
        - ${{ workflow.guest_gate_widget.return }}
        - ${{ workflow.vehicle_widget.return }}
        - ${{ workflow.table_widget.return }}
      time_range: ${{ workflow.time_range.return }}
```

### Task Gaps

None — all tasks exist in core/ecoscope/custom/mt libraries. Uses fully-qualified task names for:
- `ecoscope_workflows_ext_custom.tasks.transformation.filter_row_values`
- `ecoscope_workflows_ext_mt.tasks.mt_guest_entry`
- `ecoscope_workflows_ext_mt.tasks.mt_guest_entry_pivot`
- `ecoscope_workflows_ext_custom.tasks.results.create_docx`

## Development Strategy

### Data Source Approach

3-phase approach — EarthRanger is a remote API. Start with Phase 2 (local data).

### Phase 1 — Bootstrap Data

**Download workflow:** Use existing `mt_tourism_event_download` desktop workflow which downloads guest_entry + vehicle_entry events from EarthRanger (`mara` connection) to a local parquet file.

| Data | Source Task | Output Format | Output Path |
| ---- | ----------- | ------------- | ----------- |
| Tourism events (guest + vehicle) | get_events via wt-download-events | GeoParquet | `resources/mock-data/mt_guest_event.parquet` |

Copy existing download output from:
`/Users/yunwu/.ecoscope-desktop/workflows/mt_tourism_event_download/outputs/mt_guest_event.parquet`

### Phase 2 — Develop with Local Data

**load_df stand-in configuration:**

```yaml
- name: Load Tourism Events
  id: tourism_events
  task: load_df
  partial:
    deserialize_json: true
```

The downloaded parquet (from wt-download-events) has event_details already normalized and prefix-dropped, so `guest_info`, `gate`, `paying`, `nonpaying`, `sticker` are top-level columns. No additional normalize/drop_prefix steps needed in Phase 2.

**What to complete in Phase 2:**
- [ ] Guest entry processing pipeline (filter → mt_guest_entry → summarize → pivot)
- [ ] Vehicle entry filter + bar chart
- [ ] All 3 bar charts rendering correctly
- [ ] Guest summary table widget
- [ ] CSV export
- [ ] DOCX report renders with all charts and table
- [ ] rjsf form configured and validated
- [ ] Dashboard layout finalized
- [ ] Base test case passing (`mock_io: true`)

### Phase 3 — Reconnect Live Data Source

Replace `load_df` with ER connection tasks. Add normalize + drop prefix steps since raw ER data has `event_details` as a dict column.

**Final data source configuration:**

```yaml
- name: Data Source
  id: er_client_name
  task: set_er_connection

- name: Get Tourism Events
  id: get_event_data
  task: get_events
  partial:
    client: ${{ workflow.er_client_name.return }}
    time_range: ${{ workflow.time_range.return }}
    event_columns: null
    raise_on_empty: false
    include_details: true
    include_updates: false
    include_related_events: false
    include_display_values: false
    include_null_geometry: true

- name: Normalize Event Details
  id: normalize_details
  task: normalize_json_column
  partial:
    df: ${{ workflow.get_event_data.return }}
    column: "event_details"
    skip_if_not_exists: true
    sort_columns: false

- name: Remove Event Details Prefix
  id: tourism_events
  task: drop_column_prefix
  partial:
    df: ${{ workflow.normalize_details.return }}
    prefix: "event_details__"
    duplicate_strategy: "suffix"
```

**Test case updates:**
- Update `base` case: `mock_io: true` (mocked API)
- Add `integration` case: `mock_io: false` with `mara` connection, Jan 2026 data
- Verify data shape from live source matches Phase 2 development data

## Output Configuration

### Charts

| Chart | Type | Category | Bar Configs | Notes |
| ----- | ---- | -------- | ----------- | ----- |
| Guest Entry by Group | `draw_bar_chart` | guest_group | 12 bars: NRA, NRC, NRS, CA, CC, CS, RA, RC, RS, NA, NC, NS (all sum) | From summarize_df output |
| Guest Entry by Gate | `draw_bar_chart` | gate | Paying Guests (sum), NCG Guests (sum) | From mt_guest_entry_pivot output |
| Vehicle Entry by Gate | `draw_bar_chart` | gate | Paying (sum), Non-Paying (sum), Annual Sticker (sum) | From filtered vehicle_entry data |

### Data Exports

| Export | Format | Sanitize | Notes |
| ------ | ------ | -------- | ----- |
| Guest entry summary | CSV | false | Summary data is clean (no nested JSON) |

### DOCX Report

| Item | Type | Context Key | Source | Notes |
| ---- | ---- | ----------- | ------ | ----- |
| Report date | timerange | report_date | time_range | Format: "%b %Y" |
| Guest by group chart | image | guest_entry_bar_chart | persist_guest_group_bar | Screenshot, no wait |
| Guest by gate chart | image | guest_entry_bar_gate | persist_guest_gate_bar | Screenshot, no wait |
| Vehicle by gate chart | image | vehicle_entry_bar | persist_vehicle_gate_bar | Screenshot, no wait |
| Guest summary table | table | data | guest_summary | DataFrame rendered as table |

Context keys match the existing DOCX template at:
`/Users/yunwu/MEP/custom-workflows/mara-triangle/data/tourism_template.docx`

### Widget & Dashboard Assembly

| Widget | Type | Title | Content Source |
| ------ | ---- | ----- | -------------- |
| 0 | create_plot_widget_single_view | Guest Entry by Group | persist_guest_group_bar |
| 1 | create_plot_widget_single_view | Guest Entry by Gate | persist_guest_gate_bar |
| 2 | create_plot_widget_single_view | Vehicle Entry by Gate | persist_vehicle_gate_bar |
| 3 | create_table_widget_single_view | Guest Entry Summary | persist_guest_table |

**gather_dashboard:**
```yaml
details: ${{ workflow.workflow_details.return }}
widgets:
  - ${{ workflow.guest_group_widget.return }}
  - ${{ workflow.guest_gate_widget.return }}
  - ${{ workflow.vehicle_widget.return }}
  - ${{ workflow.table_widget.return }}
time_range: ${{ workflow.time_range.return }}
```

## Form Configuration (rjsf)

### Parameter Visibility

| Parameter | Basic / Advanced | Default | Notes |
| --------- | ---------------- | ------- | ----- |
| workflow_details | Basic | — | Name and description |
| time_range | Basic | — | Since/until |
| tourism_events.file_path (Phase 2) | Basic | — | Path to parquet |
| er_client_name (Phase 3) | Basic | — | ER connection |
| guest_summary.summary_params | Advanced | 12 guest category params | Set via rjsf-overrides |
| guest_group_bar.bar_chart_configs | Advanced | 12 category bars | Set via rjsf-overrides |
| guest_gate_bar.bar_chart_configs | Advanced | Paying + NCG bars | Set via rjsf-overrides |
| vehicle_gate_bar.bar_chart_configs | Advanced | Paying + Non-Paying + Sticker | Set via rjsf-overrides |
| tourism_report.template_path | Advanced | GitHub-hosted template | Set via rjsf-overrides |

### rjsf-overrides Draft

See spec.yaml draft above for complete rjsf-overrides section.

### Validation Checklist

- [x] All bar chart configs default to matching the existing tourism workflow
- [x] Summary params default to 12 guest categories
- [x] DOCX template URL defaults to GitHub-hosted template
- [x] ui:order NOT used

## Test Strategy

### Test Cases

| Case Name | mock_io | Purpose |
| --------- | ------- | ------- |
| base | true | Validates full pipeline with mocked data |
| integration (Phase 3) | false | Validates real ER connectivity with `mara` connection |

### test-cases.yaml Draft

```yaml
base:
  name: Base Test Case
  mock_io: true
  params:
    workflow_details:
      name: "MT TOURISM"
    time_range:
      since: "2026-01-01T00:00:00"
      until: "2026-01-31T23:59:59"
      timezone:
        label: "Africa/Nairobi (UTC+03:00)"
        tzCode: "Africa/Nairobi"
        name: "(UTC+03:00) Nairobi"
        utc: "+03:00"
    tourism_events:
      file_path: "resources/mock-data/mt_guest_event.parquet"
    guest_summary:
      summary_params:
        - display_name: "Non Resident Adults"
          aggregator: sum
          column: nra
        - display_name: "Non Resident Children"
          aggregator: sum
          column: nrc
        - display_name: "Non Resident Students"
          aggregator: sum
          column: nrs
        - display_name: "Citizen Adults"
          aggregator: sum
          column: ca
        - display_name: "Citizen Children"
          aggregator: sum
          column: cc
        - display_name: "Citizen Students"
          aggregator: sum
          column: cs
        - display_name: "Resident Adults"
          aggregator: sum
          column: ra
        - display_name: "Resident Children"
          aggregator: sum
          column: rc
        - display_name: "Resident Students"
          aggregator: sum
          column: rs
        - display_name: "Narok Adults"
          aggregator: sum
          column: na
        - display_name: "Narok Children"
          aggregator: sum
          column: nc
        - display_name: "Narok Students"
          aggregator: sum
          column: ns
    guest_group_bar:
      bar_chart_configs:
        - column: "Non Resident Adults"
          agg_func: "sum"
          label: "Non Resident Adults"
        - column: "Non Resident Children"
          agg_func: "sum"
          label: "Non Resident Children"
        - column: "Non Resident Students"
          agg_func: "sum"
          label: "Non Resident Students"
        - column: "Resident Adults"
          agg_func: "sum"
          label: "Resident Adults"
        - column: "Resident Children"
          agg_func: "sum"
          label: "Resident Children"
        - column: "Resident Students"
          agg_func: "sum"
          label: "Resident Students"
        - column: "Citizen Adults"
          agg_func: "sum"
          label: "Citizen Adults"
        - column: "Citizen Children"
          agg_func: "sum"
          label: "Citizen Children"
        - column: "Citizen Students"
          agg_func: "sum"
          label: "Citizen Students"
        - column: "Narok Adults"
          agg_func: "sum"
          label: "Narok Adults"
        - column: "Narok Children"
          agg_func: "sum"
          label: "Narok Children"
        - column: "Narok Students"
          agg_func: "sum"
          label: "Narok Students"
    guest_gate_bar:
      bar_chart_configs:
        - column: "Paying Guests"
          agg_func: "sum"
          label: "Paying Guests"
          show_label: true
        - column: "Narok County Government(NCG) Guests"
          agg_func: "sum"
          label: "Narok County Government(NCG) Guests"
          show_label: true
    vehicle_gate_bar:
      bar_chart_configs:
        - column: "paying"
          agg_func: "sum"
          label: "Paying Vehicles"
          show_label: true
        - column: "nonpaying"
          agg_func: "sum"
          label: "Non-Paying Vehicles"
          show_label: true
        - column: "sticker"
          agg_func: "sum"
          label: "Annual Sticker"
          show_label: true
    tourism_report:
      template_path: "/Users/yunwu/MEP/custom-workflows/mara-triangle/data/tourism_template.docx"
```

### Data Source Testing

| Data Source | mock_io: true behavior | mock_io: false requirements |
| ----------- | ---------------------- | --------------------------- |
| load_df (Phase 2) | Uses mock data | Reads from resources/mock-data/ |
| EarthRanger (Phase 3) | Mock API responses | `mara` connection, Jan 2026 data |

### skipif Conditions

| Task | Conditions | Rationale |
| ---- | ---------- | --------- |
| (all tasks) | any_is_empty_df, any_dependency_skipped | Global default |
| All widget tasks | never | Always create placeholder widgets |
| persist_guest_csv | never | Always export even if empty |
| tourism_report | never | Always generate report |

### Validation Approach

- Base test: verify pipeline processes mock tourism data end-to-end
- Verify guest_entry filtering produces only guest_entry rows
- Verify mt_guest_entry produces expected guest_group categories
- Verify mt_guest_entry_pivot produces gate × guest_group pivot
- Verify 3 bar charts render as HTML
- Verify guest summary table has 12 guest category columns
- Verify DOCX report renders with all 3 charts and 1 table
- Verify dashboard has 4 widgets

## Dashboard Layout

### layout.json Draft

```json
[
    {
        "i": "0",
        "x": 0, "y": 0,
        "w": 10, "h": 10,
        "minW": 4, "minH": 8,
        "widget_id": 0,
        "static": false
    },
    {
        "i": "1",
        "x": 0, "y": 10,
        "w": 5, "h": 10,
        "minW": 4, "minH": 8,
        "widget_id": 1,
        "static": false
    },
    {
        "i": "2",
        "x": 5, "y": 10,
        "w": 5, "h": 10,
        "minW": 4, "minH": 8,
        "widget_id": 2,
        "static": false
    },
    {
        "i": "3",
        "x": 0, "y": 20,
        "w": 10, "h": 10,
        "minW": 4, "minH": 8,
        "widget_id": 3,
        "static": false
    }
]
```

- Widget 0: Guest Entry by Group chart (full width)
- Widget 1: Guest Entry by Gate chart (left half)
- Widget 2: Vehicle Entry by Gate chart (right half)
- Widget 3: Guest Entry Summary table (full width)

## Implementation Plan

### Acceptance Criteria

- [ ] AC 1: Given tourism event data, when the workflow runs, then guest entry by group bar chart shows 12 guest categories summed by guest_group
- [ ] AC 2: Given guest entry data, when pivoted by gate, then guest entry by gate chart shows paying vs NCG guests per gate
- [ ] AC 3: Given vehicle entry data, when filtered and charted, then vehicle entry by gate chart shows paying, non-paying, and sticker counts per gate
- [ ] AC 4: Given guest summary data, when rendered as a table widget, then the dashboard table shows all 12 guest category columns
- [ ] AC 5: Given all outputs, when DOCX report is generated, then report contains all 3 charts and guest summary table
- [ ] AC 6: Given no tourism data for the time range, when the workflow runs, then skipif conditions prevent errors and report/exports are still generated
- [ ] AC 7: Given the rjsf form, when rendered, then summary params and bar chart configs have sensible defaults

## Additional Context

### Dependencies

- ecoscope-workflows-core >= 0.22.18
- ecoscope-workflows-ext-ecoscope >= 0.22.18
- ecoscope-workflows-ext-custom >= 0.0.41
- ecoscope-workflows-ext-mt >= 0.0.2
- pydeck 0.0.2
- DOCX template: existing at `/Users/yunwu/MEP/custom-workflows/mara-triangle/data/tourism_template.docx`

### Notes

- MT custom tasks (`mt_guest_entry`, `mt_guest_entry_pivot`) use fully-qualified names
- `filter_row_values` uses fully-qualified name (custom library)
- `create_docx` uses fully-qualified name (custom library)
- Context keys in create_docx (`guest_entry_bar_chart`, `guest_entry_bar_gate`, `vehicle_entry_bar`, `data`) match the existing DOCX template placeholders
- No map output — this workflow is chart/table/report focused
- Phase 2 uses pre-processed data (event_details already normalized by download workflow); Phase 3 adds normalize + drop prefix steps
- Guest group mapping is handled by `mt_guest_entry` task (not in spec)
