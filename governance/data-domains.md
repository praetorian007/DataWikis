# Data Domains

**TSG Architecture Office**

> **Author:** Mark Shaw – Principal Data Architect

---

The way we structure and govern our data is critical to how we operate. To manage the scale and complexity of our data landscape, we use a clear and intentional model built around core data domains – broad areas like Customer, Asset, Operations, Finance, Legal and Compliance, and People.

Each domain is responsible for a specific set of conceptual data entities, not just based on who uses the data, but who owns it. For example, while customer teams rely heavily on data from service meters for billing and usage insights, the Asset domain owns the meter itself – its specifications, installation details, and lifecycle history. That distinction matters. It ensures accountability for data quality, clarifies stewardship, and reduces duplication or confusion across systems.

We don't create dozens of domains. We define only as many as we need to reflect real lines of business responsibility. Each domain holds a strategic view over its data, even as that information is used across the corporation to support service delivery, insight, compliance, and customer outcomes.

This article explores how we define our domains, the kinds of data each one manages, and why this structure helps us deliver trusted, reliable data to everyone from field crews and customer service to finance, operations, and compliance.

---

## What Is a Data Domain?

A data domain is a high-level grouping of related data entities that reflects a core area of business responsibility and ownership. Each domain governs the structure, quality, and stewardship of its data, even if that data is used by other parts of the corporation.

---

## What Do Our Domains Look Like?

The Water Corporation's data landscape has six data domains:

| Data Domain | Focus |
|---|---|
| **Customer** | Data related to our customers and their relationships with us (e.g. customer management, service & account management, marketing & public relations, market research). |
| **Asset** | All data about our physical infrastructure and equipment – the water supply, treatment, storage, and distribution assets that enable service delivery. This includes data on network facilities and components (pipes, pumps, treatment plants, meters, etc.) and their attributes. |
| **Operations** | Data about the activities, processes, and monitoring required to deliver services on a daily basis. This includes work management (maintenance and repairs), field operations, service delivery events (e.g. outages, meter readings), and water quality monitoring. |
| **Finance** | All data related to our financial activities, including revenue management and expenditure management (procurement and cost accounting). The Finance team owns these data entities – from tariffs and invoices to budgets and ledgers – ensuring financial integrity and compliance across the corporation. |
| **Legal & Compliance** | Data related to our legal obligations, regulatory compliance, and corporate governance. This includes records of contracts, permits, regulatory reports, and compliance activities that the Legal/Compliance teams own and maintain. |
| **People** | Data about our workforce and organisational structure. It includes information on employees, their roles and qualifications, organisational units, and HR-related records. The People team own these data entities, ensuring that all personnel data – from staffing and skills to organisational hierarchy – is managed as a secure and consistent resource. |
| **Technology & Digital** *(TBD)* | Data about our IT and digital platforms, technology infrastructure, applications, and technology services. It covers the lifecycle of hardware and software assets, user-facing applications, integration services, and digital service support functions. |

---

## Core Data Concepts by Domain

### Customer

#### Core Customer Management

| Concept | Description |
|---|---|
| **Customer** | A recipient of Water Corporation services, representing an individual or organisation that engages in transactions for goods or services (e.g. residential homeowner; commercial business; government agency). |
| **Customer Account** | An account for billing and service, which may group one or more services for a customer (tied to a customer and used for invoicing). |
| **Customer Interaction** | A logged engagement between a customer and the corporation across any channel (phone, email, in-person) e.g. call to call-centre about low pressure; email enquiry on rate changes. |
| **Customer Feedback** | Information provided by customers regarding their satisfaction, complaints, or compliments e.g. online survey response rating service "excellent"; written complaint about a leak. |
| **Customer Consent** | Documented permissions granted by customers for data use, marketing communications, or third-party sharing e.g. consent to share usage data with research partners. |
| **Communication Preference** | The customer's chosen channels and formats for receiving communications (e.g. paper, email, SMS). |
| **Customer Segment** | A group of customers sharing common attributes or behaviours for targeted services e.g. high-usage residential; small-business; concession-eligible pensioners. |

#### Service & Account Management

| Concept | Description |
|---|---|
| **Service Premise (Service Location)** | The physical address or site where water service is delivered and metered. |
| **Service Agreement** | The agreement that defines the service terms for a customer at a premise (e.g. a water service contract linking a customer account to a service point), including rate plans and conditions. |
| **Consumption Record** | A recorded measurement of water usage tied to a specific premise and billing period, used to drive invoicing and analysis. |
| **Service Request** | A customer-initiated request or case (e.g. a new connection application, service issue report, or complaint). |

#### Marketing & Public Relations

