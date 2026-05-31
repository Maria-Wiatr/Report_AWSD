# Aid Worker Security Incidents BI Report

This interactive Power BI report presents major security incidents affecting humanitarian personnel worldwide, using data from the [Aid Worker Security Database (AWSD)](https://www.aidworkersecurity.org/). It supports self-directed exploration of nearly three decades of AWSD records.


### 🔍 [View the live report](https://maria-wiatr.github.io/Report_AWSD_em/)

---

## Report Structure

The report is organised into five pages, each designed to support a specific stage of data exploration:

| Page | Description |
|---|---|
| **Home** | Introduction, data background, key definitions, and a long-term incident trend chart |
| **Global Overview** | Temporal trends, geographic distribution, top affected countries, year-on-year comparison |
| **Attacks & Perpetrators** | Attack means, context, settings, perpetrator types, and motives — with a cross-country comparison matrix |
| **Victims** | National vs. international staff, gender distribution, organisation type, and outcome breakdown |
| **Data Exploration** | Open-ended decomposition tree for user-driven breakdown across any combination of dimensions |

---

## Data Architecture

The data warehouse architecture runs entirely within **Microsoft Fabric**. The solution is organized as a master pipeline that orchestrates four sub-pipelines, each responsible for a specific stage of the data processing workflow. The workflow follows the Medallion architecture pattern (Bronze → Silver → Gold), progressively refining raw API data into analytically optimised dimension and fact tables.
The diagram below shows the end-to-end flow:

<img width="949" height="222" alt="image" src="https://github.com/user-attachments/assets/2cc283fc-b877-4adb-85cd-7baf4a1c704f" />


<br>


The table below summarises the role of each pipeline component:
| Component | Description |
|---|---|
| **Source Ingestion Pipeline** | Retrieves incident records daily from the AWSD API (JSON) into the Lakehouse — no transformation, source structure preserved |
| **Curation & Staging Load Pipeline** | Deduplicates, preprocesses, and standardises Lakehouse data; loads dimension and fact tables into the staging area |
| **Data Validation & Quality Assurance Pipeline** | Runs 5 quality rules in parallel — business key uniqueness, attribute uniqueness, referential integrity |
| **Data Warehouse Load Pipeline** | Loads validated staging data into the dimensional model; generates surrogate keys and resolves foreign keys |
| **Failure Notification Pipeline** | Sends an automated alert if any pipeline fails — enables early issue detection without manual monitoring |


The semantic model sits on top of the data warehouse and prepares the data for reporting in Power BI. It follows a star schema based on Kimball's dimensional modelling methodology. The grain is one row per recorded security incident. Measures include counts of affected individuals (killed, wounded, kidnapped, detained), disaggregated by nationality status, organisation type, and gender. Nine dimension tables provide analytical context — Date, Location, Attack Context, Attack Means, Incident Setting, Motive, Perpetrator Type, Source, and Verification Status. See the semantic model's diagram below: 

<img width="949" height="370" alt="image" src="https://github.com/user-attachments/assets/63ff3cce-ffc6-4908-b238-2dfe8ae92cc9" />


---

## Technical References
Expand any section below for detailed documentation.

<details>
<summary><strong>Reference 1 – Metadata and Column Descriptions</strong></summary>

This reference describes the AWSD dataset fields, including column names, data types, and definitions as provided by the source.

| Column name | Description | Data type and allowed values |
|---|---|---|
| Incident ID | Unique identifier for each incident record. | Integer. Unique per row. |
| Year | Calendar year in which the incident occurred. | Integer. Four-digit year |
| Month | Month of the incident, when known. | Integer. 1–12 |
| Day | Day of the month on which the incident occurred, when known. | Integer. 1–31; may be blank if the exact day is unknown or withheld for anonymisation (redacted). |
| Country Code | Standard country code for the incident location. | String. ISO 3166 country code |
| Country | Country in which the incident occurred. | String. Free text; country name. |
| Region | First sub-national administrative unit where the incident took place | String. Free text; region name |
| District | Second-level sub-national administrative unit for the incident location (province/district) | String. Free text; may be withheld for anonymisation (redacted). |
| City | Nearest named settlement or locality (city, town, village, checkpoint, road marker). | String. Free text; may be withheld for anonymisation (redacted). |
| UN | Number of affected aid workers employed by UN humanitarian agencies. | Non-negative integer; 0 if none recorded. |
| INGO | Number of affected staff from international non-governmental organisations. | Non-negative integer; 0 if none recorded. |
| ICRC | Number of affected staff of the International Committee of the Red Cross (ICRC). | Non-negative integer; 0 if none recorded. |
| NRCS and IFRC | Number of affected staff from National Red Cross or Red Crescent Societies and from the IFRC. | Non-negative integer; 0 if none recorded. |
| NNGO | Number of affected aid workers employed by national NGOs. | Non-negative integer; 0 if none recorded. |
| Other | Number of affected aid workers from other humanitarian organisations (e.g. donor agencies, other international organisations with humanitarian programming). | Non-negative integer; 0 if none recorded. |
| Nationals killed | Number of national (locally recruited) aid workers killed | Non-negative integer; 0 if none recorded. |
| Nationals wounded | Number of national (locally recruited) aid workers injured by deliberate violence (including landmines), with injuries serious enough to require medical treatment. | Non-negative integer; 0 if none recorded. |
| Nationals kidnapped | Number of national (locally recruited) aid workers who were abducted or held as hostages by non-state or criminal actors for at least 24 hours. If a victim is killed during a kidnapping, the case is treated as a killing, but the incident location reflects where the abduction began. | Non-negative integer; 0 if none recorded. |
| Nationals detained | Number of national (locally recruited) aid workers who were detained or placed under arrest for at least 24 hours by state authorities, police or de facto authorities; if a detainee is killed or seriously injured during detention, the case is instead counted under the killing or injury category. | Non-negative integer; 0 if none recorded. |
| Total nationals | Total number of national aid workers affected in the incident (killed + wounded + kidnapped + detained). | Non-negative integer; usually equals the sum of national outcome fields. |
| Internationals killed | Number of international (expatriate or mobile) aid workers killed | Non-negative integer; 0 if none recorded. |
| Internationals wounded | Number of international (expatriate or mobile) aid workers injured by deliberate violence (including landmines), with injuries serious enough to require medical treatment. | Non-negative integer; 0 if none recorded. |
| Internationals kidnapped | Number of international (expatriate or mobile) aid workers who were abducted or held as hostages by non-state or criminal actors for at least 24 hours. If a victim is killed during a kidnapping, the case is treated as a killing, but the incident location reflects where the abduction began. | Non-negative integer; 0 if none recorded. |
| Internationals detained | Number of international (expatriate or mobile) aid workers who were detained or placed under arrest for at least 24 hours by state authorities, police or de facto authorities; if a detainee is killed or seriously injured during detention, the case is instead counted under the killing or injury category. | Non-negative integer; 0 if none recorded. |
| Total internationals | Total number of international aid workers affected (killed + wounded + kidnapped + detained). | Non-negative integer; usually equals the sum of international outcome fields. |
| Total killed | Total number of aid workers killed (nationals + internationals). | Non-negative integer; 0 if no fatalities. |
| Total wounded | Total number of aid workers seriously wounded (nationals + internationals). | Non-negative integer; 0 if none wounded. |
| Total kidnapped | Total number of aid workers kidnapped, abducted or held hostage (nationals + internationals). | Non-negative integer; 0 if none kidnapped. |
| Total detained | Total number of aid workers detained or arrested (nationals + internationals). | Non-negative integer; 0 if none detained. |
| Total affected | Overall number of national and international aid workers affected (killed, wounded, kidnapped and detained). | Non-negative integer; typically equals Total killed + Total wounded + Total kidnapped + Total detained. |
| Gender Male | Number of male aid workers affected. | Non-negative integer; 0 if none recorded. |
| Gender Female | Number of female aid workers affected. | Non-negative integer; 0 if none recorded. |
| Gender Unknown | Number of affected aid workers whose gender is not reported or cannot be inferred. | Non-negative integer; 0 if gender is known for all victims. |
| Means of attack | Primary method, weapon or tactic used in the incident. | String (categorical). Permitted values (exhaustive): <br>• Aerial bombardment – attack using missiles, rockets, drones or other air-delivered weapons <br>• Bodily assault – beating or attack using hands or non-firearm weapons (e.g. knife, club) <br>• Shelling – artillery fire or other ground-launched explosive munitions <br>• Body-borne IED – explosive device carried on a person <br>• Detention/arrest – person seized and held by state or de facto authorities, without confirmed injury or death <br>• Complex attack – explosives used together with gunfire/small arms <br>• Roadside IED – improvised explosive device placed on or beside a road <br>• Vehicle-borne IED – explosive device in a vehicle, detonated remotely or by a suicide attacker <br>• Other explosives – explosive weapons not covered above (e.g. grenades, RPGs) <br>• Kidnapping – abduction where the victim is eventually released, rescued or escapes <br>• Kidnap-killing – kidnapping that results in the victim being killed <br>• Rape or serious sexual assault – sexual violence as the main form of attack <br>• Landmine – victim triggered a landmine <br>• Shooting – attack using small arms or light weapons (e.g. pistols, rifles, machine guns) <br>• Unknown – primary means of attack not reported or cannot be determined |
| Attack context | Operational or situational context in which the attack occurred. | String (categorical). Observed values (exhaustive): <br>• Ambush – attack on a vehicle or convoy while travelling on a road <br>• Combat / Crossfire – incident occurring during active fighting or police/security operations <br>• Individual attack or assassination – targeted attack on a specific person outside a wider battle <br>• Mob violence – assault carried out by a crowd or during rioting <br>• Raid – armed group entering a home, office or project site to carry out the attack <br>• Detention – incident happening during arrest, restraint or while the victim is being held <br>• Unknown – context not reported or cannot be determined from available information |
| Location | Type of physical location where the incident occurred. | String (categorical). Primary values: <br>• Home – incident at a private residence, not an office compound <br>• Office/compound – incident at an organisational office, base or compound (including guesthouses) <br>• Project site – incident at a work site such as a camp, clinic, hospital, distribution or project location <br>• Public location – incident in a general public area (e.g. street, market, café) <br>• Road – incident while travelling by vehicle <br>• Custody – incident while the victim was under the control of the perpetrator <br>• Unknown – information on the type of place is not available |
| Latitude | Latitude coordinate of the incident location in decimal degrees, generally approximated to the centre of the settlement, landmark or route segment and, where needed, adjusted to avoid exposing sensitive locations. | Decimal degrees; may be blank if not geocoded or if coordinates are withheld. |
| Longitude | Longitude coordinate of the incident location in decimal degrees, generally approximated to the centre of the settlement, landmark or route segment and, where needed, adjusted to avoid exposing sensitive locations. | Decimal degrees; may be blank if not geocoded or if coordinates are withheld. |
| Motive | Assessed motive of the perpetrator(s) with respect to the victim's role as an aid worker. | String (categorical). Permitted values (exhaustive): <br>• Political – attack linked to the victim's role or activities as an aid worker, aiming to disrupt, divert or punish aid for political/military/ideological reasons <br>• Economic – primary aim is material gain (e.g. robbery, banditry, ransom) <br>• Incidental – victim's aid-worker status was unknown or irrelevant, or they were off-duty <br>• Unknown – motive cannot be determined from the available information <br>• Disputed – information on motive is conflicting <br>• Other – the motive does not clearly fit another category or is not further specified |
| Actor type | Category of the primary perpetrator responsible for the incident. | String (categorical). Permitted values (exhaustive): <br>Individuals: <br>• Unaffiliated – no group <br>• Aid recipient – beneficiary with grievance <br>• Staff member – current/former staff with grievance <br>Organised groups: <br>• Criminal – organised criminal groups (gangs, pirates, cartels, etc.) <br>• State – foreign or coalition forces, host state, state: unknown, police or paramilitary <br>• Non-state armed groups – global, regional, national, subnational/local <br>• Unknown – perpetrator cannot be determined from available information |
| Actor name | Name or label of the perpetrator group, entity or individual, when reported; in many incidents the perpetrator is unknown or attribution is uncertain, with any further explanation recorded in the "Details" column. | String. Free text. |
| Details | Short narrative description of the incident, including what happened, to whom, and any clarifying information on context or perpetrator. | String. Free text. |
| Verified | Status of AWSD verification for this incident. | String (categorical). Permitted values (exhaustive): <br>• Verified – incident information confirmed by the affected agency or other verification process <br>• Pending – incident likely occurred but details are still being confirmed <br>• Archived – older or incomplete cases where further details may exist and can sometimes be provided on request |
| Source | Type of primary source from which information about the incident was obtained. | String. Primary values: <br>• Focal point – report from a security manager or security consortium <br>• Media – print, broadcast or online news <br>• Official report – formal document from a humanitarian agency, consortium or authority <br>• ACLED – incident imported via the Armed Conflict Location & Event Data partnership |

</details>

<details>
<summary><strong>Reference 2 – Pipeline Activities and Functions</strong></summary>

This reference lists the activities and functions used in each pipeline, including their execution phase, activity type, and purpose.

| Execution phase | Activity type | Activity name (pattern) | Function | Pipeline |
|---|---|---|---|---|
| Ingestion | Dataflow | DF_INGEST_LH_API_AWSD | Ingests incident data from the AWSD API and stores raw records in the Lakehouse. | PL_LOAD_LH_AWSD |
| Preparation | Dataflow | DF_DEDUPLICATE_LH_AWSD | Removes exact technical duplicates from the Lakehouse dataset using conservative deduplication logic. | PL_LOAD_STG_AWSD |
| Preparation | Dataflow | DF_STANDARDIZE_LH_AWSD | Applies preprocessing and standardization, including text normalization and controlled attribute harmonization. | PL_LOAD_STG_AWSD |
| Staging reset | Script | SQL CLEAR FACT TABLE STG | Clears the staging fact table to ensure a full and deterministic reload. | PL_LOAD_STG_AWSD |
| Staging reset | Script | SQL CLEAR ALL DIM TABLES STG | Clears all staging dimension tables prior to reload. | PL_LOAD_STG_AWSD |
| Control | Wait | WAIT FOR STG CLEAR COMPLETE | Ensures all staging tables have been successfully cleared before loading begins. | PL_LOAD_STG_AWSD |
| Dimension loading | Dataflow | DF_LOAD_DIM_*_STG | Loads individual dimension tables (date, location, motive, perpetrator type, source, etc.) into the staging area. | PL_LOAD_STG_AWSD |
| Control | Wait | WAIT FOR DIM LOAD COMPLETE | Ensures that all dimension tables are fully populated before fact loading. | PL_LOAD_STG_AWSD |
| Fact loading | Dataflow | DF_LOAD_FACT_INCIDENT_AWSD_STG | Loads the incident fact table into the staging area with resolved foreign keys. | PL_LOAD_STG_AWSD |
| Initialization | Script | SQL CLEAR LOG QUALITY CHECKS | Clears the data quality log table to remove results from previous validation runs. | PL_VALIDATE_STG_AWSD |
| Dimension validation control | Wait | CHECK STG DIM * | Control steps that initiate validation rules for each staging dimension table. | PL_VALIDATE_STG_AWSD |
| Rule 1 – Dimension BK integrity | Script | SQL RUN RULE 1 ON STG DIM * | Validates uniqueness of business keys in staging dimension tables. | PL_VALIDATE_STG_AWSD |
| Rule 2 – Dimension full-row uniqueness | Script | SQL RUN RULE 2 ON STG DIM LOCATION | Validates uniqueness of the full set of non-business-key attributes in multi-attribute dimensions. | PL_VALIDATE_STG_AWSD |
| Rule 3 – Single-attribute uniqueness | Script | SQL RUN RULE 3 ON STG DIM * | Validates uniqueness of single descriptive attributes in applicable dimension tables. | PL_VALIDATE_STG_AWSD |
| Fact validation control | Wait | CHECK STG FACT INCIDENT | Control step initiating validation rules for the staging fact table. | PL_VALIDATE_STG_AWSD |
| Rule 4 – Fact PK integrity | Script | SQL RUN RULE 4 ON STG FACT INCIDENT | Validates uniqueness of the fact table primary key (combination of all foreign keys and incident identifier). | PL_VALIDATE_STG_AWSD |
| Rule 5 – Referential integrity | Script | SQL RUN RULE 5 ON STG FACT INCIDENT * FK | Validates that all foreign keys in the fact table reference existing dimension business keys. | PL_VALIDATE_STG_AWSD |
| Non-executed check | Wait | Check STG Dim Date Not Required | Placeholder indicating that date dimension validation is not required due to controlled dimension generation. | PL_VALIDATE_STG_AWSD |
| DW reset | Script | SQL CLEAR FACT TABLE DW | Truncates the DW fact table to guarantee a full and deterministic reload and avoid dependency conflicts. | PL_LOAD_DW_AWSD |
| Control | Wait | WAIT FOR FACT TABLE DW CLEAR COMPLETE | Ensures the DW fact table has been cleared before dimension tables are cleared. | PL_LOAD_DW_AWSD |
| DW reset | Script | SQL CLEAR ALL DIM TABLES DW | Clears all DW dimension tables to establish a clean target state prior to loading. | PL_LOAD_DW_AWSD |
| Dimension loading | Dataflow | DF LOAD DIM *_DW | Loads DW dimensions (location, attack means/context, motive, perpetrator type, source, verification status, incident setting), generating surrogate keys where applicable. | PL_LOAD_DW_AWSD |
| Dimension loading | Copy | CD LOAD DIM DATE DW | Copies the Date dimension from STG to DW directly (no additional transformation or key generation required). | PL_LOAD_DW_AWSD |
| Control | Wait | WAIT FOR DIM LOAD COMPLETE | Ensures all dimensions (including date) are fully loaded before the fact table load begins. | PL_LOAD_DW_AWSD |
| Fact loading | Dataflow | DF LOAD FACT INCIDENT DW | Loads the DW fact table and resolves foreign keys by mapping staging business keys to DW surrogate keys. | PL_LOAD_DW_AWSD |

</details>

<details>
<summary><strong>Reference 3 – Data Preprocessing and Standardization Rules</strong></summary>

This reference documents the preprocessing and standardisation rules applied during the Curation & Staging Load pipeline, organised by category, operation, and affected columns.

| Category | Operation | Columns Affected | Rule / Modification |
|---|---|---|---|
| Text normalization | Trim text | All string columns | Removed leading and trailing whitespace |
| Text normalization | Clean text | All string columns | Removed non-printable characters |
| Missing value handling | Replace missing values | attack_context, means_of_attack, location | "" → "U" |
| Missing value handling | Replace missing values | motive, actor_type, verified | "" → "Unknown" |
| Capitalization | Standardize case | motive | Converted to proper case |
| Label standardization | Standardize labels | actor_type | "Host state" → "Host State"; "State: unknown" → "State: Unknown" |
| Label standardization | Standardize labels | source | "Focal point" → "Focal Point"; "media" → "Media"; "Offial report", "Offiicial Report", "Official report" → "Official Report" |
| Standardized fields | Create standardized columns | country_std, country_code_std, region_std, city_std, district_std | Added new \*_std columns (type: text) |
| Missing value handling | Default unknowns (hierarchy-aware) | country_std, country_code_std, region_std, city_std, district_std | If country is null/blank → "Unknown"; if respective field is null/blank → "Unknown" |
| Special-case mapping | Territorial/entity normalization | country_std | "Chechnya" → "Russia"; "Kashmir" → "Pakistan" |
| Special-case mapping | Standardized country codes | country_code_std | If country="Chechnya" → "RU"; if country="Kashmir" → "PK"; if country="Kosovo" → "XK" |
| Special-case mapping | Fill missing region based on country | region_std | If country="Chechnya" and region null/blank → "Chechnya"; if country="Kashmir" and region null/blank → "Kashmir" |
| Canonical country names | Shorten official/alternate names | country_std | "Syrian Arab Republic" → "Syria"; "Libyan Arab Jamahiriya" → "Libya"; "Iran, Islamic Republic of" → "Iran"; "Cote D'Ivoire" → "Ivory Coast"; "Occupied Palestinian Territories" → "Palestine"; "Swaziland" → "Eswatini" |
| Regional label standardization | Standardize region naming | region_std | "Donetsk" → "Donetsk Oblast"; "Kherson" → "Kherson Oblast"; "Ar-Raqqah" → "Raqqa"; "Lowgar" → "Logar" |
| City label standardization | Standardize city naming/case | city_std | "Ar-Raqqah" → "Raqqa"; "refugee" → "Refugee"; "camp" → "Camp" |
| Missing / non-informative values | Replace missing and "Not applicable" | actor_name | "" → "Unknown"; "Not applicable" → "Unknown"; "Not apliccble" → "Unknown" |
| Typo correction | Correct spelling variants | actor_name | "Uknown" → "Unknown"; "Husband of aid receipient" → "Husband of aid recipient"; "Unite pour la Paix en Centrafique (UPC)" → "Unite pour la Paix en Centrafrique (UPC)" |
| Category harmonization | Singular/plural normalization | actor_name | "Community member" → "Community members"; "Community memberss" → "Community members"; "Civilian" → "Civilians"; "Criminal" → "Criminals"; "IDP" → "IDPs"; "IDPss" → "IDPs"; "Internally Displaced People (IDPss)" → "IDPs"; "Youth" → "Youths" |
| Label standardization | Simplify label | actor_name | "Police officer" → "Police" |
| Spelling correction | Replacement as defined in query | actor_name | "Civilianss" → "Civilian" |

</details>

<details>
<summary><strong>Reference 4 – Data Quality Validation Results Log</strong></summary>

This reference logs the results of the five quality rules executed during the Data Validation & QA pipeline.

| id_check | etl_phase | etl_table | etl_checktype | description_result | etl_result |
|---|---|---|---|---|---|
| 1 | Staging Area | stg_dim_motive | check integrity of BK | number of rows with repeated BK: 0 | OK |
| 1 | Staging Area | stg_dim_location | check integrity of BK | number of rows with repeated BK: 0 | OK |
| 1 | Staging Area | stg_dim_attack_context | check integrity of BK | number of rows with repeated BK: 0 | OK |
| 1 | Staging Area | stg_dim_source | check integrity of BK | number of rows with repeated BK: 0 | OK |
| 1 | Staging Area | stg_dim_attack_means | check integrity of BK | number of rows with repeated BK: 0 | OK |
| 1 | Staging Area | stg_dim_perpetrator_type | check integrity of BK | number of rows with repeated BK: 0 | OK |
| 1 | Staging Area | stg_dim_incident_setting | check integrity of BK | number of rows with repeated BK: 0 | OK |
| 1 | Staging Area | stg_dim_verification_status | check integrity of BK | number of rows with repeated BK: 0 | OK |
| 2 | Staging Area | stg_dim_location | check uniqueness of all dim attributes | number of rows NOT unique: 0 | OK |
| 3 | Staging Area | stg_dim_perpetrator_type | check uniqueness of dimension attribute | number of duplicate attribute values: 0 | OK |
| 3 | Staging Area | stg_dim_verification_status | check uniqueness of dimension attribute | number of duplicate attribute values: 0 | OK |
| 3 | Staging Area | stg_dim_incident_setting | check uniqueness of dimension attribute | number of duplicate attribute values: 0 | OK |
| 3 | Staging Area | stg_dim_attack_context | check uniqueness of dimension attribute | number of duplicate attribute values: 0 | OK |
| 3 | Staging Area | stg_dim_motive | check uniqueness of dimension attribute | number of duplicate attribute values: 0 | OK |
| 3 | Staging Area | stg_dim_source | check uniqueness of dimension attribute | number of duplicate attribute values: 0 | OK |
| 3 | Staging Area | stg_dim_attack_means | check uniqueness of dimension attribute | number of duplicate attribute values: 0 | OK |
| 4 | Staging Area | stg_fact_incident | check integrity of fact PK (combo all FKs) + ID | number of rows with repeated PK: 0 | OK |
| 5 | Staging Area | stg_fact_incident | check parent of FK for verification_status dimension | number of rows without parent key: 0 | OK |
| 5 | Staging Area | stg_fact_incident | check parent of FK for attack_context dimension | number of rows without parent key: 0 | OK |
| 5 | Staging Area | stg_fact_incident | check parent of FK for date dimension | number of rows without parent key: 0 | OK |
| 5 | Staging Area | stg_fact_incident | check parent of FK for location dimension | number of rows without parent key: 0 | OK |
| 5 | Staging Area | stg_fact_incident | check parent of FK for perpetrator_type dimension | number of rows without parent key: 0 | OK |
| 5 | Staging Area | stg_fact_incident | check parent of FK for incident_setting dimension | number of rows without parent key: 0 | OK |
| 5 | Staging Area | stg_fact_incident | check parent of FK for attack_means dimension | number of rows without parent key: 0 | OK |
| 5 | Staging Area | stg_fact_incident | check parent of FK for motive dimension | number of rows without parent key: 0 | OK |
| 5 | Staging Area | stg_fact_incident | check parent of FK for source dimension | number of rows without parent key: 0 | OK |

</details>

<details>
<summary><strong>Reference 5 – Semantic Model: Tables and Columns</strong></summary>

This reference lists all tables and columns in the Power BI semantic model, organised by keys, attributes, and core analytical measures.

| Table | Keys | Attributes | Core Analytical Measures |
|---|---|---|---|
| F Incident | incident_id, fk_date, fk_location, fk_attack_context, fk_attack_means, fk_incident_setting, fk_motive, fk_perpetrator_type, fk_source, fk_verification_status | Details, Perpetrator Name, Latitude, Longitude | Nationals Detained, Nationals Kidnapped, Nationals Killed, Nationals Wounded, Internationals Detained, Internationals Kidnapped, Internationals Killed, Internationals Wounded, Total Detained, Total Kidnapped, Total Killed, Total Wounded, Total Nationals Detained, Total Nationals Kidnapped, Total Nationals Killed, Total Nationals Wounded, Total Internationals Detained, Total Internationals Kidnapped, Total Internationals Killed, Total Internationals Wounded, Total Victims IFRC ICRC, Total Victims INGO, Total Victims NNGO, Total Victims Other Org, Total Victims Males, Total Victims Females, Total Victims Gender Unknown |
| D Date | sk_date | Date, Full Date, Month, Month Abbrev, Month Number, Monthday Number, Quarter, Quarter Name, Weekday, Weekday Abbrev, Weekday Number, Weekday Type, Year | N/A |
| D Location | bk_location, sk_location | City, Country, Country Code, District, Region | N/A |
| D Attack Means | bk_attack_means, sk_attack_means | Attack Means | N/A |
| D Attack Context | bk_attack_context, sk_attack_context | Attack Context | N/A |
| D Incident Setting | bk_incident_setting, sk_incident_setting | Incident Setting | N/A |
| D Motive | bk_motive, sk_motive | Motive | N/A |
| D Perpetrator Type | bk_perpetrator_type, sk_perpetrator_type | Perpetrator Type | N/A |
| D Source | bk_source, sk_source | Source | N/A |
| D Verification Status | bk_verification_status, sk_verification_status | Verification Status | N/A |

</details>

<details>
<summary><strong>Reference 6 – Full DAX Measure Catalogue</strong></summary>

This reference lists the measures used in the Power BI report and their DAX expressions.


<details>
<summary><strong>% Nationals</strong></summary>

```DAX

DIVIDE(
    [Selected Outcome Total — Nationals],
    [Selected Outcome Total1]
)

```

</details>

<details>
<summary><strong>All Countries Selected</strong></summary>

```DAX

VAR _selected =
    COUNTROWS ( ALLSELECTED ( 'D Location'[Country] ) )
VAR _total =
    COUNTROWS ( ALL ( 'D Location'[Country] ) )
RETURN
IF ( _selected = _total, 1, 0 )
```

</details>

<details>
<summary><strong>All Countries Selected (Slicer)</strong></summary>

```DAX

VAR Sel =
    CALCULATE(
        DISTINCTCOUNT('D Location'[Country]),
        ALLSELECTED('D Location'[Country]),
        REMOVEFILTERS('D Location'[Country])   
    )
VAR AllC =
    CALCULATE(
        DISTINCTCOUNT('D Location'[Country]),
        ALL('D Location'[Country])
    )
RETURN
IF( Sel = 0 || Sel = AllC, 1, 0 )

```

</details>

<details>
<summary><strong>Attack Dimension Definition</strong></summary>

```DAX

VAR DimSel =
    SELECTEDVALUE('Attack Dimension Parameter'[Attack Dimension Parameter Fields])

RETURN
SWITCH(
    TRUE(),

    CONTAINSSTRING(DimSel, "Attack Means"),
        SELECTEDVALUE(AttackMeans_Definitions[Definition]),

    CONTAINSSTRING(DimSel, "Attack Context"),
        SELECTEDVALUE(AttackContext_Definitions[Definition]),

    CONTAINSSTRING(DimSel, "Incident Setting"),
        SELECTEDVALUE(IncidentSetting_Definitions[Definition]),

    BLANK()
)
```

</details>

<details>
<summary><strong>Comparison Value (Outcome)</strong></summary>

```DAX

SWITCH(
    SELECTEDVALUE('Compare Period'[Period]),
    "Selected year", [Selected Year (Outcome)],
    "Previous year", [Previous Year (Outcome)],
    "3-year avg",    [Outcome (3-year avg)],
    BLANK()
)

```

</details>

<details>
<summary><strong>Country Filter Applied</strong></summary>

```DAX

VAR SelCountries =
    COUNTROWS(ALLSELECTED('D Location'[Country]))
VAR AllCountries =
    COUNTROWS(ALL('D Location'[Country]))
RETURN
IF(
    SelCountries = 0 || SelCountries = AllCountries,
    0,   -- not applied (All)
    1    -- applied (some subset selected)
)

```

</details>

<details>
<summary><strong>Country Filter Applied (Slicer Only)</strong></summary>

```DAX

VAR Sel =
    CALCULATE(
        DISTINCTCOUNT('D Location'[Country]),
        ALLSELECTED('D Location'[Country]),
        REMOVEFILTERS('D Location'[Country])   -- ignores visual-level filtering like Top 5
    )
VAR AllC =
    CALCULATE(
        DISTINCTCOUNT('D Location'[Country]),
        ALL('D Location'[Country])
    )
RETURN
IF(
    Sel = 0 || Sel = AllC,
    0,   -- slicer is effectively "All"
    1    -- slicer is selecting a subset
)

```

</details>

<details>
<summary><strong>Country Phrase (Map Title)</strong></summary>

```DAX

IF(
    [All Countries Selected] = 1,
    "worldwide",
    "in " & SELECTEDVALUE('D Location'[Country])
)

```

</details>

<details>
<summary><strong>Country Rank (Incidents)</strong></summary>

```DAX

RANKX (
    ALL ( 'D Location'[Country] ),
    [Total Incidents],
    ,
    DESC,
    DENSE
)
```

</details>

<details>
<summary><strong>Country Rank (Nat+Int)</strong></summary>

```DAX

RANKX(
    ALLSELECTED('D Location'[Country]),
    [Selected Outcome Total — Nat+Int],
    ,
    DESC,
    DENSE
)

```

</details>

<details>
<summary><strong>Country Rank (Selected Outcome)</strong></summary>

```DAX

RANKX(
    ALLSELECTED('D Location'[Country]),
    [Selected Outcome Total1],
    ,
    DESC,
    DENSE
)

```

</details>

<details>
<summary><strong>Country Total (TopN Perpetrator Heatmap)</strong></summary>

```DAX

CALCULATE(
    [Selected Outcome Total1],
    REMOVEFILTERS('Perpetrator Dimension Parameter')
)

```

</details>

<details>
<summary><strong>Country Total (TopN)</strong></summary>

```DAX

CALCULATE(
    [Selected Outcome Total1],
    REMOVEFILTERS('Attack Dimension Parameter')
)

```

</details>

<details>
<summary><strong>Dynamic Tree Title</strong></summary>

```DAX

VAR _outcome =
    SELECTEDVALUE(Outcome[Outcome], "Incidents")

VAR _country =
    SELECTEDVALUE('D Location'[Country], "All")

VAR _year =
    SELECTEDVALUE('D Date'[Year], "All")

VAR _value =
    [Selected Outcome Total1]

VAR _outcomeText =
    SWITCH(
        _outcome,
        "Incidents", "incidents",
        "All", "all affected aid workers",
        "Killed", "aid workers killed",
        "Wounded", "aid workers wounded",
        "Kidnapped", "aid workers kidnapped",
        "Detained", "aid workers detained",
        _outcome
    )

VAR _countryText =
    IF(
        _country = "All",
        "worldwide",
        "in " & _country
    )

VAR _yearText =
    IF(
        _year = "All",
        "across all reported years",
        "in " & _year
    )

VAR _noDataText =
    IF(
        _value = 0,
        " (no incidents recorded)",
        ""
    )

RETURN
"Aid worker security incidents – "
    & _outcomeText & " "
    & _countryText & " "
    & _yearText
    & _noDataText

```

</details>

<details>
<summary><strong>First Data Year</strong></summary>

```DAX

CALCULATE (
    MIN ( 'D Date'[Year] ),
    ALL ( 'D Date' )
)

```

</details>

<details>
<summary><strong>Gender Female Total</strong></summary>

```DAX

SUM('F Incident'[Gender Female])

```

</details>

<details>
<summary><strong>Gender Male Total</strong></summary>

```DAX

SUM('F Incident'[Gender Male])

```

</details>

<details>
<summary><strong>Gender Total (All outcomes)</strong></summary>

```DAX

SWITCH(
    SELECTEDVALUE('D Gender'[Gender]),
    "Female",  SUM('F Incident'[Gender Female]),
    "Male",    SUM('F Incident'[Gender Male]),
    "Unknown", SUM('F Incident'[Gender Unknown]),
    BLANK()
)

```

</details>

<details>
<summary><strong>Gender Unknown Total</strong></summary>

```DAX

SUM('F Incident'[Gender Unknown])

```

</details>

<details>
<summary><strong>Has Data (Nat+Int)</strong></summary>

```DAX

IF ( [Selected Outcome Total — Nat+Int1] > 0, 1, 0 )
```

</details>

<details>
<summary><strong>HelpHover</strong></summary>

```DAX
"ⓘ"
```

</details>

<details>
<summary><strong>Incidents Kidnapped</strong></summary>

```DAX

SUMX(
    VALUES('F Incident'[Incident ID]),
    VAR v = [Total Kidnapped]
    RETURN IF( v > 0, 1, 0 )
)

```

</details>

<details>
<summary><strong>Incidents Killed</strong></summary>

```DAX

SUMX(
    VALUES('F Incident'[Incident ID]),
    VAR v = [Total Killed]
    RETURN IF( v > 0, 1, 0 )
)
```

</details>

<details>
<summary><strong>Incidents Wounded</strong></summary>

```DAX

SUMX(
    VALUES('F Incident'[Incident ID]),
    VAR v = [Total Wounded]
    RETURN IF( v > 0, 1, 0 )
)

```

</details>

<details>
<summary><strong>Internationals by Outcome (Axis)</strong></summary>

```DAX

VAR o = SELECTEDVALUE('D Outcome Axis'[Outcome])
RETURN
SWITCH(
    o,
    "Killed",     SUM('F Incident'[Internationals Killed]),
    "Wounded",    SUM('F Incident'[Internationals Wounded]),
    "Kidnapped",  SUM('F Incident'[Internationals Kidnapped]),
    "Detained",   SUM('F Incident'[Internationals Detained])
)

```

</details>

<details>
<summary><strong>Kidnapped (3-year avg)</strong></summary>

```DAX

VAR IsSingleYear = HASONEVALUE ( 'D Date'[Year] )
VAR Y = SELECTEDVALUE ( 'D Date'[Year] )
VAR FirstY = [First Data Year]

VAR V1 =
    CALCULATE (
        [Total Kidnapped],
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 1
    )
VAR V2 =
    CALCULATE (
        [Total Kidnapped],
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 2
    )
VAR V3 =
    CALCULATE (
        [Total Kidnapped],
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 3
    )

VAR YearsOverZero =
    IF ( NOT ISBLANK ( V1 ) && V1 > 0, 1, 0 ) +
    IF ( NOT ISBLANK ( V2 ) && V2 > 0, 1, 0 ) +
    IF ( NOT ISBLANK ( V3 ) && V3 > 0, 1, 0 )

RETURN
IF (
    NOT IsSingleYear
        || Y <= FirstY + 2
        || YearsOverZero < 2,
    BLANK(),
    DIVIDE ( COALESCE ( V1, 0 ) + COALESCE ( V2, 0 ) + COALESCE ( V3, 0 ), 3 )
)

```

</details>

<details>
<summary><strong>Kidnapped (line)</strong></summary>

```DAX

IF (
    SELECTEDVALUE ( Outcome[Outcome], "Killed" ) = "Kidnapped",
    [Total Kidnapped],
    BLANK()
)

```

</details>

<details>
<summary><strong>Kidnapped (Previous Year)</strong></summary>

```DAX

VAR Y = [Selected Year]
VAR FirstY = [First Data Year]
RETURN
IF (
    Y <= FirstY,
    BLANK(),
    CALCULATE (
        [Total Kidnapped],
        FILTER ( ALL ( 'D Date'[Year] ), 'D Date'[Year] = Y - 1 )
    )
)

```

</details>

<details>
<summary><strong>Kidnapped (Selected Year)</strong></summary>

```DAX

VAR Y = [Selected Year]
RETURN
CALCULATE(
    [Total Kidnapped],
    FILTER(ALL('D Date'[Year]), 'D Date'[Year] = Y)
)

```

</details>

<details>
<summary><strong>Kidnapped Comparison Title</strong></summary>

```DAX

VAR IsSingleYear = HASONEVALUE ( 'D Date'[Year] )

VAR IsGlobal =
    NOT ISFILTERED ( 'D Location'[Country] )

VAR CountryLabel =
    SELECTEDVALUE ( 'D Location'[Country] )

VAR TitlePrefix =
    IF (
        IsGlobal,
        "Kidnapped globally — ",
        "Kidnapped in " & CountryLabel & " — "
    )

VAR Y = SELECTEDVALUE ( 'D Date'[Year] )
VAR CurKidnapped = COALESCE ( [Total Kidnapped], 0 )

VAR CurIncRows =
    CALCULATE (
        COUNTROWS ( 'F Incident' ),
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y
    )

VAR PrevIncRows =
    CALCULATE (
        COUNTROWS ( 'F Incident' ),
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 1
    )

VAR FirstIncidentYear =
    MINX (
        FILTER (
            ALL ( 'D Date'[Year] ),
            CALCULATE ( COUNTROWS ( 'F Incident' ) ) > 0
        ),
        'D Date'[Year]
    )

VAR PrevKidnapped =
    COALESCE (
        CALCULATE (
            [Total Kidnapped],
            REMOVEFILTERS ( 'D Date'[Year] ),
            'D Date'[Year] = Y - 1
        ),
        0
    )

VAR YoY_Pct =
    IF ( PrevKidnapped > 0, DIVIDE ( CurKidnapped - PrevKidnapped, PrevKidnapped ), BLANK () )

VAR BracketLabel =
    SWITCH (
        TRUE (),

        PrevIncRows = 0,
            BLANK (),

        PrevKidnapped > 0 && CurKidnapped > PrevKidnapped,
            "(" & UNICHAR ( 9650 ) & " " & FORMAT ( YoY_Pct, "0%" ) & ")",

        PrevKidnapped > 0 && CurKidnapped < PrevKidnapped,
            "(" & UNICHAR ( 9660 ) & " " & FORMAT ( ABS ( YoY_Pct ), "0%" ) & ")",

        PrevKidnapped > 0 && CurKidnapped = PrevKidnapped,
            "(no change)",

        PrevKidnapped = 0 && CurKidnapped > 0,
            "(" & UNICHAR ( 9650 ) & " from 0)",

        PrevKidnapped = 0 && CurKidnapped = 0,
            "(no change)"
    )

VAR Avg3 = [Kidnapped (3-year avg)]
VAR AvgText =
    IF (
        NOT ISBLANK ( Avg3 ),
        " and the 3-year average (" & FORMAT ( Y - 3, "0" ) & "–" & FORMAT ( Y - 1, "0" ) & ")",
        BLANK ()
    )

VAR MainText =
    SWITCH (
        TRUE (),

        NOT IsSingleYear,
            "Graph not applicable for multiple years",

        CurIncRows = 0,
            FORMAT ( Y, "0" ) & " (no incidents recorded)",

        Y = FirstIncidentYear,
            FORMAT ( Y, "0" ) & " (first year with incidents recorded)",

        PrevIncRows = 0,
            FORMAT ( Y, "0" ) & " (no incidents recorded in " & FORMAT ( Y - 1, "0" ) & ")",

        TRUE,
            FORMAT ( Y, "0" ) &
            " compared with " & FORMAT ( Y - 1, "0" ) &
            IF ( NOT ISBLANK ( BracketLabel ), " " & BracketLabel, BLANK () ) &
            AvgText
    )

RETURN
TitlePrefix & MainText

```

</details>

<details>
<summary><strong>Killed (3-year avg)</strong></summary>

```DAX

VAR IsSingleYear = HASONEVALUE ( 'D Date'[Year] )
VAR Y = SELECTEDVALUE ( 'D Date'[Year] )
VAR FirstY = [First Data Year]

VAR V1 =
    CALCULATE (
        [Total Killed],
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 1
    )
VAR V2 =
    CALCULATE (
        [Total Killed],
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 2
    )
VAR V3 =
    CALCULATE (
        [Total Killed],
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 3
    )

VAR YearsOverZero =
    IF ( NOT ISBLANK ( V1 ) && V1 > 0, 1, 0 ) +
    IF ( NOT ISBLANK ( V2 ) && V2 > 0, 1, 0 ) +
    IF ( NOT ISBLANK ( V3 ) && V3 > 0, 1, 0 )

RETURN
IF (
    NOT IsSingleYear
        || Y <= FirstY + 2
        || YearsOverZero < 2,
    BLANK(),
    DIVIDE ( COALESCE ( V1, 0 ) + COALESCE ( V2, 0 ) + COALESCE ( V3, 0 ), 3 )
)

```

</details>

<details>
<summary><strong>Killed (line)</strong></summary>

```DAX

IF (
    SELECTEDVALUE ( Outcome[Outcome], "Killed" ) = "Killed",
    [Total Killed],
    BLANK()
)

```

</details>

<details>
<summary><strong>Killed (Previous Year)</strong></summary>

```DAX

VAR Y = [Selected Year]
VAR FirstY = [First Data Year]
RETURN
IF (
    Y <= FirstY,
    BLANK(),
    CALCULATE (
        [Total Killed],
        FILTER ( ALL ( 'D Date'[Year] ), 'D Date'[Year] = Y - 1 )
    )
)

```

</details>

<details>
<summary><strong>Killed (Selected Year)</strong></summary>

```DAX

VAR Y = [Selected Year]
RETURN
CALCULATE(
    [Total Killed],
    FILTER(ALL('D Date'[Year]), 'D Date'[Year] = Y)
)

```

</details>

<details>
<summary><strong>Killed Comparison Title</strong></summary>

```DAX

VAR IsSingleYear = HASONEVALUE ( 'D Date'[Year] )

VAR IsGlobal =
    NOT ISFILTERED ( 'D Location'[Country] )

VAR CountryLabel =
    SELECTEDVALUE ( 'D Location'[Country] )

VAR TitlePrefix =
    IF (
        IsGlobal,
        "Killed globally — ",
        "Killed in " & CountryLabel & " — "
    )

VAR Y = SELECTEDVALUE ( 'D Date'[Year] )
VAR CurKilled = COALESCE ( [Total Killed], 0 )

-- incident rows in selected year / prev year for the current context (country etc.)
VAR CurIncRows =
    CALCULATE (
        COUNTROWS ( 'F Incident' ),
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y
    )

VAR PrevIncRows =
    CALCULATE (
        COUNTROWS ( 'F Incident' ),
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 1
    )

-- first year with ANY incidents recorded in the current context
VAR FirstIncidentYear =
    MINX (
        FILTER (
            ALL ( 'D Date'[Year] ),
            CALCULATE ( COUNTROWS ( 'F Incident' ) ) > 0
        ),
        'D Date'[Year]
    )

-- previous year killed value (0 if none)
VAR PrevKilled =
    COALESCE (
        CALCULATE (
            [Total Killed],
            REMOVEFILTERS ( 'D Date'[Year] ),
            'D Date'[Year] = Y - 1
        ),
        0
    )

VAR YoY_Pct =
    IF ( PrevKilled > 0, DIVIDE ( CurKilled - PrevKilled, PrevKilled ), BLANK () )

-- bracket label rules
VAR BracketLabel =
    SWITCH (
        TRUE (),

        PrevIncRows = 0,
            BLANK (),  -- don't show brackets if there were no incidents at all in prev year

        PrevKilled > 0 && CurKilled > PrevKilled,
            "(" & UNICHAR ( 9650 ) & " " & FORMAT ( YoY_Pct, "0%" ) & ")",

        PrevKilled > 0 && CurKilled < PrevKilled,
            "(" & UNICHAR ( 9660 ) & " " & FORMAT ( ABS ( YoY_Pct ), "0%" ) & ")",

        PrevKilled > 0 && CurKilled = PrevKilled,
            "(no change)",

        PrevKilled = 0 && CurKilled > 0,
            "(" & UNICHAR ( 9650 ) & " from 0)",

        PrevKilled = 0 && CurKilled = 0,
            "(no change)"
    )

-- 3-year avg text only when your avg measure returns a value
VAR Avg3 = [Killed (3-year avg)]
VAR AvgText =
    IF (
        NOT ISBLANK ( Avg3 ),
        " and the 3-year average (" & FORMAT ( Y - 3, "0" ) & "–" & FORMAT ( Y - 1, "0" ) & ")",
        BLANK ()
    )

VAR MainText =
    SWITCH (
        TRUE (),

        NOT IsSingleYear,
            "Comparison chart not available when Years = All",

        -- No incidents in selected year: be explicit
        CurIncRows = 0,
            FORMAT ( Y, "0" ) & " (no incidents recorded)",

        -- First year with incidents: this is your Ukraine 2022 case
        Y = FirstIncidentYear,
            FORMAT ( Y, "0" ) & " (first year with incidents recorded)",

        -- Incidents exist this year but not last year
        PrevIncRows = 0,
            FORMAT ( Y, "0" ) & " (no incidents recorded in " & FORMAT ( Y - 1, "0" ) & ")",

        -- Normal comparison
        TRUE,
            FORMAT ( Y, "0" ) &
            " compared with " & FORMAT ( Y - 1, "0" ) &
            IF ( NOT ISBLANK ( BracketLabel ), " " & BracketLabel, BLANK () ) &
            AvgText
    )

RETURN
TitlePrefix & MainText

```

</details>

<details>
<summary><strong>Killed YoY %</strong></summary>

```DAX

VAR Y = SELECTEDVALUE('D Date'[Year])
VAR Cur = [Total Killed]
VAR Prev =
    CALCULATE(
        [Total Killed],
        REMOVEFILTERS('D Date'[Year]),
        'D Date'[Year] = Y - 1
    )
RETURN
IF(
    NOT HASONEVALUE('D Date'[Year]),
    BLANK(),
    IF(
        ISBLANK(Prev) || Prev = 0,
        BLANK(),
        DIVIDE(Cur - Prev, Prev)
    )
)

```

</details>

<details>
<summary><strong>Killed YoY Text</strong></summary>

```DAX

VAR Y = SELECTEDVALUE('D Date'[Year])
VAR Cur = [Total Killed]
VAR Prev =
    CALCULATE(
        [Total Killed],
        REMOVEFILTERS('D Date'[Year]),
        'D Date'[Year] = Y - 1
    )
VAR Pct = [Killed YoY %]
RETURN
IF(
    NOT HASONEVALUE('D Date'[Year]),
    "Select a single year",
    IF(
        ISBLANK(Prev),
        "N/A vs previous year",
        IF(
            Prev = 0,
            IF(Cur = 0, "No change vs previous year", "New vs previous year"),
            IF(
                Pct > 0,
                UNICHAR(9650) & " " & FORMAT(Pct, "0%") & " vs previous year",
                IF(
                    Pct < 0,
                    UNICHAR(9660) & " " & FORMAT(ABS(Pct), "0%") & " vs previous year",
                    "No change vs previous year"
                )
            )
        )
    )
)

```

</details>

<details>
<summary><strong>KPI Column Header</strong></summary>

```DAX

VAR _countryText =
    IF (
        [All Countries Selected] = 1,
        "Worldwide",
        SELECTEDVALUE ( 'D Location'[Country] )
    )
RETURN
    _countryText & " " & [Year Phrase (Title)]
```

</details>

<details>
<summary><strong>Last Refresh Text</strong></summary>

```DAX

"Report last refreshed on " & 
FORMAT(
    MAX(refresh_audit[last_successful_load]),
    "dd mmmm yyyy"
) & "."

```

</details>

<details>
<summary><strong>Line Title</strong></summary>

```DAX

VAR O = SELECTEDVALUE(Outcome[Outcome], "Killed")
RETURN
"Trend in aid workers " & LOWER(O) & " over time (total numbers)"

```

</details>

<details>
<summary><strong>Map Title</strong></summary>

```DAX

VAR O = SELECTEDVALUE(Outcome[Outcome], "Killed")
RETURN
"Geographic distribution of aid workers " & LOWER(O) & " (total numbers)"
```

</details>

<details>
<summary><strong>Map Title (Dynamic)</strong></summary>

```DAX

VAR _selOutcome =
    SELECTEDVALUE ( Outcome[Outcome], "All" )

VAR _metricText =
    IF ( _selOutcome = "Incidents", "Incidents", [Outcome Phrase (Title)] )

VAR _countryText = "globally"

/* Show note ONLY when a subset of non-blank countries is selected */
VAR _availableCountries =
    CALCULATETABLE (
        VALUES ( 'D Location'[Country] ),
        REMOVEFILTERS ( 'D Location'[Country] ),
        'D Location'[Country] <> BLANK()
    )

VAR _selectedCountries =
    FILTER (
        VALUES ( 'D Location'[Country] ),
        'D Location'[Country] <> BLANK()
    )

VAR _countryFiltered =
    COUNTROWS ( EXCEPT ( _availableCountries, _selectedCountries ) ) > 0

VAR _countryNote =
    IF ( _countryFiltered, " (not impacted by country selection)", "" )

VAR _baseTitleRaw =
    "Total "
        & _metricText & " "
        & _countryText & " "
        & [Year Phrase (Title)]

VAR _baseTitleNoTotal =
    SUBSTITUTE ( _baseTitleRaw, "Total ", "" )

VAR _baseTitleCased =
    UPPER ( LEFT ( _baseTitleNoTotal, 1 ) )
        & MID ( _baseTitleNoTotal, 2, LEN ( _baseTitleNoTotal ) - 1 )

VAR _valueGlobal =
    CALCULATE ( [Selected Outcome Total1], REMOVEFILTERS ( 'D Location'[Country] ) )

VAR _noRecorded =
    ISBLANK ( _valueGlobal ) || _valueGlobal = 0

VAR _noDataSuffix =
    IF ( _selOutcome = "Incidents", " (no incidents recorded)", " (no data recorded)" )

RETURN
    IF (
        _noRecorded,
        _baseTitleCased & _countryNote & _noDataSuffix,
        _baseTitleCased & _countryNote
    )
```

</details>

<details>
<summary><strong>Map Value</strong></summary>

```DAX

VAR O = SELECTEDVALUE ( Outcome[Outcome], "Killed" )
RETURN
SWITCH(
    O,
    "Killed",    [Total Killed],
    "Wounded",   [Total Wounded],
    "Kidnapped", [Total Kidnapped],
    BLANK()
)

```

</details>

<details>
<summary><strong>Msg — Select one year</strong></summary>

```DAX

IF(
    HASONEVALUE('D Date'[Year]),
    "",
    "Select one year to enable comparison"
)

```

</details>

<details>
<summary><strong>Nat vs Int Difference</strong></summary>

```DAX

[Selected Outcome Total — Nationals]
-
[Selected Outcome Total — Internationals]

```

</details>

<details>
<summary><strong>Nationality Group</strong></summary>

```DAX

DATATABLE(
    "Group", STRING,
    {
        {"Nationals"},
        {"Internationals"}
    }
)
```

</details>

<details>
<summary><strong>Nationals by Outcome (Axis)</strong></summary>

```DAX

VAR o = SELECTEDVALUE('D Outcome Axis'[Outcome])
RETURN
SWITCH(
    o,
    "Killed",     SUM('F Incident'[Nationals Killed]),
    "Wounded",    SUM('F Incident'[Nationals Wounded]),
    "Kidnapped",  SUM('F Incident'[Nationals Kidnapped]),
    "Detained",   SUM('F Incident'[Nationals Detained])
)

```

</details>

<details>
<summary><strong>Org Victims Total (Pie)</strong></summary>

```DAX
VAR _org = SELECTEDVALUE('D Org Type'[Org Type]) RETURN SWITCH( _org, "IFRC/ICRC", CALCULATE([Total Victims IFRC ICRC], 
REMOVEFILTERS(Outcome[Outcome])), "INGO", CALCULATE([Total Victims INGO], 
REMOVEFILTERS(Outcome[Outcome])), "NNGO", CALCULATE([Total Victims NNGO], 
REMOVEFILTERS(Outcome[Outcome])), "NRCS", CALCULATE([Total Victims NRCS], 
REMOVEFILTERS(Outcome[Outcome])), "UN", CALCULATE([Total Victims UN], 
REMOVEFILTERS(Outcome[Outcome])), "Other", CALCULATE([Total Victims Other Org], 
REMOVEFILTERS(Outcome[Outcome])), BLANK() )
```

</details>

<details>
<summary><strong>Outcome (3-year avg)</strong></summary>

```DAX

VAR IsSingleYear = HASONEVALUE ( 'D Date'[Year] )
VAR Y            = SELECTEDVALUE ( 'D Date'[Year] )
VAR FirstY       = [First Data Year]

VAR V1 =
    CALCULATE (
        [Selected Outcome Total1],
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 1
    )
VAR V2 =
    CALCULATE (
        [Selected Outcome Total1],
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 2
    )
VAR V3 =
    CALCULATE (
        [Selected Outcome Total1],
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 3
    )

VAR YearsOverZero =
    IF ( NOT ISBLANK ( V1 ) && V1 > 0, 1, 0 ) +
    IF ( NOT ISBLANK ( V2 ) && V2 > 0, 1, 0 ) +
    IF ( NOT ISBLANK ( V3 ) && V3 > 0, 1, 0 )

RETURN
IF (
    NOT IsSingleYear
        || Y <= FirstY + 2
        || YearsOverZero < 2,
    BLANK(),
    DIVIDE ( COALESCE ( V1, 0 ) + COALESCE ( V2, 0 ) + COALESCE ( V3, 0 ), 3 )
)

```

</details>

<details>
<summary><strong>Outcome Comparison Title</strong></summary>

```DAX

/*** YEAR SELECTION (robust, ignores blank Year) ***/
VAR _yearsSelectedNonBlank =
    COUNTROWS (
        FILTER ( ALLSELECTED ( 'D Date'[Year] ), NOT ISBLANK ( 'D Date'[Year] ) )
    )

VAR _isSingleYear =
    _yearsSelectedNonBlank = 1

VAR _Y =
    MAXX (
        FILTER ( ALLSELECTED ( 'D Date'[Year] ), NOT ISBLANK ( 'D Date'[Year] ) ),
        'D Date'[Year]
    )

/*** COUNTRY SELECTION (robust “worldwide”, ignores blank Country) ***/
VAR _availableCountries =
    CALCULATETABLE (
        VALUES ( 'D Location'[Country] ),
        REMOVEFILTERS ( 'D Location'[Country] ),
        'D Location'[Country] <> BLANK ()
    )

VAR _selectedCountries =
    FILTER ( VALUES ( 'D Location'[Country] ), 'D Location'[Country] <> BLANK () )

VAR _isWorldwide =
    COUNTROWS ( EXCEPT ( _availableCountries, _selectedCountries ) ) = 0

VAR _CountryLabel =
    SELECTEDVALUE ( 'D Location'[Country] )

VAR _O =
    SELECTEDVALUE ( Outcome[Outcome], "All" )

/*** Outcome label ***/
VAR _OutcomeLabel =
    SWITCH (
        _O,
        "Incidents", "Incidents",
        "All",       "Aid workers affected",
        "Affected",  "Aid workers affected",
        "Killed",    "Aid workers killed",
        "Wounded",   "Aid workers wounded",
        "Kidnapped", "Aid workers kidnapped",
        "Detained",  "Aid workers detained",
        "Affected aid workers"
    )

VAR _TitlePrefix =
    SWITCH (
        TRUE(),
        _isWorldwide, _OutcomeLabel & " worldwide — ",
        NOT ISBLANK ( _CountryLabel ), _OutcomeLabel & " in " & _CountryLabel & " — ",
        _OutcomeLabel & " in selected countries — "
    )

/*** If not exactly one year, prompt user ***/
VAR _PromptText =
    "year comparison"

/*** From here down, only meaningful if single year ***/
VAR _CurVal =
    COALESCE ( [Selected Outcome Total1], 0 )

VAR _CurOutcomeIncRows =
    CALCULATE (
        COUNTROWS (
            FILTER ( 'F Incident', COALESCE ( [Selected Outcome Total1], 0 ) > 0 )
        ),
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = _Y
    )

VAR _PrevOutcomeIncRows =
    CALCULATE (
        COUNTROWS (
            FILTER ( 'F Incident', COALESCE ( [Selected Outcome Total1], 0 ) > 0 )
        ),
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = _Y - 1
    )

VAR _FirstOutcomeIncidentYear =
    MINX (
        FILTER (
            ALL ( 'D Date'[Year] ),
            NOT ISBLANK ( 'D Date'[Year] )
                && CALCULATE (
                    COUNTROWS (
                        FILTER ( 'F Incident', COALESCE ( [Selected Outcome Total1], 0 ) > 0 )
                    )
                ) > 0
        ),
        'D Date'[Year]
    )

VAR _PrevVal =
    COALESCE (
        CALCULATE (
            [Selected Outcome Total1],
            REMOVEFILTERS ( 'D Date'[Year] ),
            'D Date'[Year] = _Y - 1
        ),
        0
    )

VAR _YoY_Pct =
    IF ( _PrevVal > 0, DIVIDE ( _CurVal - _PrevVal, _PrevVal ), BLANK () )

VAR _BracketLabel =
    SWITCH (
        TRUE (),
        _PrevOutcomeIncRows = 0, BLANK (),
        _PrevVal > 0 && _CurVal > _PrevVal, "(" & UNICHAR ( 9650 ) & " " & FORMAT ( _YoY_Pct, "0%" ) & ")",
        _PrevVal > 0 && _CurVal < _PrevVal, "(" & UNICHAR ( 9660 ) & " " & FORMAT ( ABS ( _YoY_Pct ), "0%" ) & ")",
        _PrevVal > 0 && _CurVal = _PrevVal, "(no change)",
        _PrevVal = 0 && _CurVal > 0, "(" & UNICHAR ( 9650 ) & " new)",
        _PrevVal = 0 && _CurVal = 0, "(no change)"
    )

VAR _Avg3 =
    [Outcome (3-year avg)]

VAR _AvgText =
    IF (
        NOT ISBLANK ( _Avg3 ),
        " and the 3-year average (" & FORMAT ( _Y - 3, "0" ) & "–" & FORMAT ( _Y - 1, "0" ) & ")",
        BLANK ()
    )

VAR _NoCurrentText =
    IF ( _O = "Incidents", " (no incidents recorded)", " (no data recorded)" )

VAR _NoPrevYearText =
    IF ( _O = "Incidents", " (no previous-year incidents recorded)", " (no previous-year cases recorded)" )

VAR _MainText =
    SWITCH (
        TRUE (),

        NOT _isSingleYear,
            _PromptText,

        _CurOutcomeIncRows = 0,
            FORMAT ( _Y, "0" ) & _NoCurrentText,

        _Y = _FirstOutcomeIncidentYear,
            FORMAT ( _Y, "0" ) & " (first year recorded)",

        _PrevOutcomeIncRows = 0,
            FORMAT ( _Y, "0" ) & _NoPrevYearText,

        TRUE,
            FORMAT ( _Y, "0" )
                & " compared with " & FORMAT ( _Y - 1, "0" )
                & IF ( NOT ISBLANK ( _BracketLabel ), " " & _BracketLabel, BLANK () )
                & _AvgText
    )

RETURN
    _TitlePrefix & _MainText
```

</details>

<details>
<summary><strong>Outcome Label (Title)</strong></summary>

```DAX

VAR _o = SELECTEDVALUE(Outcome[Outcome], "All")
RETURN
SWITCH (
    _o,
    "All", "Aid workers attacked",
    "Killed", "Aid workers killed",
    "Wounded", "Aid workers wounded",
    "Kidnapped", "Aid workers kidnapped",
    "Detained", "Aid workers detained",
    "Aid workers attacked"
)
```

</details>

<details>
<summary><strong>Outcome Phrase (Title)</strong></summary>

```DAX

VAR _o = SELECTEDVALUE(Outcome[Outcome], "All")
RETURN
SWITCH(
    _o,
    "All", "aid workers affected",
    "Killed", "aid workers killed",
    "Wounded", "aid workers wounded",
    "Kidnapped", "aid workers kidnapped",
    "Detained", "aid workers detained",
    "aid workers affected"
)
```

</details>

<details>
<summary><strong>Prev Year</strong></summary>

```DAX

[Selected Year] - 1

```

</details>

<details>
<summary><strong>Previous Year (Outcome)</strong></summary>

```DAX

VAR Y = SELECTEDVALUE('D Date'[Year])
RETURN
IF(
    NOT ISBLANK(Y),
    CALCULATE(
        [Selected Outcome Total1],
        REMOVEFILTERS('D Date'[Year]),
        'D Date'[Year] = Y - 1
    )
)

```

</details>

<details>
<summary><strong>Selected Attack Dimension</strong></summary>

```DAX

SELECTEDVALUE(
    'Attack Dimension Parameter'[Attack Dimension Parameter Fields]
)
```

</details>

<details>
<summary><strong>Selected Category</strong></summary>

```DAX

SWITCH(
    TRUE(),
    ISINSCOPE('D Source'[Source]) && HASONEVALUE('D Source'[Source]), "Source",
    ISINSCOPE('D Verification Status'[Verification Status]) && HASONEVALUE('D Verification Status'[Verification Status]), "Verification Status",
    ISINSCOPE('D Perpetrator Type'[Perpetrator Type]) && HASONEVALUE('D Perpetrator Type'[Perpetrator Type]), "Perpetrator Type",
    ISINSCOPE('D Incident Setting'[Incident Setting]) && HASONEVALUE('D Incident Setting'[Incident Setting]), "Incident Setting",
    ISINSCOPE('D Attack Means'[Attack Means]) && HASONEVALUE('D Attack Means'[Attack Means]), "Attack Means",
    ISINSCOPE('D Motive'[Motive]) && HASONEVALUE('D Motive'[Motive]), "Motive",
    ISINSCOPE('D Attack Context'[Attack Context]) && HASONEVALUE('D Attack Context'[Attack Context]), "Attack Context",
    BLANK()
)

```

</details>

<details>
<summary><strong>Selected Definition</strong></summary>

```DAX

SWITCH(
    TRUE(),

    ISINSCOPE('D Source'[Source]) && HASONEVALUE('D Source'[Source]),
        LOOKUPVALUE(
            Source_Definitions[Definition],
            Source_Definitions[Source],
            SELECTEDVALUE('D Source'[Source])
        ),

    ISINSCOPE('D Verification Status'[Verification Status]) && HASONEVALUE('D Verification Status'[Verification Status]),
        LOOKUPVALUE(
            VerificationStatus_Definitions[Definition],
            VerificationStatus_Definitions[Verification Status],
            SELECTEDVALUE('D Verification Status'[Verification Status])
        ),

    ISINSCOPE('D Perpetrator Type'[Perpetrator Type]) && HASONEVALUE('D Perpetrator Type'[Perpetrator Type]),
        LOOKUPVALUE(
            PerpetratorType_Definitions[Definition],
            PerpetratorType_Definitions[Perpetrator Type],
            SELECTEDVALUE('D Perpetrator Type'[Perpetrator Type])
        ),

    ISINSCOPE('D Incident Setting'[Incident Setting]) && HASONEVALUE('D Incident Setting'[Incident Setting]),
        LOOKUPVALUE(
            IncidentSetting_Definitions[Definition],
            IncidentSetting_Definitions[Incident Setting],
            SELECTEDVALUE('D Incident Setting'[Incident Setting])
        ),

    ISINSCOPE('D Attack Means'[Attack Means]) && HASONEVALUE('D Attack Means'[Attack Means]),
        LOOKUPVALUE(
            AttackMeans_Definitions[Definition],
            AttackMeans_Definitions[Attack Means],
            SELECTEDVALUE('D Attack Means'[Attack Means])
        ),

    ISINSCOPE('D Motive'[Motive]) && HASONEVALUE('D Motive'[Motive]),
        LOOKUPVALUE(
            Motive_Definitions[Definition],
            Motive_Definitions[Motive],
            SELECTEDVALUE('D Motive'[Motive])
        ),

    ISINSCOPE('D Attack Context'[Attack Context]) && HASONEVALUE('D Attack Context'[Attack Context]),
        LOOKUPVALUE(
            AttackContext_Definitions[Definition],
            AttackContext_Definitions[Attack Context],
            SELECTEDVALUE('D Attack Context'[Attack Context])
        ),

    BLANK()
)

```

</details>

<details>
<summary><strong>Selected Dictionary Definition</strong></summary>

```DAX

VAR _FieldKey = SELECTEDVALUE(DictionaryFields[FieldKey])
RETURN
CALCULATE(
    MAX(DictionaryDetails[Definition]),
    DictionaryDetails[FieldKey] = _FieldKey
)

```

</details>

<details>
<summary><strong>Selected Dictionary Field</strong></summary>

```DAX

SELECTEDVALUE(DictionaryFields[FieldName], "Select a field")

```

</details>

<details>
<summary><strong>Selected Outcome Color</strong></summary>

```DAX

SELECTEDVALUE(Outcome[Outcome Color], "#333333")

```

</details>

<details>
<summary><strong>Selected Outcome Total</strong></summary>

```DAX

VAR O = SELECTEDVALUE ( Outcome[Outcome], "Killed" )
RETURN
SWITCH(
    O,
    "Killed",    [Total Killed],
    "Wounded",   [Total Wounded],
    "Kidnapped", [Total Kidnapped],
    BLANK()
)

```

</details>

<details>
<summary><strong>Selected Outcome Total — Internationals</strong></summary>

```DAX

VAR o = SELECTEDVALUE(Outcome[Outcome], "All")
RETURN
SWITCH(
    o,
    "Killed",     SUM('F Incident'[Internationals Killed]),
    "Wounded",    SUM('F Incident'[Internationals Wounded]),
    "Kidnapped",  SUM('F Incident'[Internationals Kidnapped]),
    "Detained",   SUM('F Incident'[Internationals Detained]),
    -- All:
    SUM('F Incident'[Internationals Killed])
    + SUM('F Incident'[Internationals Wounded])
    + SUM('F Incident'[Internationals Kidnapped])
    + SUM('F Incident'[Internationals Detained])
)

```

</details>

<details>
<summary><strong>Selected Outcome Total — Internationals (Plot)</strong></summary>

```DAX

VAR o =
    SELECTEDVALUE ( Outcome[Outcome], "All" )
VAR v =
    SWITCH (
        o,
        "Killed",     SUM ( 'F Incident'[Internationals Killed] ),
        "Wounded",    SUM ( 'F Incident'[Internationals Wounded] ),
        "Kidnapped",  SUM ( 'F Incident'[Internationals Kidnapped] ),
        "Detained",   SUM ( 'F Incident'[Internationals Detained] ),
        SUM ( 'F Incident'[Internationals Killed] )
            + SUM ( 'F Incident'[Internationals Wounded] )
            + SUM ( 'F Incident'[Internationals Kidnapped] )
            + SUM ( 'F Incident'[Internationals Detained] )
    )
RETURN
    IF ( v = 0 || ISBLANK ( v ), BLANK (), v )
```

</details>

<details>
<summary><strong>Selected Outcome Total — Nat+Int</strong></summary>

```DAX

[Selected Outcome Total — Nationals] + [Selected Outcome Total — Internationals]

```

</details>

<details>
<summary><strong>Selected Outcome Total — Nat+Int1</strong></summary>

```DAX

COALESCE ( [Selected Outcome Total — Nationals], 0 )
+ COALESCE ( [Selected Outcome Total — Internationals], 0 )
```

</details>

<details>
<summary><strong>Selected Outcome Total — Nationals</strong></summary>

```DAX

VAR o = SELECTEDVALUE(Outcome[Outcome], "All")
RETURN
SWITCH(
    o,
    "Killed",     SUM('F Incident'[Nationals Killed]),
    "Wounded",    SUM('F Incident'[Nationals Wounded]),
    "Kidnapped",  SUM('F Incident'[Nationals Kidnapped]),
    "Detained",   SUM('F Incident'[Nationals Detained]),
    -- All:
    SUM('F Incident'[Nationals Killed])
    + SUM('F Incident'[Nationals Wounded])
    + SUM('F Incident'[Nationals Kidnapped])
    + SUM('F Incident'[Nationals Detained])
)

```

</details>

<details>
<summary><strong>Selected Outcome Total — Nationals (Plot)</strong></summary>

```DAX

VAR o =
    SELECTEDVALUE ( Outcome[Outcome], "All" )
VAR v =
    SWITCH (
        o,
        "Killed",     SUM ( 'F Incident'[Nationals Killed] ),
        "Wounded",    SUM ( 'F Incident'[Nationals Wounded] ),
        "Kidnapped",  SUM ( 'F Incident'[Nationals Kidnapped] ),
        "Detained",   SUM ( 'F Incident'[Nationals Detained] ),
        SUM ( 'F Incident'[Nationals Killed] )
            + SUM ( 'F Incident'[Nationals Wounded] )
            + SUM ( 'F Incident'[Nationals Kidnapped] )
            + SUM ( 'F Incident'[Nationals Detained] )
    )
RETURN
    IF ( v = 0 || ISBLANK ( v ), BLANK (), v )
```

</details>

<details>
<summary><strong>Selected Outcome Total (Gender)</strong></summary>

```DAX

SWITCH(
    SELECTEDVALUE('D Gender'[Gender]),
    "Female", [Gender Female Total],
    "Male", [Gender Male Total],
    "Unknown", [Gender Unknown Total]
)

```

</details>

<details>
<summary><strong>Selected Outcome Total1</strong></summary>

```DAX

SWITCH(
    SELECTEDVALUE(Outcome[Outcome], "All"),
    "Incidents", COALESCE([Total Incidents], 0),
    "Killed",    COALESCE([Total Killed], 0),
    "Wounded",   COALESCE([Total Wounded], 0),
    "Kidnapped", COALESCE([Total Kidnapped], 0),
    "Detained",  COALESCE([Total Detained], 0),
    "All",
        COALESCE([Total Killed], 0)
        + COALESCE([Total Wounded], 0)
        + COALESCE([Total Kidnapped], 0)
        + COALESCE([Total Detained], 0),
    BLANK()
)

```

</details>

<details>
<summary><strong>Selected Term</strong></summary>

```DAX

SWITCH(
    TRUE(),
    ISINSCOPE('D Source'[Source]) && HASONEVALUE('D Source'[Source]),
        SELECTEDVALUE('D Source'[Source]),

    ISINSCOPE('D Verification Status'[Verification Status]) && HASONEVALUE('D Verification Status'[Verification Status]),
        SELECTEDVALUE('D Verification Status'[Verification Status]),

    ISINSCOPE('D Perpetrator Type'[Perpetrator Type]) && HASONEVALUE('D Perpetrator Type'[Perpetrator Type]),
        SELECTEDVALUE('D Perpetrator Type'[Perpetrator Type]),

    ISINSCOPE('D Incident Setting'[Incident Setting]) && HASONEVALUE('D Incident Setting'[Incident Setting]),
        SELECTEDVALUE('D Incident Setting'[Incident Setting]),

    ISINSCOPE('D Attack Means'[Attack Means]) && HASONEVALUE('D Attack Means'[Attack Means]),
        SELECTEDVALUE('D Attack Means'[Attack Means]),

    ISINSCOPE('D Motive'[Motive]) && HASONEVALUE('D Motive'[Motive]),
        SELECTEDVALUE('D Motive'[Motive]),

    ISINSCOPE('D Attack Context'[Attack Context]) && HASONEVALUE('D Attack Context'[Attack Context]),
        SELECTEDVALUE('D Attack Context'[Attack Context]),

    BLANK()
)

```

</details>

<details>
<summary><strong>Selected Year</strong></summary>

```DAX

COALESCE( SELECTEDVALUE('D Date'[Year]), MAX('D Date'[Year]) )
```

</details>

<details>
<summary><strong>Selected Year (Outcome)</strong></summary>

```DAX

VAR Y = SELECTEDVALUE('D Date'[Year])
RETURN
IF(
    NOT ISBLANK(Y),
    CALCULATE(
        [Selected Outcome Total1],
        KEEPFILTERS('D Date'[Year] = Y)
    )
)

```

</details>

<details>
<summary><strong>Show Comparison</strong></summary>

```DAX

IF( HASONEVALUE('D Date'[Year]), 1, 0 )
```

</details>

<details>
<summary><strong>Show Country Message</strong></summary>

```DAX

IF( ISFILTERED('D Location'[Country]), 1, 0 )

```

</details>

<details>
<summary><strong>Show Top 3 Countries</strong></summary>

```DAX

IF(
    [Country Rank (Selected Outcome)] <= 3,
    1,
    0
)

```

</details>

<details>
<summary><strong>Show Top Countries Chart</strong></summary>

```DAX

IF( [All Countries Selected] = 1, 1, 0 )
```

</details>

<details>
<summary><strong>Show Top Countries Visual</strong></summary>

```DAX

VAR SelCountries = COUNTROWS( ALLSELECTED('D Location'[Country]) )
VAR AllCountries = COUNTROWS( ALL('D Location'[Country]) )
RETURN
IF( SelCountries = 0 || SelCountries = AllCountries, 1, 0 )

```

</details>

<details>
<summary><strong>SortValue_UnknownLast_AttackPage</strong></summary>

```DAX

VAR Label =
    COALESCE(
        SELECTEDVALUE('D Attack Means'[Attack Means]),
        SELECTEDVALUE('D Attack Context'[Attack Context]),
        SELECTEDVALUE('DimPerpetratorGroup'[Perpetrator Group]),
        SELECTEDVALUE('D Perpetrator Type'[Perpetrator Type]),
        SELECTEDVALUE('D Incident Setting'[Incident Setting]),
        SELECTEDVALUE('D Motive'[Motive])
    )
VAR IsUnknown =
    Label = "Unknown"
RETURN
IF( IsUnknown, -1e15, [Selected Outcome Total1] )

```

</details>

<details>
<summary><strong>Title — <Visual Name></strong></summary>

```DAX

VAR V =
    CALCULATE(
        COALESCE( SUM('F Incident'[<ReplaceWithColumn>]), 0 )
    )

VAR C =
    SELECTEDVALUE('D Location'[Country])

VAR CountryText =
    IF(
        HASONEVALUE('D Location'[Country]),
        " in " & C,
        " globally"
    )

VAR MinY = MIN('D Date'[Year])
VAR MaxY = MAX('D Date'[Year])
VAR OneYear = HASONEVALUE('D Date'[Year])

VAR YearText =
    IF(
        ISFILTERED('D Date'[Year]),
        " in " &
        IF(
            OneYear,
            FORMAT(MinY,"0"),
            FORMAT(MinY,"0") & "–" & FORMAT(MaxY,"0")
        ),
        " across all years"
    )

RETURN
IF(
    V = 0,
    "No incidents recorded" & CountryText & YearText,
    "<Base title text>" & CountryText & YearText
)

```

</details>

<details>
<summary><strong>Title — Attack Dimension (Dynamic)</strong></summary>

```DAX

VAR DimName =
    SELECTEDVALUE ( 'Attack Dimension Parameter'[Attack Dimension Parameter Order] )

VAR DimText =
    SWITCH (
        DimName,
        0, "Attack means",
        1, "Attack context",
        2, "Incident setting",
        "Attack dimension"
    )

VAR V =
    CALCULATE (
        COALESCE ( [Selected Outcome Total1], 0 ),
        REMOVEFILTERS ( 'Attack Dimension Parameter' )
    )

VAR C = SELECTEDVALUE ( 'D Location'[Country] )

VAR MinY = MIN ( 'D Date'[Year] )
VAR MaxY = MAX ( 'D Date'[Year] )
VAR OneYear = HASONEVALUE ( 'D Date'[Year] )

VAR CountryText =
    IF (
        HASONEVALUE ( 'D Location'[Country] ),
        " in " & C,
        " globally"
    )

VAR YearText =
    IF (
        ISFILTERED ( 'D Date'[Year] ),
        " in "
            & IF (
                OneYear,
                FORMAT ( MinY, "0" ),
                FORMAT ( MinY, "0" ) & "-" & FORMAT ( MaxY, "0" )
            ),
        " across all reported years"
    )

VAR MetricSelected =
    SELECTEDVALUE ( Outcome[Outcome], "All" )

VAR MetricPhrase =
    SWITCH (
        MetricSelected,
        "Incidents", "incidents",
        "All", "affected aid workers",
        "Killed", "aid workers killed",
        "Wounded", "aid workers wounded",
        "Kidnapped", "aid workers kidnapped",
        "Detained", "aid workers detained",
        "affected aid workers"
    )

VAR TitleText =
    DimText & " breakdown - " & MetricPhrase & CountryText & YearText

VAR NoDataText =
    DimText & " breakdown - no " & MetricPhrase & CountryText & YearText

RETURN
    IF ( V > 0, TitleText, NoDataText )
```

</details>

<details>
<summary><strong>Title — Gender Distribution</strong></summary>

```DAX

VAR V =
    COALESCE (
        SUM ( 'F Incident'[Gender Female] )
            + SUM ( 'F Incident'[Gender Male] )
            + SUM ( 'F Incident'[Gender Unknown] ),
        0
    )

VAR C = SELECTEDVALUE ( 'D Location'[Country] )

VAR CountryText =
    IF (
        HASONEVALUE ( 'D Location'[Country] ),
        " in " & C,
        " worldwide"
    )

VAR MinY = MIN ( 'D Date'[Year] )
VAR MaxY = MAX ( 'D Date'[Year] )
VAR OneYear = HASONEVALUE ( 'D Date'[Year] )

VAR YearText =
    IF (
        ISFILTERED ( 'D Date'[Year] ),
        " in "
            & IF (
                OneYear,
                FORMAT ( MinY, "0" ),
                FORMAT ( MinY, "0" ) & "–" & FORMAT ( MaxY, "0" )
            ),
        " across all reported years"
    )

VAR OutcomeSelected =
    SELECTEDVALUE ( Outcome[Outcome], "All" )

VAR AllOutcomesNote =
    IF (
        OutcomeSelected = "All",
        "",
        ""
    )

RETURN
IF (
    V = 0,
    "No incidents recorded" & CountryText & YearText & AllOutcomesNote,
    "Gender distribution of aid workers affected" & CountryText & YearText & AllOutcomesNote)
```

</details>

<details>
<summary><strong>Title — Heatmap (Dynamic)</strong></summary>

```DAX

VAR DimName =
    SELECTEDVALUE (
        'Perpetrator Dimension Parameter'[Perpetrator Dimension Parameter Order]
    )

VAR DimText =
    SWITCH (
        DimName,
        0, "perpetrator group",
        1, "motive",
        "Perpetrator"
    )

VAR MetricSelected =
    SELECTEDVALUE ( Outcome[Outcome], "All" )

VAR MetricPhrase =
    SWITCH (
        MetricSelected,
        "Incidents", "incidents",
        "All", "affected aid workers",
        "Killed", "aid workers killed",
        "Wounded", "aid workers wounded",
        "Kidnapped", "aid workers kidnapped",
        "Detained", "aid workers detained",
        "affected aid workers"
    )

VAR MinY = MIN ( 'D Date'[Year] )
VAR MaxY = MAX ( 'D Date'[Year] )
VAR OneYear = HASONEVALUE ( 'D Date'[Year] )

VAR YearText =
    IF (
        ISFILTERED ( 'D Date'[Year] ),
        " in "
            & IF (
                OneYear,
                FORMAT ( MinY, "0" ),
                FORMAT ( MinY, "0" ) & "–" & FORMAT ( MaxY, "0" )
            ),
        " across all reported years"
    )

VAR CountryNote =
    IF (
        ISFILTERED ( 'D Location'[Country] ),
        " (not affected by country selection)",
        ""
    )

VAR V =
    CALCULATE (
        COALESCE ( [Selected Outcome Total1], 0 ),
        REMOVEFILTERS ( 'Perpetrator Dimension Parameter' ),
        REMOVEFILTERS ( 'D Location'[Country] )
    )

VAR TitleText =
    "Top countries by " & MetricPhrase
        & " - " & DimText & " breakdown globally"
        & YearText
        & CountryNote

VAR NoDataText =
    "Top countries by " & MetricPhrase
        & " - no " & MetricPhrase & " globally"
        & YearText
        & CountryNote

RETURN
    IF ( V > 0, TitleText, NoDataText )
```

</details>

<details>
<summary><strong>Title — Motive (Dynamic)</strong></summary>

```DAX

VAR V =
    COALESCE ( [Selected Outcome Total1], 0 )

VAR C = SELECTEDVALUE ( 'D Location'[Country] )

VAR MinY = MIN ( 'D Date'[Year] )
VAR MaxY = MAX ( 'D Date'[Year] )
VAR OneYear = HASONEVALUE ( 'D Date'[Year] )

VAR CountryText =
    IF (
        HASONEVALUE ( 'D Location'[Country] ),
        " in " & C,
        " globally"
    )

VAR YearText =
    IF (
        ISFILTERED ( 'D Date'[Year] ),
        " in "
            & IF (
                OneYear,
                FORMAT ( MinY, "0" ),
                FORMAT ( MinY, "0" ) & "–" & FORMAT ( MaxY, "0" )
            ),
        " across all reported years"
    )

VAR MetricSelected =
    SELECTEDVALUE ( Outcome[Outcome], "All" )

VAR MetricPhrase =
    SWITCH (
        MetricSelected,
        "Incidents", "incidents",
        "All", "affected aid workers",
        "Killed", "aid workers killed",
        "Wounded", "aid workers wounded",
        "Kidnapped", "aid workers kidnapped",
        "Detained", "aid workers detained",
        "affected aid workers"
    )

VAR TitleText =
    "Motive breakdown - " & MetricPhrase & CountryText & YearText

VAR NoDataText =
    "Motive breakdown - no " & MetricPhrase & CountryText & YearText

RETURN
    IF ( V > 0, TitleText, NoDataText )
```

</details>

<details>
<summary><strong>Title — Nat vs Int by Outcome</strong></summary>

```DAX

VAR V =
    COALESCE (
        SUM ( 'F Incident'[Nationals Killed] )
            + SUM ( 'F Incident'[Nationals Wounded] )
            + SUM ( 'F Incident'[Nationals Kidnapped] )
            + SUM ( 'F Incident'[Nationals Detained] )
            + SUM ( 'F Incident'[Internationals Killed] )
            + SUM ( 'F Incident'[Internationals Wounded] )
            + SUM ( 'F Incident'[Internationals Kidnapped] )
            + SUM ( 'F Incident'[Internationals Detained] ),
        0
    )

VAR C = SELECTEDVALUE ( 'D Location'[Country] )

VAR CountryText =
    IF (
        HASONEVALUE ( 'D Location'[Country] ),
        " in " & C,
        " worldwide"
    )

VAR MinY = MIN ( 'D Date'[Year] )
VAR MaxY = MAX ( 'D Date'[Year] )
VAR OneYear = HASONEVALUE ( 'D Date'[Year] )

VAR YearText =
    IF (
        ISFILTERED ( 'D Date'[Year] ),
        " in "
            & IF (
                OneYear,
                FORMAT ( MinY, "0" ),
                FORMAT ( MinY, "0" ) & "–" & FORMAT ( MaxY, "0" )
            ),
        " across all reported years"
    )

VAR OutcomeSelected =
    SELECTEDVALUE ( Outcome[Outcome], "All" )

VAR AllOutcomesNote =
    IF ( OutcomeSelected = "All", "", "" )

RETURN
IF (
    V = 0,
    "No incidents recorded" & CountryText & YearText & AllOutcomesNote,
    "Nationality group of aid workers by attack outcome"
        & CountryText
        & YearText
        & AllOutcomesNote
)
```

</details>

<details>
<summary><strong>Title — Nat vs Int Trend</strong></summary>

```DAX

VAR V =
    CALCULATE (
        COALESCE ( [Selected Outcome Total1], 0 ),
        REMOVEFILTERS ( 'D Date'[Year] )
    )

VAR C =
    SELECTEDVALUE ( 'D Location'[Country] )

VAR CountryText =
    IF (
        HASONEVALUE ( 'D Location'[Country] ),
        " in " & C,
        " worldwide"
    )

VAR OutcomeSelected =
    SELECTEDVALUE ( Outcome[Outcome], "All" )

VAR OutcomeWord =
    SWITCH (
        OutcomeSelected,
        "All", "aid workers affected",
        "Killed", "aid workers killed",
        "Wounded", "aid workers wounded",
        "Kidnapped", "aid workers kidnapped",
        "Detained", "aid workers detained",
        "aid workers affected"
    )

VAR OutcomeVerbPhrase =
    SUBSTITUTE ( OutcomeWord, "aid workers ", "" )

VAR AllOutcomesNote =
    IF (
        OutcomeSelected = "All",
        " - all attack outcomes",
        ""
    )

VAR NoIncidentsNote =
    IF (
        V = 0,
        " - no incidents recorded",
        ""
    )

VAR YearFiltered =
    ISFILTERED ( 'D Date'[Year] )
        || ISCROSSFILTERED ( 'D Date'[Year] )

VAR YearNote =
    IF (
        YearFiltered,
        " - not impacted by year selection",
        ""
    )

VAR BaseTitle =
    "Nationality group of aid workers "
        & OutcomeVerbPhrase
        & CountryText
        & " across all reported years"
        & AllOutcomesNote

RETURN
    BaseTitle & " (trend)" & NoIncidentsNote & YearNote
```

</details>

<details>
<summary><strong>Title — Nationals vs Internationals</strong></summary>

```DAX

VAR V =
    COALESCE ( [Selected Outcome Total1], 0 )

VAR C = SELECTEDVALUE ( 'D Location'[Country] )
VAR MinY = MIN ( 'D Date'[Year] )
VAR MaxY = MAX ( 'D Date'[Year] )
VAR OneYear = HASONEVALUE ( 'D Date'[Year] )

VAR CountryText =
    IF ( HASONEVALUE ( 'D Location'[Country] ), " in " & C, " worldwide" )

VAR YearText =
    IF (
        ISFILTERED ( 'D Date'[Year] ),
        " in "
            & IF (
                OneYear,
                FORMAT ( MinY, "0" ),
                FORMAT ( MinY, "0" ) & "–" & FORMAT ( MaxY, "0" )
            ),
        " across all reported years"
    )

VAR OutcomeSelected =
    SELECTEDVALUE ( Outcome[Outcome], "All" )

VAR OutcomeWord =
    SWITCH (
        OutcomeSelected,
        "All", "aid workers affected",
        "Killed", "aid workers killed",
        "Wounded", "aid workers wounded",
        "Kidnapped", "aid workers kidnapped",
        "Detained", "aid workers detained",
        "aid workers affected"
    )

VAR OutcomeVerbPhrase =
    SUBSTITUTE ( OutcomeWord, "aid workers ", "" )

VAR AllOutcomesNote =
    IF ( OutcomeSelected = "All", " - all attack outcomes", "" )

VAR NoIncidentsNote =
    IF ( V = 0, " - no incidents recorded", "" )

VAR BaseTitle =
    "Nationality group of aid workers "
        & OutcomeVerbPhrase
        & CountryText
        & YearText
        & AllOutcomesNote

RETURN
    BaseTitle & " (summary)" & NoIncidentsNote
```

</details>

<details>
<summary><strong>Title — Organisation Type Distribution</strong></summary>

```DAX

VAR V =
    COALESCE (
        CALCULATE (
            [Total Victims IFRC ICRC]
                + [Total Victims INGO]
                + [Total Victims NNGO]
                + [Total Victims NRCS]
                + [Total Victims Other Org],
            REMOVEFILTERS ( Outcome[Outcome] )
        ),
        0
    )

VAR C = SELECTEDVALUE ( 'D Location'[Country] )

VAR CountryText =
    IF (
        HASONEVALUE ( 'D Location'[Country] ),
        " in " & C,
        " worldwide"
    )

VAR MinY = MIN ( 'D Date'[Year] )
VAR MaxY = MAX ( 'D Date'[Year] )
VAR OneYear = HASONEVALUE ( 'D Date'[Year] )

VAR YearText =
    IF (
        ISFILTERED ( 'D Date'[Year] ),
        " in "
            & IF (
                OneYear,
                FORMAT ( MinY, "0" ),
                FORMAT ( MinY, "0" ) & "–" & FORMAT ( MaxY, "0" )
            ),
        " across all reported years"
    )

VAR OutcomeSelected =
    SELECTEDVALUE ( Outcome[Outcome], "All" )

VAR AllOutcomesNote =
    IF ( OutcomeSelected = "All", "", "" )

RETURN
IF (
    V = 0,
    "No incidents recorded" & CountryText & YearText & AllOutcomesNote,
    "Organisation type of aid workers affected" & CountryText & YearText & AllOutcomesNote)
```

</details>

<details>
<summary><strong>Title — Perpetrator Type (Dynamic)</strong></summary>

```DAX

VAR V =
    COALESCE ( [Selected Outcome Total1], 0 )

VAR C = SELECTEDVALUE ( 'D Location'[Country] )

VAR MinY = MIN ( 'D Date'[Year] )
VAR MaxY = MAX ( 'D Date'[Year] )
VAR OneYear = HASONEVALUE ( 'D Date'[Year] )

VAR CountryText =
    IF (
        HASONEVALUE ( 'D Location'[Country] ),
        " in " & C,
        " globally"
    )

VAR YearText =
    IF (
        ISFILTERED ( 'D Date'[Year] ),
        " in "
            & IF (
                OneYear,
                FORMAT ( MinY, "0" ),
                FORMAT ( MinY, "0" ) & "–" & FORMAT ( MaxY, "0" )
            ),
        " across all reported years"
    )

VAR MetricSelected =
    SELECTEDVALUE ( Outcome[Outcome], "All" )

VAR MetricPhrase =
    SWITCH (
        MetricSelected,
        "Incidents", "incidents",
        "All", "affected aid workers",
        "Killed", "aid workers killed",
        "Wounded", "aid workers wounded",
        "Kidnapped", "aid workers kidnapped",
        "Detained", "aid workers detained",
        "affected aid workers"
    )

VAR TitleText =
    "Perpetrator type breakdown - " & MetricPhrase & CountryText & YearText

VAR NoDataText =
    "Perpetrator type breakdown - no " & MetricPhrase & CountryText & YearText

RETURN
    IF ( V > 0, TitleText, NoDataText )
```

</details>

<details>
<summary><strong>Title — Top Countries (Nat vs Int)</strong></summary>

```DAX

VAR YearText =
    IF (
        HASONEVALUE ( 'D Date'[Year] ),
        SELECTEDVALUE ( 'D Date'[Year] ),
        "all reported years"
    )

VAR C =
    SELECTEDVALUE ( 'D Location'[Country] )

VAR CountryText =
    IF (
        HASONEVALUE ( 'D Location'[Country] ),
        " in " & C,
        ""
    )

VAR OutcomeSelected =
    SELECTEDVALUE ( Outcome[Outcome], "All" )

VAR OutcomeNoun =
    SWITCH (
        OutcomeSelected,
        "All", "attacks",
        "Killed", "killings",
        "Wounded", "woundings",
        "Kidnapped", "kidnappings",
        "Detained", "detentions",
        "attacks"
    )

VAR AllOutcomesNote =
    IF ( OutcomeSelected = "All", " (all outcomes)", "" )

RETURN
    "Top countries by " &
    OutcomeNoun &
    " - nationality group of aid workers affected" &
    CountryText &
    " in " & YearText &
    AllOutcomesNote
```

</details>

<details>
<summary><strong>Tooltip - Motive TermDefinition</strong></summary>

```DAX

VAR _Motive =
    SELECTEDVALUE ( 'D Motive'[Motive] )   -- change table name if your funnel uses a different one
RETURN
LOOKUPVALUE(
    Motive_Definitions[TermDefinition],
    Motive_Definitions[Motive], _Motive
)
```

</details>

<details>
<summary><strong>Tooltip - Organisation TermDefinition</strong></summary>

```DAX

VAR _Org =
    SELECTEDVALUE ( 'D Org Type'[Org Type])   -- adjust table/column name if needed
RETURN
LOOKUPVALUE(
    OrganisationAbbreviation_Definitions[TermDefinition],
    OrganisationAbbreviation_Definitions[Abbreviation], _Org
)
```

</details>

<details>
<summary><strong>Tooltip - Perpetrator Definition</strong></summary>

```DAX

VAR _Type =
    SELECTEDVALUE ( 'D Perpetrator Type'[Perpetrator Type] )

VAR _Group =
    SELECTEDVALUE ( 'DimPerpetratorGroup'[Perpetrator Group] )

RETURN
IF(
    NOT ISBLANK ( _Type ),
    LOOKUPVALUE(
        PerpetratorType_Definitions[Definition],
        PerpetratorType_Definitions[Perpetrator Type], _Type
    ),
    LOOKUPVALUE(
        PerpetratorGroup_Definitions[Definition],
        PerpetratorGroup_Definitions[Perpetrator Group], _Group
    )
)
```

</details>

<details>
<summary><strong>Tooltip - Perpetrator Term + Definition</strong></summary>

```DAX

VAR _Type =
    SELECTEDVALUE ( 'D Perpetrator Type'[Perpetrator Type] )
VAR _Group =
    SELECTEDVALUE ( 'DimPerpetratorGroup'[Perpetrator Group] )

VAR _Term =
    COALESCE ( _Type, _Group )

VAR _Definition =
    IF(
        NOT ISBLANK ( _Type ),
        LOOKUPVALUE(
            PerpetratorType_Definitions[Definition],
            PerpetratorType_Definitions[Perpetrator Type], _Type
        ),
        LOOKUPVALUE(
            PerpetratorGroup_Definitions[Definition],
            PerpetratorGroup_Definitions[Perpetrator Group], _Group
        )
    )
RETURN
IF(
    NOT ISBLANK(_Term) && NOT ISBLANK(_Definition),
    _Term & " - " & _Definition,
    BLANK()
)
```

</details>

<details>
<summary><strong>Tooltip - PerpParam Active</strong></summary>

```DAX

VAR _fields =
    SELECTEDVALUE ( 'Perpetrator Dimension Parameter'[Perpetrator Dimension Parameter Fields] )
RETURN
SWITCH(
    TRUE(),
    NOT ISBLANK(_fields) && CONTAINSSTRING(_fields, "Motive"), "Motive",
    NOT ISBLANK(_fields) && CONTAINSSTRING(_fields, "Perpetrator Type"), "Perpetrator Type",
    NOT ISBLANK(_fields) && CONTAINSSTRING(_fields, "Perpetrator Group"), "Perpetrator Group",
    BLANK()
)

```

</details>

<details>
<summary><strong>Tooltip - PerpParam TermDefinition</strong></summary>

```DAX

VAR _active = [Tooltip - PerpParam Active]

VAR _Type  = SELECTEDVALUE ( 'D Perpetrator Type'[Perpetrator Type] )
VAR _Group = SELECTEDVALUE ( 'DimPerpetratorGroup'[Perpetrator Group] )
VAR _Motive = SELECTEDVALUE ( 'D Motive'[Motive] )

RETURN
SWITCH(
    _active,

    "Motive",
        LOOKUPVALUE(
            Motive_Definitions[TermDefinition],
            Motive_Definitions[Motive], _Motive
        ),

    "Perpetrator Type",
        LOOKUPVALUE(
            PerpetratorType_Definitions[TermDefinition],
            PerpetratorType_Definitions[Perpetrator Type], _Type
        ),

    "Perpetrator Group",
        LOOKUPVALUE(
            PerpetratorGroup_Definitions[TermDefinition],
            PerpetratorGroup_Definitions[Perpetrator Group], _Group
        ),

    BLANK()
)
```

</details>

<details>
<summary><strong>Tooltip Definition</strong></summary>

```DAX
VAR _order = SELECTEDVALUE('Attack Dimension Parameter'[Attack Dimension Parameter Order]) RETURN SWITCH( _order, 0, SELECTEDVALUE(AttackMeans_Definitions[Definition]), 1, SELECTEDVALUE(AttackContext_Definitions[Definition]), 2, SELECTEDVALUE(IncidentSetting_Definitions[Definition]), BLANK() )
```

</details>

<details>
<summary><strong>Top 10 Share %</strong></summary>

```DAX

VAR GlobalTotal =
    CALCULATE(
        [Selected Outcome Total1],
        REMOVEFILTERS('D Location'[Country])
    )
VAR Top10Total =
    SUMX(
        TOPN(
            10,
            ALL('D Location'[Country]),
            CALCULATE([Selected Outcome Total1]),
            DESC,
            'D Location'[Country], ASC
        ),
        CALCULATE([Selected Outcome Total1])
    )
RETURN
DIVIDE( Top10Total, GlobalTotal )
```

</details>

<details>
<summary><strong>Top 5 Countries Group</strong></summary>

```DAX

IF (
    [Country Rank (Incidents)] <= 5,
    "Top 5 countries",
    "All other countries"
)

```

</details>

<details>
<summary><strong>Top Countries Title (Dynamic)</strong></summary>

```DAX

VAR _selOutcome =
    SELECTEDVALUE ( Outcome[Outcome], "All" )

VAR _metricText =
    IF ( _selOutcome = "Incidents", "incidents", [Outcome Phrase (Title)] )

VAR _baseTitle =
    "Countries with the most " & _metricText & " " & [Year Phrase (Title)]

-- Show note only when a single country is selected
VAR _singleCountrySelected =
    HASONEVALUE ( 'D Location'[Country] )

VAR _noteCountry =
    IF ( _singleCountrySelected, " (not impacted by country selection)", "" )

VAR _totalValGlobal =
    CALCULATE (
        COALESCE ( [Selected Outcome Total1], 0 ),
        REMOVEFILTERS ( 'D Location'[Country] )
    )

VAR _noDataSuffix =
    IF ( _selOutcome = "Incidents", " (no incidents recorded)", " (no data recorded)" )

VAR _noteNoData =
    IF ( _totalValGlobal = 0, _noDataSuffix, "" )

RETURN
    _baseTitle & _noteCountry & _noteNoData
```

</details>

<details>
<summary><strong>Top Countries Title (Kidnapped)</strong></summary>

```DAX

IF(
    [All Countries Selected] = 1,
    "Total Kidnapped by Country (Top 3)",
    "Top countries chart is available only when Country = All"
)

```

</details>

<details>
<summary><strong>Top Countries Title (Killed)</strong></summary>

```DAX

IF(
    [All Countries Selected] = 1,
    "Total Killed by Country (Top 3)",
    "Top countries chart available only when Country = All"
)
```

</details>

<details>
<summary><strong>Top Countries Title (Wounded)</strong></summary>

```DAX

IF(
    [All Countries Selected] = 1,
    "Total Wounded by Country (Top 3)",
    "Top countries chart is available only when Country = All"
)

```

</details>

<details>
<summary><strong>Top Countries Value (Display)</strong></summary>

```DAX

IF(
    [All Countries Selected] = 1,
    [Selected Outcome Total1],
    BLANK()
)

```

</details>

<details>
<summary><strong>TopN + Other Value</strong></summary>

```DAX

VAR r = SELECTEDVALUE('TopN Axis'[Rank])

VAR Top10Table =
    TOPN(
        10,
        ALL('D Location'[Country]),
        CALCULATE([Selected Outcome Total1]),
        DESC,
        'D Location'[Country], ASC
    )

VAR GlobalTotal =
    CALCULATE(
        [Selected Outcome Total],
        REMOVEFILTERS('D Location'[Country])
    )

VAR Top10Total =
    SUMX( Top10Table, CALCULATE([Selected Outcome Total1]) )

VAR NthValue =
    IF(
        r <= 10,
        VAR NthCountry =
            MAXX(
                TOPN(
                    1,
                    TOPN( r, Top10Table, CALCULATE([Selected Outcome Total1]), DESC, 'D Location'[Country], ASC ),
                    CALCULATE([Selected Outcome Total1]), ASC,
                    'D Location'[Country], DESC
                ),
                'D Location'[Country]
            )
        RETURN
            CALCULATE( [Selected Outcome Total1], KEEPFILTERS('D Location'[Country] = NthCountry) )
    )

RETURN
IF(
    r <= 10,
    NthValue,
    GlobalTotal - Top10Total
)

```

</details>

<details>
<summary><strong>TopN Country Name</strong></summary>

```DAX

VAR r = SELECTEDVALUE('TopN Axis'[Rank])
VAR Top10 =
    TOPN(
        10,
        ADDCOLUMNS(
            ALL('D Location'[Country]),
            "__v", CALCULATE([Selected Outcome Total1])
        ),
        [__v], DESC,
        'D Location'[Country], ASC
    )
VAR NthRow =
    TOPN(
        1,
        TOPN(
            r,
            Top10,
            [__v], DESC,
            'D Location'[Country], ASC
        ),
        [__v], ASC,
        'D Location'[Country], DESC
    )
RETURN
IF(
    r = 999,
    "Other countries",
    MAXX(NthRow, 'D Location'[Country])
)

```

</details>

<details>
<summary><strong>TopN Cumulative %</strong></summary>

```DAX

VAR r = SELECTEDVALUE('TopN Axis'[Rank])

VAR GlobalTotal =
    CALCULATE(
        [Selected Outcome Total1],
        REMOVEFILTERS('D Location'[Country])
    )

VAR CumTopN =
    SUMX(
        FILTER( ALL('TopN Axis'), 'TopN Axis'[Rank] <= r && 'TopN Axis'[Rank] <= 10 ),
        CALCULATE( [TopN + Other Value] )
    )

RETURN
IF(
    r = 999,
    1,
    DIVIDE( CumTopN, GlobalTotal )
)
```

</details>

<details>
<summary><strong>Total affected</strong></summary>

```DAX

[Total Killed]
+ [Total Wounded]
+ [Total Kidnapped] 
+ [Total Detained] 
```

</details>

<details>
<summary><strong>Total Countries (Year + Country only)</strong></summary>

```DAX

VAR Countries =
    VALUES ( 'D Location'[Country] )
RETURN
COUNTROWS (
    FILTER (
        Countries,
        CALCULATE (
            COALESCE ( [Total Incidents], 0 ),
            REMOVEFILTERS ( Outcome[Outcome] )
        ) > 0
    )
)
```

</details>

<details>
<summary><strong>Total Detained</strong></summary>

```DAX

SUMX(
    'F Incident',
    COALESCE('F Incident'[Nationals Detained],0) +
    COALESCE('F Incident'[Internationals Detained],0)
)
```

</details>

<details>
<summary><strong>Total Detained (2025+)</strong></summary>

```DAX

IF(
    MAX('D Date'[Year]) >= 2025,
    [Total Detained],
    BLANK()
)

```

</details>

<details>
<summary><strong>Total Detained (pre-2025)</strong></summary>

```DAX

IF(
    MAX('D Date'[Year]) <= 2025,
    [Total Detained],
    BLANK()
)

```

</details>

<details>
<summary><strong>Total Incidents</strong></summary>

```DAX

COUNTROWS ( 'F Incident' )
```

</details>

<details>
<summary><strong>Total Incidents (Selected Outcome)</strong></summary>

```DAX

SWITCH(
    SELECTEDVALUE(Outcome[Outcome]),
    "Killed",     [Incidents Killed],
    "Wounded",    [Incidents Wounded],
    "Kidnapped",  [Incidents Kidnapped],
    BLANK()
)

```

</details>

<details>
<summary><strong>Total Internationals Detained</strong></summary>

```DAX

COALESCE( SUM('F Incident'[Internationals Detained]), 0 )
```

</details>

<details>
<summary><strong>Total Internationals Kidnapped</strong></summary>

```DAX

COALESCE( SUM('F Incident'[Internationals Kidnapped]), 0 )
```

</details>

<details>
<summary><strong>Total Internationals Killed</strong></summary>

```DAX

COALESCE( SUM('F Incident'[Internationals Killed]), 0 )
```

</details>

<details>
<summary><strong>Total Internationals Wounded</strong></summary>

```DAX

COALESCE ( SUM ( 'F Incident'[Internationals Wounded] ), 0 )


```

</details>

<details>
<summary><strong>Total Kidnapped</strong></summary>

```DAX

SUMX(
    'F Incident',
    COALESCE('F Incident'[Nationals Kidnapped],0) +
    COALESCE('F Incident'[Internationals Kidnapped],0)
)
```

</details>

<details>
<summary><strong>Total Kidnapped (Top Countries Display))</strong></summary>

