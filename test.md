### Diagram Description (Visual Layout)

**Boundary: Customer Environment**

Inside the customer environment, the flow is strictly left‑to‑right:

1. **Ticket Tree Builder (Jira / ADO)**

   * Queries Jira or Azure DevOps using customer credentials
   * Builds a hierarchical ticket tree (stories, children, subtasks)
   * Produces a structured `ticket_info.json`

2. **Coverage Report Generation (US1 – Create Report Info)**

   * Consumes `ticket_info.json` and customer settings
   * Interacts with SeaLights via outbound REST APIs only:

     * **Create coverage report** (single report for all tickets)
     * **Fetch coverage results per top‑level ticket subtree**
   * Produces:

     * `ReportInfo_<TICKET>.json` (stable intermediate model)
     * Per‑ticket raw coverage result files

3. **Intermediate Contract: ReportInfo Files**

   * `ReportInfo_*.json` acts as the formal boundary between data generation and publishing
   * All downstream consumers rely on this format

4. **Publishers / Outputs** (parallel fan‑out):

   * **HTML Reports** (US2)
   * **Confluence Pages** (US3)
   * **Jira Updates**:

     * Plugin‑backed coverage panel (US4 – Jira plugin)
     * Native Jira custom field updates (US4 – Jira fields)
   * **Azure DevOps Updates**:

     * SeaLights Coverage Extension data (US4 – ADO plugin)

**External Systems (no inbound access):**

* **SeaLights**: accessed only via outbound REST API calls from the customer environment
* **Jira / ADO**: accessed via customer‑managed credentials and network rules

### Key Architectural Properties

* All scripts execute **entirely within the customer environment**
* SeaLights never initiates connections into customer systems
* Jira / ADO are both **inputs** (ticket metadata) and **outputs** (coverage publishing)
* `ReportInfo_*.json` is the stable, auditable hand‑off between stages
* US1 is the only component responsible for SeaLights report orchestration
  )**
* `ReportInfo_*.json` is the stable intermediate contract between generation and publishing