| Concept | Description |
|---|---|
| **Marketing Campaign** | A coordinated series of marketing messages sharing a single theme, delivered across multiple channels e.g. "Think Climate Change. Be Waterwise" social-media drive; seasonal conservation ads. |
| **Campaign Audience** | The defined subset of customers targeted by a specific marketing campaign e.g. customers in high-usage suburbs; environmentally conscious households. |
| **Marketing Material** | Collateral and content created to support marketing campaigns e.g. brochures; digital banner ads; educational videos. |
| **Marketing Channel** | The mediums through which marketing messages are delivered e.g. email; Facebook; local newspapers; bill inserts. |
| **Campaign Performance Metric** | Quantitative measures used to assess campaign effectiveness e.g. email open rate; click-through rate; attendance at promotional events. |
| **Media Contact** | Journalists or media outlets with whom the corporation maintains relationships e.g. reporter at The West Australian; ABC Perth newsroom. |
| **Press Release** | An official statement issued to news media announcing corporate news or events e.g. release on new desalination plant opening; service interruption notices. |
| **Public Statement** | Formal communications directed at the public, beyond traditional media releases e.g. website notice on water quality; community notice board updates. |
| **Stakeholder Engagement** | Documented interactions with key stakeholders (e.g. regulators, community groups) e.g. meetings with local councils; workshops with environmental NGOs. |
| **Community Event** | Organised public events aimed at outreach or education e.g. open day at treatment plant; school water-safety workshops. |
| **Public Feedback** | Opinions and comments collected from the public, not limited to existing customers e.g. social-media comments; submissions to public consultation. |

#### Market Research

| Concept | Description |
|---|---|
| **Market Research Study** | A structured investigation to gather insight into customer needs and market conditions e.g. customer satisfaction survey; focus groups on bill payment methods. |
| **Research Participant** | Individuals or organisations taking part in market research activities e.g. customers completing surveys; community members in interviews. |
| **Research Finding** | Insights and conclusions derived from analysing market research data e.g. "75% of customers prefer e-billing"; "High awareness but low adoption of leak-detection tips." |

#### Billing & Tariffs

| Concept | Description |
|---|---|
| **Customer Billing Account** | A record of the financial relationship for a customer, capturing billing cycles and status e.g. monthly billing cycle; active/inactive status. |
| **Billing Address** | The address where billing statements are sent e.g. postal address; email address for e-bills. |
| **Billing Preference** | The customer's chosen format and frequency for receiving bills e.g. paper-based quarterly; electronic monthly. |
| **Billing Contact** | The individual authorised to receive and manage billing communications e.g. property manager; accounts payable officer. |
| **Tariff Assignment** | The mapping of a customer to a specific rate plan based on usage and eligibility e.g. residential standard tariff; commercial peak-time tariff. |
| **Concession Eligibility** | Criteria and records determining a customer's qualification for discounted tariffs e.g. pensioner concession; low-income household discount. |
| **Billing Communication Log** | A history of all billing-related communications sent to a customer e.g. overdue notices; tariff change notifications. |

---

### Asset

#### Core Asset Registry

| Concept | Description |
|---|---|
| **Asset** | An item of physical infrastructure with potential or actual value to the corporation, such as pipes, pumps, or service meters. |
| **Asset Register** | A detailed list of all assets, each assigned a unique identifier and core attributes, serving as the authoritative source for asset data e.g. GIS-linked registry capturing every valve's ID, installation date, and location. |
| **Asset Category** | A classification schema grouping assets by common characteristics (function or physical form) to enable standardised management e.g. "Pipeline," "Hydrant," "Treatment Facility," "Electrical Motor." |
| **Asset Hierarchy** | A parent–child structure showing how assets relate functionally or physically (e.g. facility – pump station – individual pump). |

#### Location & Spatial Context

| Concept | Description |
|---|---|
| **Asset Location** | Geospatial coordinates or descriptive references denoting where an asset is installed within the network e.g. latitude/longitude of a pressure control valve; site address of a reservoir. |
| **Network Segment** | A contiguous section of distribution or transmission infrastructure linking two or more assets, used for analysis and modelling e.g. main line segment from Pump Station B to Reservoir C. |

#### Lifecycle & Condition

| Concept | Description |
|---|---|
| **Asset Lifecycle Stage** | The phase an asset occupies from planning and procurement through installation, operation, maintenance, to disposal. |
| **Condition Assessment** | A periodic evaluation of an asset's physical state to inform maintenance or replacement decisions e.g. inspection report rating pipeline corrosion; structural assessment of a clarifier tank. |
| **Maintenance Plan** | A scheduled strategy outlining preventive and corrective maintenance tasks for each asset e.g. quarterly lubrication of valves; annual overhaul of high-pressure pumps. |