```DAX

IF( [All Countries Selected (Slicer)] = 1, [Total Kidnapped], BLANK() )

```

</details>

<details>
<summary><strong>Total Killed</strong></summary>

```DAX

SUMX(
    'F Incident',
    COALESCE('F Incident'[Nationals Killed],0) +
    COALESCE('F Incident'[Internationals Killed],0)
)
```

</details>

<details>
<summary><strong>Total Killed (Heatmap)</strong></summary>

```DAX

VAR k = [Total Killed]
RETURN IF(k > 0, k, BLANK())

```

</details>

<details>
<summary><strong>Total Killed (Top Countries Display)</strong></summary>

```DAX

IF( [All Countries Selected (Slicer)] = 1, [Total Killed], BLANK() )


```

</details>

<details>
<summary><strong>Total Nationals Detained</strong></summary>

```DAX

COALESCE ( SUM ( 'F Incident'[Nationals Detained]), 0 )
```

</details>

<details>
<summary><strong>Total Nationals Kidnapped</strong></summary>

```DAX

COALESCE ( SUM ( 'F Incident'[Nationals Kidnapped]), 0 )
```

</details>

<details>
<summary><strong>Total Nationals Killed</strong></summary>

```DAX

COALESCE ( SUM ( 'F Incident'[Nationals Killed]), 0 )
```

