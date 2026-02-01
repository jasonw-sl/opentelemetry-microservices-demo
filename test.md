```mermaid
graph TD

subgraph CustomerEnv["Customer Environment"]
  A["Ticket Tree Builder - Jira or ADO query"] -->|ticket_info.json| B["Coverage Report Generator - US_RPT_create_report_info_files.py"]

  B -->|Create report| SL1["SeaLights REST API - Create coverage report"]
  SL1 -->|report_id| B

  B -->|Fetch per ticket subtree| SL2["SeaLights REST API - Get coverage results"]
  SL2 -->|coverage data| B

  B -->|ReportInfo_<TICKET>.json| C["Intermediate Coverage Model - ReportInfo_*.json"]

  C --> D1["HTML Reports - US2_RPT_create_html_reports.py"]
  C --> D2["Confluence Publisher - US3_RPT_create_confluence_reports.py"]
  C --> D3["Jira Plugin Update - US4_RPT_update_jira_plugin.py"]
  C --> D4["Jira Field Update - US4_RPT_update_jira.py"]
  C --> D5["ADO Coverage Extension - US4_RPT_update_ado_plugin.py"]
end

```