#### Financial & Regulatory Attributes

| Concept | Description |
|---|---|
| **Asset Valuation** | The process of determining the fair market or book value of an asset for financial reporting and decision-making e.g. replacement cost estimate for distribution mains; depreciated book value of a treatment plant. |
| **Asset Depreciation Schedule** | A timetable allocating the cost of an asset over its useful life, tracking annual depreciation and accumulated depreciation e.g. straight-line depreciation of pumps over 20 years; double-declining balance schedule for electronic meters. |
| **Regulatory Compliance Document** | Official permits, licences, or certificates tied to specific assets to demonstrate regulatory adherence e.g. water abstraction licence for a bore well; safety certification for a pressure vessel. |

#### Documentation & Ownership

| Concept | Description |
|---|---|
| **Asset Owner** | The business unit or role accountable for an asset's lifecycle, stewardship, and investment decisions e.g. Asset Management division; Infrastructure Planning team. |
| **Asset Documentation** | Technical manuals, engineering drawings, warranties, and other records supporting asset operation and maintenance e.g. engineering diagrams; equipment warranty certificates. |
| **Asset History Record** | A chronological log of significant events, changes, and interventions affecting an asset over its lifecycle e.g. installation date; major repair in 2022; firmware upgrade on a smart meter. |

#### Extended Asset Concepts

| Concept | Description |
|---|---|
| **Asset Manufacturer & Model** | Information about the make, model, and manufacturer details of an asset to support part replacements and compliance. |
| **Spare Part & Inventory Item** | Components stocked to support asset maintenance and repairs, tracked separately from the primary asset register e.g. seals for valves; spare impellers for pumps. |
| **Asset Warranty** | Terms and expiry dates of the warranty coverage associated with an asset. |
| **Asset Criticality** | A ranking of assets based on their impact on service delivery and risk profile e.g. high-criticality main transmission mains; low-criticality secondary piping. |
| **Asset Specification** | Detailed engineering specifications (e.g. material, size, capacity, performance ratings) defining asset requirements. |
| **Inspection Event** | A formal inspection record for assets, distinct from maintenance tasks, capturing findings and recommendations e.g. CCTV inspection of sewer pipes; SCADA pressure sensor calibration check. |
| **Performance Indicator** | Key metrics tracking asset performance over time, used for reliability and optimisation e.g. mean time between failures (MTBF); pump efficiency ratio. |
| **Asset Material** | Classification of the construction material of an asset for corrosion, lifecycle, and maintenance planning e.g. cast iron; PVC; carbon steel. |

---

### Operations

#### Work Management

| Concept | Description |
|---|---|
| **Work Order** | A formal request and authorisation for work (maintenance, repair, installation) on an asset or service e.g. preventive maintenance on a pump station; corrective repair of a leaking valve. |
| **Work Task** | A discrete unit of work under a work order, specifying steps, resources, and timelines e.g. inspect pipeline segment; replace pump seal; update SCADA configuration. |
| **Job Plan** | A predefined sequence of tasks, labour, materials, and tools required to execute a work order e.g. standard valve lubrication procedure; step-by-step pump overhaul instructions. |

#### Service Delivery

| Concept | Description |
|---|---|
| **Service Order** | An authorisation for non-maintenance operations, such as new connections, disconnections, or meter installations e.g. new service connection for a residential property; service meter exchange request. |
| **Meter Reading** | A recorded measurement of water consumption captured by field personnel or automated systems e.g. monthly reading of a residential service meter; hourly data from a smart meter. |
| **Field Activity** | A record of all field-based operations, including crew dispatch, travel, and onsite activities e.g. crew assignment logs; GPS-tracked route for meter reading. |

#### Water Quality

| Concept | Description |
|---|---|
| **Monitoring Program** | Coordinated water quality and environmental monitoring plans. |
| **Monitoring Location / Sampling Location** | Points where water samples or sensor readings are collected. |
| **Sampling Activity** | Collection events including sample points, field measurements, and custody records. |
| **Analytical Sample & Laboratory Method** | Lab testing of water and environmental parameters. |
| **Result & QC Sample** | Analytical outputs, detection limits, and quality control checks. |
| **Environmental Parameter & Regulatory Limit** | Specific measures (e.g. pH, E. coli) with thresholds mandated by regulators. |
| **Compliance Report & Environmental Incident** | Reporting of monitoring outcomes, incidents, and corrective actions. |
| **Monitoring Schedule & Program Review** | Governance of when/how monitoring occurs and its periodic evaluation. |

#### Incident & Outage Management