</details>

<details>
<summary><strong>Total Nationals Wounded</strong></summary>

```DAX

COALESCE ( SUM ( 'F Incident'[Nationals Wounded]), 0 )
```

</details>

<details>
<summary><strong>Total Victims Females</strong></summary>

```DAX

COALESCE ( SUM ( 'F Incident'[Gender Female]), 0 )
```

</details>

<details>
<summary><strong>Total Victims Gender Unknown</strong></summary>

```DAX

COALESCE ( SUM ( 'F Incident'[Gender Unknown]), 0 )
```

</details>

<details>
<summary><strong>Total Victims IFRC ICRC</strong></summary>

```DAX

COALESCE ( SUM ( 'F Incident'[IFRC ICRC]), 0 )
```

</details>

<details>
<summary><strong>Total Victims INGO</strong></summary>

```DAX

COALESCE ( SUM ( 'F Incident'[INGO]), 0 )
```

</details>

<details>
<summary><strong>Total Victims Males</strong></summary>

```DAX

COALESCE ( SUM ( 'F Incident'[Gender Male] ), 0 )
```

</details>

<details>
<summary><strong>Total Victims NNGO</strong></summary>

```DAX

COALESCE ( SUM ( 'F Incident'[NNGO]), 0 )
```

</details>

<details>
<summary><strong>Total Victims NRCS</strong></summary>