| Concept | Description |
|---|---|
| **Operational Incident** | An unplanned event impacting service delivery, such as leaks, breaks, or equipment failures. |
| **Planned Outage** | A scheduled interruption of service for maintenance or upgrades, communicated in advance. |
| **Emergency Response** | Rapid mobilisation and tracking of resources to address critical incidents requiring immediate action. |

#### Resource & Crew Management

| Concept | Description |
|---|---|
| **Crew** | A team of field personnel with assigned skills and certifications to perform operational tasks. |
| **Resource Allocation** | The assignment of equipment, materials, and personnel to work orders or field activities e.g. assigning a backhoe and two technicians to a main break repair. |
| **Qualification & Certification** | Records of required training, licences, and certifications held by field personnel e.g. confined-space entry certification; operator licence for heavy machinery. |

#### Monitoring & Control

| Concept | Description |
|---|---|
| **SCADA Event** | An automated alert or log entry generated by the SCADA system indicating operational anomalies e.g. low-pressure alarm at Zone 3; turbidity spike at Treatment Plant B. |
| **Sensor Reading** | Continuous or periodic measurements from field sensors monitoring parameters like flow, pressure, or quality e.g. real-time flow rate reading; reservoir level data. |
| **Operational Event Log** | A consolidated record of operational events, including SCADA alerts, manual interventions, and system messages e.g. log entry for a pump start/stop; manual override of a valve. |

#### Performance & Reporting

| Concept | Description |
|---|---|
| **Key Performance Indicator (KPI)** | Metrics tracking operational efficiency, reliability, and compliance over time e.g. mean time to repair (MTTR); percentage of on-time maintenance tasks. |
| **Operational Report** | Summarised documents detailing operational metrics, incident summaries, and maintenance outcomes e.g. weekly outage report; monthly maintenance compliance summary. |

#### Service Level Management

| Concept | Description |
|---|---|
| **Service Level Agreement (SLA)** | A formal agreement defining performance targets and commitments between the service provider and its stakeholders e.g. 4-hour response time for critical leaks; 95% on-time completion of preventive maintenance. |

---

### Finance

#### Core Accounting Entities

| Concept | Description |
|---|---|
| **Chart of Accounts** | The hierarchical list of all financial accounts used by the corporation to classify and record transactions, grouping them into assets, liabilities, equity, revenue, and expenses. |
| **General Ledger Account** | A record within the general ledger that aggregates all debit and credit transactions for a specific account, serving as the foundation for financial statements. |
| **Journal Entry** | A double-entry record of a business transaction, specifying one or more debits and credits to General Ledger Accounts, ensuring the accounting equation remains balanced. |
| **Trial Balance** | A report listing the balances of all General Ledger Accounts at a point in time, used to verify that total debits equal total credits before preparing financial statements. |
| **Financial Statement** | Formal reports summarising the corporation's financial performance and position, typically including the Balance Sheet, Income Statement, and Cash Flow Statement. |

#### Revenue & Receivables

| Concept | Description |
|---|---|
| **Invoice** | A document issued to a customer requesting payment for goods or services rendered, containing details such as invoice number, date, and amount due. |
| **Accounts Receivable** | A subsidiary ledger or entity tracking amounts owed by customers for issued invoices, representing short-term assets. |
| **Payment Transaction** | A record of funds received from a customer, applying payments against outstanding invoices in Accounts Receivable. |
| **Credit Note** | A document issued to reduce the amount owed on an invoice, typically for returns, billing errors, or concessions. |

#### Expenditure & Payables

| Concept | Description |
|---|---|
| **Purchase Order** | An authorisation issued to a supplier specifying items, quantities, and agreed prices for the procurement of goods or services. |
| **Supplier (Vendor)** | A party providing goods or services to the Water Corporation, maintained as a master entity in the finance system. |
| **Accounts Payable** | A subsidiary ledger or entity tracking amounts owed to suppliers based on received invoices, representing short-term liabilities. |
| **Payment Disbursement** | A record of funds paid to suppliers, reducing Accounts Payable balances. |
| **Expense Accrual** | A journal entry recognising expenses incurred but not yet invoiced by suppliers, recorded as an accrued liability. |

#### Planning & Control

| Concept | Description |
|---|---|
| **Budget** | A financial plan outlining expected revenues and expenditures over a future period, used for performance measurement and control. |
| **Forecast** | A projection of future financial results (revenues, expenses, cash flow) based on historical data and assumptions. |
| **Cost Centre** | A unit within the corporation (e.g. department or facility) for which costs are tracked separately to control and analyse expenses. |
| **Cost Allocation** | The methodology and records for distributing shared costs (e.g. overhead) across multiple cost centres or activities. |

#### Reporting & Compliance

| Concept | Description |
|---|---|
| **Financial Report** | Formalised outputs (e.g. management reports, regulatory filings) summarising financial performance and position for stakeholders. |
| **Audit Trail** | A sequence of linked records (journal entries, approvals, adjustments) ensuring traceability and transparency of all financial transactions. |

#### Tariff Management

| Concept | Description |
|---|---|
| **Tariff Plan** | A structured pricing scheme that defines how water and related services are charged over time e.g. "Standard Residential Tiered Plan"; "Bulk Industrial Flat Rate." |
| **Tariff Rate** | The specific charge applied per unit of consumption or service as defined in a tariff plan e.g. $2.00 per kilolitre for Tier 1; $3.50/kL for non-residential peak usage. |
| **Tariff Tier** | A consumption band within a tariff plan, each with its own rate e.g. Tier 1: 0–150 kL; Tier 2: 151–500 kL; Tier 3: >500 kL. |
| **Tariff Effective Period** | The date range during which a given tariff plan or rate is valid. |
| **Tariff Change History** | A record of all modifications made to tariff plans or rates over time. |

---

### Legal & Compliance

#### Legal Records

| Concept | Description |
|---|---|
| **Contract / Agreement** | A formal, legally binding document capturing rights and obligations between Water Corporation and another party. |
| **Legal Matter / Case** | A record of any dispute, claim, or litigation involving the Water Corporation, tracking status, filings, and outcomes e.g. lawsuit over alleged water contamination; insurance claim for flood damage. |
| **Legal Opinion** | Formal advice provided by internal or external counsel interpreting laws, regulations, or contractual terms e.g. counsel's opinion on compliance risk under new wastewater discharge rules. |

#### Regulatory Compliance

| Concept | Description |
|---|---|
| **Regulatory Permit / Licence** | Official authorisations granted by regulators permitting specific activities (e.g. water abstraction, discharge) under defined conditions e.g. drinking-water supply licence; wastewater discharge permit. |
| **Compliance Obligation** | A documented requirement imposed by statutes, regulations, or permits that the Water Corporation must satisfy e.g. maximum contaminant levels under the Safe Drinking Water Act; storm-water runoff reporting thresholds. |
| **Regulatory Report** | Official submissions of compliance data to authorities, demonstrating adherence to permit conditions and regulatory standards e.g. monthly water-quality report; annual environmental compliance summary. |

#### Risk & Audit

| Concept | Description |
|---|---|
| **Compliance Audit Record** | Documentation of internal or external audits assessing adherence to laws, standards, and internal policies e.g. ISO 9001 quality-management audit; EPA site inspection report. |
| **Inspection Record** | Records from site inspections (regulatory or self-conducted) verifying asset and process compliance e.g. safety inspection at treatment plant; field inspection of discharge outfall. |
| **Violation / Incident Record** | A record of any non-compliance event or breach, including incident details and corrective measures taken e.g. notice of violation for turbidity exceedance; spill incident and response log. |
| **Risk Register** | A log of identified legal and compliance risks, with assessment of likelihood, impact, and assigned mitigation plans e.g. risk of permit non-renewal; risk of contractual penalty for service interruptions. |

#### Policies & Procedures

| Concept | Description |
|---|---|
| **Compliance Policy** | The corporation's formal statement of compliance commitments, principles, and objectives e.g. water quality compliance policy; data-privacy policy. |
| **Policy / Procedure Document** | Detailed guidelines and standard operating procedures governing legal, regulatory, or ethical activities e.g. contractor management procedure; incident-response procedure. |
| **Compliance Calendar** | A schedule of all compliance deadlines, review dates, and renewal events for regulatory obligations e.g. permit renewal due dates; quarterly internal audit schedule. |

#### Training & Ethics

| Concept | Description |
|---|---|
| **Training & Certification Record** | Records of completion for mandatory compliance training and required certifications by employees or contractors e.g. WHS induction certificate; hazardous-materials handling training. |
| **Ethics & Whistleblower Report** | Documentation of ethics investigations, whistleblower submissions, and resolution outcomes e.g. report of fraudulent billing; outcome of ethics-hotline investigation. |

#### Insurance & Claims

| Concept | Description |
|---|---|
| **Insurance Policy** | Details of insurance coverages maintained by the Water Corporation, including terms, limits, and expiry dates e.g. public-liability insurance; property-damage insurance for treatment plants. |
| **Insurance Claim** | Records of claims lodged under insurance policies, tracking claim details and settlement status e.g. claim for flood damage at reservoir; third-party injury claim. |

#### Monitoring & Nonconformance