```DAX

COALESCE ( SUM ( 'F Incident'[NRCS]), 0 )
```

</details>

<details>
<summary><strong>Total Victims Other Org</strong></summary>

```DAX

COALESCE ( SUM ( 'F Incident'[Other Org] ), 0 )
```

</details>

<details>
<summary><strong>Total Victims UN</strong></summary>

```DAX

COALESCE ( SUM ( 'F Incident'[UN]), 0 )
```

</details>

<details>
<summary><strong>Total Wounded</strong></summary>

```DAX

SUMX (
    'F incident',
    COALESCE ( 'F incident'[Nationals wounded], 0 )
    + COALESCE ( 'F incident'[Internationals wounded], 0 )
)
```

</details>

<details>
<summary><strong>Total Wounded (Top Countries Display)</strong></summary>

```DAX

IF(
    [All Countries Selected (Slicer)]= 1,
    [Total Wounded],
    BLANK()
)

```

</details>

<details>
<summary><strong>Trend Multi-Outcome Title</strong></summary>

```DAX

VAR _countrySingle =
    SELECTEDVALUE ( 'D Location'[Country] )

VAR _availableCountries =
    CALCULATETABLE (
        VALUES ( 'D Location'[Country] ),
        REMOVEFILTERS ( 'D Location'[Country] ),
        'D Location'[Country] <> BLANK()
    )

VAR _selectedCountries =
    FILTER (
        VALUES ( 'D Location'[Country] ),
        'D Location'[Country] <> BLANK()
    )

VAR _excludedCount =
    COUNTROWS ( EXCEPT ( _availableCountries, _selectedCountries ) )

VAR _allCountriesSelected =
    _excludedCount = 0

VAR _baseTitle =
    SWITCH (
        TRUE(),
        _allCountriesSelected,
            "Outcomes of attacks on aid workers worldwide across all reported years",
        NOT ISBLANK ( _countrySingle ),
            "Outcomes of attacks on aid workers in " & _countrySingle & " across all reported years",
        "Outcomes of attacks on aid workers in selected countries across all reported years"
    )

VAR _yearFiltered =
    ISFILTERED ( 'D Date'[Year] )
        || ISCROSSFILTERED ( 'D Date'[Year] )

VAR _selectedOutcome =
    SELECTEDVALUE ( Outcome[Outcome], "All" )

VAR _countChangedFromDefault =
    _selectedOutcome <> "All"

VAR _showNote =
    _yearFiltered || _countChangedFromDefault

RETURN
    _baseTitle
        & IF ( _showNote, " (not impacted by year or count selection)", "" )
```