| Concept | Description |
|---|---|
| **Compliance Risk Assessment** | A structured evaluation identifying, analysing, and prioritising compliance risks e.g. assessment of risk for non-renewal of a discharge permit; PRIS data-protection risk evaluation. |
| **Compliance Monitoring Event** | Ongoing checks or system-generated alerts tracking adherence to compliance obligations e.g. monthly checklist completion; real-time alert for water-quality threshold breach. |
| **Nonconformance / Corrective Action** | Records of non-compliance findings and associated corrective or preventive measures taken e.g. corrective action following a water-quality violation; process change after safety audit. |
| **Compliance Report** | Internal or external management reports summarising compliance performance, audit findings, and improvement plans e.g. quarterly compliance-health report; annual ISO 19600 conformance summary. |

---

### People

#### Workforce Identity

| Concept | Description |
|---|---|
| **Employee** | A person employed by the Water Corporation under a contract of service, with a corporation-issued identifier e.g. full-time Treatment Plant Operator; part-time Customer Service Representative. |
| **Contractor** | A non-employee individual or organisation engaged under a service contract to perform work for the Water Corporation e.g. external pipeline inspection firm; IT support consultant. |
| **Worker Contact** | Contact information linked to an Employee or Contractor record for official and emergency communications e.g. home address; personal and emergency phone numbers. |
| **Employment Contract** | A legal agreement outlining the terms, conditions, and obligations of employment between the Water Corporation and an Employee e.g. fixed-term contract for a six-month project; permanent staff agreement with termination clauses. |
| **Job Description** | A document detailing the duties, responsibilities, qualifications, and reporting lines for a Position e.g. Senior Engineer role description; Customer Service Agent responsibilities. |

#### Organisational Structure

| Concept | Description |
|---|---|
| **Organisation Unit** | A business division or department within the Water Corporation, arranged in a hierarchical structure e.g. Water Treatment Operations; Finance & Procurement Department. |
| **Position / Job Role** | A specific role within an Org Unit, defining responsibilities, required competencies, and reporting relationships e.g. Technician; Compliance Officer. |
| **Organisation Hierarchy** | The parent–child relationships among Org Units and Positions, illustrating reporting lines and oversight e.g. CEO – COO – Regional Manager – Site Supervisor. |
| **Job Classification** | A grouping of Positions into standard grades or bands for pay scales and career progression e.g. Grade 1 – Entry Level Technician; Grade 5 – Senior Specialist. |
| **Succession Plan** | A plan identifying potential internal candidates for key Positions to ensure leadership continuity e.g. Deputy Plant Manager as successor to Plant Manager; bench strength for CFO role. |

#### Roles & Assignments

| Concept | Description |
|---|---|
| **Employment Assignment** | A record linking an Employee or Contractor to a Position and Org Unit over a specific timeframe e.g. Jane Doe assigned as SCADA Analyst in IT from Jan 2024 to present. |
| **Worker Group / Team** | A collection of Employees and Contractors organised to perform joint tasks or projects e.g. Emergency Response Crew; Meter Reading Squad. |

#### Workforce Development

| Concept | Description |
|---|---|
| **Training Program** | A structured course or module required or offered to Workers to develop skills and comply with regulations e.g. Confined Space Entry Training; Water Quality Certification Workshop. |
| **Certification** | An official credential awarded upon successful completion of Training Programs or professional standards e.g. Certified Water Treatment Operator; First-Aid Certification. |
| **Competency** | A defined skill or capability that a Worker must demonstrate for a Position or assignment e.g. SCADA System Operation; Emergency Response Coordination. |
| **Performance Review** | A formal evaluation of a Worker's performance, goals, and competencies over a period e.g. Annual Performance Appraisal; 6-month probation assessment. |
| **Employee Engagement Survey** | A structured questionnaire assessing workforce morale, satisfaction, and commitment e.g. anonymous Pulse Survey on workplace culture; annual engagement index survey. |

#### Administrative & Compensation

| Concept | Description |
|---|---|
| **Payroll Group** | A grouping of Workers processed together in the payroll system based on pay schedule and classification e.g. monthly salaried employees; weekly casual labourers. |
| **Pay Cycle** | The frequency and schedule for payroll runs within a Payroll Group e.g. last working day of every month; every Friday for hourly workers. |
| **Timesheet** | A detailed record of hours worked by a Worker over a pay period e.g. weekly timesheet logging on-site maintenance hours. |
| **Compensation & Benefits Election** | A record of a Worker's choices regarding benefit plans, allowances, and deferred compensation e.g. selection of health insurance tier; participation in elective deferral plan. |
| **Benefit Plan** | Frameworks of non-salary benefits offered to Workers, managed as part of HR administration e.g. superannuation scheme; wellness program subscription. |

#### Governance & Compliance