</details>

<details>
<summary><strong>Update Message</strong></summary>

```DAX

"The report is updated weekly, with the most recent update on "
& FORMAT ( MAX ( refresh_audit[last_successful_load] ), "d MMMM yyyy" )
& "."
```

</details>

<details>
<summary><strong>Wounded (3-year avg)</strong></summary>

```DAX

VAR IsSingleYear = HASONEVALUE ( 'D Date'[Year] )
VAR Y = SELECTEDVALUE ( 'D Date'[Year] )
VAR FirstY = [First Data Year]

VAR V1 =
    CALCULATE (
        [Total Wounded],
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 1
    )
VAR V2 =
    CALCULATE (
        [Total Wounded],
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 2
    )
VAR V3 =
    CALCULATE (
        [Total Wounded],
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 3
    )

VAR YearsOverZero =
    IF ( NOT ISBLANK ( V1 ) && V1 > 0, 1, 0 ) +
    IF ( NOT ISBLANK ( V2 ) && V2 > 0, 1, 0 ) +
    IF ( NOT ISBLANK ( V3 ) && V3 > 0, 1, 0 )

RETURN
IF (
    NOT IsSingleYear
        || Y <= FirstY + 2
        || YearsOverZero < 2,
    BLANK(),
    DIVIDE ( COALESCE ( V1, 0 ) + COALESCE ( V2, 0 ) + COALESCE ( V3, 0 ), 3 )
)


```