| Concept | Description |
|---|---|
| **Worker Health & Safety Record** | Records of health and safety training, incident reports, and medical checks for Workers e.g. safety induction certificate; incident report for on-site injury. |
| **Employee Retention Metric** | A measurement of the corporation's ability to retain Workers over time, typically expressed as a rate e.g. 85% annual retention rate; turnover rate of 10% per quarter. |
| **Organisational Chart** | A visual or data representation of Org Units, Positions, and reporting relationships within the Water Corporation e.g. diagram showing department heads, team leads, and direct reports. |

---

### Technology & Digital *(TBD)*

#### Application & Service Inventory

| Concept | Description |
|---|---|
| **Application** | A packaged software system used to support business functions (e.g. SCADA platform, Customer Portal). |
| **Microservice** | A discrete, independently deployable service component (e.g. Meter-Reading API). |
| **Integration Flow** | A defined data or process integration between systems (e.g. CRM – Billing ETL). |
| **API Endpoint** | A published interface URL or URI through which applications expose functionality or data (e.g. /api/meters/readings). |
| **Data Pipeline** | A managed sequence of data-movement and transformation steps. |

#### Platform & Infrastructure

| Concept | Description |
|---|---|
| **Data Platform** | A hosted environment for data storage and processing (e.g. Enterprise Data Platform, Analytics Sandbox). |
| **Cloud Service** | A managed cloud offering used by the Water Corporation (e.g. Azure Functions, AWS Lambda). |
| **Database Instance** | A running database or data store (e.g. Production PostgreSQL). |
| **Event Stream** | A real-time message bus or streaming channel (e.g. Kafka topic carrying sensor data). |

#### Governance & Metadata

| Concept | Description |
|---|---|
| **Data Catalogue Entry** | A published metadata record describing a data asset or dataset (e.g. "Customer Consumption Data Mart"). |
| **Data Schema** | A formal definition of data structures (tables/fields) used by applications (e.g. CDM schema for meter reads). |
| **Data Access Policy** | A rule set governing who can read or modify specific data assets (e.g. only Data Stewards can update reference tables). |
| **Service Level Agreement** | A formal commitment defining performance/reliability targets for an IT service (e.g. 99.9% uptime for field-mobility app). |
| **Change Request** | A tracked request to modify an application or integration (e.g. add a new field to the billing API schema). |

#### Security & Identity

| Concept | Description |
|---|---|
| **User Account** | A digital identity used to access IT systems (e.g. Active Directory user, service principal). |
| **Access Control Policy** | A set of permissions linking User Accounts or Groups to application roles or data assets. |
| **Audit Log** | A time-stamped record of system actions and accesses (e.g. who changed API Endpoint configurations, when). |

#### Operations & Monitoring

| Concept | Description |
|---|---|
| **System Configuration** | Settings or parameters defining application behaviour (e.g. connection strings, feature toggles). |
| **Deployment Release** | A packaged version of application code or artefacts, with version metadata (e.g. v3.2.1 of the Billing service). |
| **Incident Ticket** | A logged operational issue or outage, tracked through resolution (e.g. API latency incident on SCADA stream). |
| **Monitoring Metric** | A quantitative measure collected for service health (e.g. API response time, data-pipeline lag). |
| **SLA Compliance Report** | A periodic report showing adherence to Service Level Agreement targets. |

---

## How Are the Data Domains Used?

Data domains are not just technical constructs – they reflect real-world ownership. For example, the Customer domain owns data about customer accounts and service agreements, while the Asset domain owns data about meters, pipes, and treatment facilities. Even though multiple teams may access and use the same data, clear ownership ensures that each domain is accountable for the integrity and lifecycle of the data it governs.

This structure supports our data governance framework in several important ways:

**Clarity of Stewardship** – Every conceptual data entity has a clear data owner, typically aligned to the domain's business function. This makes it easier to assign data stewards, resolve issues, and maintain consistency.

**Improved Data Quality** – Because domains are responsible for the data they own, they can put in place quality rules, validation checks, and change control processes tailored to their business needs.

**Cross-Domain Collaboration** – While each domain governs its own data, our architecture allows for safe and reliable data sharing between domains. For example, Operations using asset data or Finance using customer billing information.

**Governance Alignment** – Data governance policies, standards, and processes are implemented within each domain but coordinated at the corporate level. This ensures that governance is both decentralised (where appropriate) and consistent across the corporation.

By using data domains as the organising principle, we create a more scalable, transparent, and accountable approach to corporate data governance – one that supports both operational efficiency and strategic insight.

---

## Considerations for Assigning Data Concepts to Domains

The following considerations help to ensure that each data concept is assigned to the appropriate domain, facilitating effective data governance and management:

**Primary Business Function Alignment** – Assign the data concept to the domain whose core business function is most closely associated with it e.g. data related to customer billing should reside in the Finance domain, as it pertains to financial transactions and revenue management.

**Data Ownership and Stewardship** – Determine which business unit is responsible for the creation, maintenance, and quality of the data e.g. asset specifications and maintenance records are managed by the Asset Management team, placing them in the Asset domain.

**Lifecycle Management Responsibility** – Consider which domain oversees the data throughout its lifecycle, from creation to archival or deletion e.g. employee records, from hiring to termination, are managed by Human Resources, aligning them with the People domain.

**Regulatory and Compliance Considerations** – If the data concept is subject to specific regulatory requirements, assign it to the domain responsible for ensuring compliance e.g. environmental impact data, subject to regulatory reporting, should be under the Legal and Compliance domain.

**Data Usage vs. Data Ownership** – Differentiate between domains that use the data and those that own it. Assign the data concept to the owning domain e.g. operational teams may use customer feedback data, but if the Customer Service team manages and maintains it, it belongs to the Customer domain.

**Shared Data Concepts** – For data concepts used across multiple domains, determine the primary owner based on who has the most significant responsibility for the data e.g. service meter data is used by both Customer and Operations domains, but as Asset Management installs and maintains the meters, the data resides in the Asset domain.

---

## FAQs

### How do Data Domains relate to applications and business units?

Data domains are aligned to the logical ownership of data, not directly to specific applications or business units – though there are important relationships between them.

**Applications** are the tools and systems we use to create, store, process, and consume data. A single application may support multiple business processes and touch data from several domains. For example, our asset management system may handle both Asset data (e.g. equipment details) and Operations data (e.g. work orders). Our customer relationship management (CRM) system might contain both Customer domain data (e.g. contact details) and Service Operations data (e.g. service requests). However, data in an application is governed according to the domain that owns the data, not necessarily the team that uses the system. For instance, meter readings recorded in a field app are used by Customer and Finance teams, but the Operations domain owns the reading event and its integrity.

**Business units** are the organisational structures that carry out work and make decisions. Each domain typically aligns with a specific business unit that acts as the data owner. For example, the Customer Service team owns the Customer domain; the Asset Management group owns the Asset domain; the People & Culture division owns the People domain. Business units are responsible for maintaining the quality and stewardship of their domain's data, including defining rules, handling data issues, and collaborating with other domains when shared data is involved.

**Putting It Together:** Data domains govern the data. Applications manage the data. Business units own the data. This separation of concerns helps us scale our data governance efforts, improve accountability, and ensure that data is always managed by the team with the best understanding of its meaning and importance, regardless of where or how it's used.

### Why fewer domains are better

We've intentionally kept the number of data domains small. Each domain represents a strategic area of data ownership, and fewer domains mean clearer accountability. When too many domains exist, it becomes difficult to assign responsibility, trace data lineage, or manage governance effectively. Our goal is to group data in a way that reflects genuine business stewardship – not just technical convenience – so that every domain has the scale and authority to manage its data across the full lifecycle.

### How domains enable cross-functional collaboration

While data domains define ownership, they don't create silos. Instead, they provide a trusted foundation for collaboration. Each domain owns the quality and meaning of its data, but makes that data available to others through well-governed interfaces – like APIs, shared data views, or curated datasets in our Enterprise Data Platform. This model supports shared analytics, integrated processes, and consistent decision-making across Water Corporation.

### How does this relate to metadata?

Metadata reinforces domain boundaries by clearly labelling data with its owner, description, sensitivity, update frequency, and more. As part of our metadata standards, every key data asset includes domain attribution. This helps avoid confusion about who is responsible for maintaining data and ensures traceability from source to consumption. Metadata is also the foundation for tools like the data catalogue, automated lineage, and impact analysis.

### How domains support strategic initiatives

Our domain model plays a foundational role in major corporate efforts. It supports Enterprise Data Platform architecture (where domain data is ingested, transformed, and curated into trusted layers), regulatory compliance (by assigning ownership to entities responsible for PRIS, audit, or environmental reporting), AI and analytics (by giving data scientists confidence in the meaning and quality of their training data), and operational efficiency (through data services that expose cross-domain information with confidence in its integrity). This domain alignment is not just a modelling exercise – it directly supports corporate strategy.

### How do data domains help us?

As our data environment grows in scale and complexity, the role of domains becomes even more important. They are the backbone of a federated data governance model, where responsibility is distributed but aligned under shared policies and standards. Each domain has embedded data stewards who understand the business meaning, quality expectations, and lifecycle of their data. Meanwhile, central governance ensures consistency, security, and corporate-wide coordination.