</details>

<details>
<summary><strong>Wounded (line)</strong></summary>

```DAX

IF (
    SELECTEDVALUE ( Outcome[Outcome], "Killed" ) = "Wounded",
    [Total Wounded],
    BLANK()
)
```

</details>

<details>
<summary><strong>Wounded (Previous Year)</strong></summary>

```DAX

VAR Y = [Selected Year]
VAR FirstY = [First Data Year]
RETURN
IF (
    Y <= FirstY,
    BLANK(),
    CALCULATE (
        [Total Wounded],
        FILTER ( ALL ( 'D Date'[Year] ), 'D Date'[Year] = Y - 1 )
    )
)


```

</details>

<details>
<summary><strong>Wounded (Selected Year)</strong></summary>

```DAX

VAR Y = [Selected Year]
RETURN
CALCULATE(
    [Total Wounded],
    FILTER(ALL('D Date'[Year]), 'D Date'[Year] = Y)
)

```

</details>

<details>
<summary><strong>Wounded Comparison Title</strong></summary>

```DAX

VAR IsSingleYear = HASONEVALUE ( 'D Date'[Year] )

VAR IsGlobal =
    NOT ISFILTERED ( 'D Location'[Country] )

VAR CountryLabel =
    SELECTEDVALUE ( 'D Location'[Country] )

VAR TitlePrefix =
    IF (
        IsGlobal,
        "Wounded globally — ",
        "Wounded in " & CountryLabel & " — "
    )

VAR Y = SELECTEDVALUE ( 'D Date'[Year] )
VAR CurWounded = COALESCE ( [Total Wounded], 0 )

VAR CurIncRows =
    CALCULATE (
        COUNTROWS ( 'F Incident' ),
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y
    )

VAR PrevIncRows =
    CALCULATE (
        COUNTROWS ( 'F Incident' ),
        REMOVEFILTERS ( 'D Date'[Year] ),
        'D Date'[Year] = Y - 1
    )

VAR FirstIncidentYear =
    MINX (
        FILTER (
            ALL ( 'D Date'[Year] ),
            CALCULATE ( COUNTROWS ( 'F Incident' ) ) > 0
        ),
        'D Date'[Year]
    )

VAR PrevWounded =
    COALESCE (
        CALCULATE (
            [Total Wounded],
            REMOVEFILTERS ( 'D Date'[Year] ),
            'D Date'[Year] = Y - 1
        ),
        0
    )

VAR YoY_Pct =
    IF ( PrevWounded > 0, DIVIDE ( CurWounded - PrevWounded, PrevWounded ), BLANK () )

VAR BracketLabel =
    SWITCH (
        TRUE (),

        PrevIncRows = 0,
            BLANK (),

        PrevWounded > 0 && CurWounded > PrevWounded,
            "(" & UNICHAR ( 9650 ) & " " & FORMAT ( YoY_Pct, "0%" ) & ")",

        PrevWounded > 0 && CurWounded < PrevWounded,
            "(" & UNICHAR ( 9660 ) & " " & FORMAT ( ABS ( YoY_Pct ), "0%" ) & ")",

        PrevWounded > 0 && CurWounded = PrevWounded,
            "(no change)",

        PrevWounded = 0 && CurWounded > 0,
            "(" & UNICHAR ( 9650 ) & " from 0)",

        PrevWounded = 0 && CurWounded = 0,
            "(no change)"
    )

VAR Avg3 = [Wounded (3-year avg)]
VAR AvgText =
    IF (
        NOT ISBLANK ( Avg3 ),
        " and the 3-year average (" & FORMAT ( Y - 3, "0" ) & "–" & FORMAT ( Y - 1, "0" ) & ")",
        BLANK ()
    )

VAR MainText =
    SWITCH (
        TRUE (),

        NOT IsSingleYear,
            "Graph not applicable for multiple years",

        CurIncRows = 0,
            FORMAT ( Y, "0" ) & " (no incidents recorded)",

        Y = FirstIncidentYear,
            FORMAT ( Y, "0" ) & " (first year with incidents recorded)",

        PrevIncRows = 0,
            FORMAT ( Y, "0" ) & " (no incidents recorded in " & FORMAT ( Y - 1, "0" ) & ")",

        TRUE,
            FORMAT ( Y, "0" ) &
            " compared with " & FORMAT ( Y - 1, "0" ) &
            IF ( NOT ISBLANK ( BracketLabel ), " " & BracketLabel, BLANK () ) &
            AvgText
    )

RETURN
TitlePrefix & MainText
```

</details>

<details>
<summary><strong>Year Label (Title)</strong></summary>

```DAX

VAR _hasOneYear = HASONEVALUE('D Date'[Year])
VAR _hasYearFilter = ISFILTERED('D Date'[Year])
VAR _year = SELECTEDVALUE('D Date'[Year])
RETURN
SWITCH(
    TRUE(),
    NOT _hasYearFilter, "all years",
    _hasOneYear, FORMAT(_year, "0"),
    "selected years"
)

```

</details>

<details>
<summary><strong>Year Phrase (Title)</strong></summary>

```DAX

VAR _hasOneYear = HASONEVALUE('D Date'[Year])
VAR _hasYearFilter = ISFILTERED('D Date'[Year])
VAR _year = SELECTEDVALUE('D Date'[Year])
RETURN
SWITCH(
    TRUE(),
    NOT _hasYearFilter, "across all reported years",
    _hasOneYear, "in " & FORMAT(_year, "0"),
    "across all reported years"
)

```

</details>

</details>
