# Aid Worker Security Incidents BI Report

This interactive Power BI report presents major security incidents affecting humanitarian personnel worldwide, using data from the [Aid Worker Security Database (AWSD)](https://www.aidworkersecurity.org/). It supports self-directed exploration of nearly three decades of AWSD records.


🔗 [View the live report](https://maria-wiatr.github.io/Report_AWSD_em/) 🔗

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

<img width="949" height="345" alt="image" src="https://github.com/user-attachments/assets/63ff3cce-ffc6-4908-b238-2dfe8ae92cc9" />


---

## Technical References

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
### Reference F – DAX Measures

The list below contains the report measures and their DAX definitions.

<details>
<summary><strong>Tooltip - PerpParam Active1</strong></summary>

```DAX
VAR _fields =
    SELECTEDVALUE ( 'Perpetrator Dimension Parameter1'[Parameter Fields] )
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
<summary><strong>Tooltip - PerpParam TermDefinition correct</strong></summary>

```DAX
VAR _active = [Tooltip - PerpParam Active1]

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
<summary><strong>Tooltip Definition correct</strong></summary>

```DAX
VAR _order = SELECTEDVALUE('Attack Dimension Parameter1'[Parameter Order]) RETURN SWITCH( _order, 0, SELECTEDVALUE(AttackMeans_Definitions[Definition]), 1, SELECTEDVALUE(AttackContext_Definitions[Definition]), 2, SELECTEDVALUE(IncidentSetting_Definitions[Definition]), BLANK() )
```
</details>

<details>
<summary><strong>% Nationals</strong></summary>

```DAX
EXTERNALMEASURE("% Nationals", DOUBLE, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Comparison Value (Outcome)</strong></summary>

```DAX
EXTERNALMEASURE("Comparison Value (Outcome)", DOUBLE, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Country Phrase (Map Title)</strong></summary>

```DAX
EXTERNALMEASURE("Country Phrase (Map Title)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Country Rank (Nat+Int)</strong></summary>

```DAX
EXTERNALMEASURE("Country Rank (Nat+Int)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Country Rank (Selected Outcome)</strong></summary>

```DAX
EXTERNALMEASURE("Country Rank (Selected Outcome)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Dynamic Total Label</strong></summary>

```DAX
VAR OutcomeSelected =
    SELECTEDVALUE ( Outcome[Outcome], "All" )
RETURN
    SWITCH (
        OutcomeSelected,
        "All", "Total affected:",
        "Killed", "Total killed:",
        "Wounded", "Total wounded:",
        "Kidnapped", "Total kidnapped:",
        "Detained", "Total detained:",
        "Total affected:"
    )
```
</details>

<details>
<summary><strong>Gender Total (All outcomes)</strong></summary>

```DAX
EXTERNALMEASURE("Gender Total (All outcomes)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Has Data (Nat+Int)</strong></summary>

```DAX
EXTERNALMEASURE("Has Data (Nat+Int)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Internationals by Outcome (Axis)</strong></summary>

```DAX
EXTERNALMEASURE("Internationals by Outcome (Axis)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>KPI Column Header</strong></summary>

```DAX
EXTERNALMEASURE("KPI Column Header", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Map Title (Dynamic)</strong></summary>

```DAX
EXTERNALMEASURE("Map Title (Dynamic)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Nat vs Int Difference</strong></summary>

```DAX
EXTERNALMEASURE("Nat vs Int Difference", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Nationals by Outcome (Axis)</strong></summary>

```DAX
EXTERNALMEASURE("Nationals by Outcome (Axis)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Org Victims Total (Pie)</strong></summary>

```DAX
EXTERNALMEASURE("Org Victims Total (Pie)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Outcome Comparison Title</strong></summary>

```DAX
EXTERNALMEASURE("Outcome Comparison Title", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Outcome Label (Title)</strong></summary>

```DAX
EXTERNALMEASURE("Outcome Label (Title)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Outcome Phrase (Title)</strong></summary>

```DAX
EXTERNALMEASURE("Outcome Phrase (Title)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Previous Year (Outcome)</strong></summary>

```DAX
EXTERNALMEASURE("Previous Year (Outcome)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Outcome Color</strong></summary>

```DAX
EXTERNALMEASURE("Selected Outcome Color", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Outcome Total — Internationals</strong></summary>

```DAX
EXTERNALMEASURE("Selected Outcome Total — Internationals", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Outcome Total — Internationals (Plot)</strong></summary>

```DAX
EXTERNALMEASURE("Selected Outcome Total — Internationals (Plot)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Outcome Total — Nat+Int</strong></summary>

```DAX
EXTERNALMEASURE("Selected Outcome Total — Nat+Int", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Outcome Total — Nat+Int1</strong></summary>

```DAX
EXTERNALMEASURE("Selected Outcome Total — Nat+Int1", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Outcome Total — Nationals</strong></summary>

```DAX
EXTERNALMEASURE("Selected Outcome Total — Nationals", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Outcome Total — Nationals (Plot)</strong></summary>

```DAX
EXTERNALMEASURE("Selected Outcome Total — Nationals (Plot)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Year (Outcome)</strong></summary>

```DAX
EXTERNALMEASURE("Selected Year (Outcome)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Show Top 3 Countries</strong></summary>

```DAX
EXTERNALMEASURE("Show Top 3 Countries", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Title — &lt;Visual Name&gt;</strong></summary>

```DAX
EXTERNALMEASURE("Title — <Visual Name>", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Title — Gender Distribution</strong></summary>

```DAX
EXTERNALMEASURE("Title — Gender Distribution", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Title — Nat vs Int by Outcome</strong></summary>

```DAX
EXTERNALMEASURE("Title — Nat vs Int by Outcome", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Title — Nat vs Int Trend</strong></summary>

```DAX
EXTERNALMEASURE("Title — Nat vs Int Trend", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Title — Nationals vs Internationals</strong></summary>

```DAX
EXTERNALMEASURE("Title — Nationals vs Internationals", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Title — Top Countries (Nat vs Int)</strong></summary>

```DAX
EXTERNALMEASURE("Title — Top Countries (Nat vs Int)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Top Countries Title (Dynamic)</strong></summary>

```DAX
EXTERNALMEASURE("Top Countries Title (Dynamic)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Trend Multi-Outcome Title</strong></summary>

```DAX
EXTERNALMEASURE("Trend Multi-Outcome Title", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Year Label (Title)</strong></summary>

```DAX
EXTERNALMEASURE("Year Label (Title)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Year Phrase (Title)</strong></summary>

```DAX
EXTERNALMEASURE("Year Phrase (Title)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>All Countries Selected</strong></summary>

```DAX
EXTERNALMEASURE("All Countries Selected", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Country Filter Applied</strong></summary>

```DAX
EXTERNALMEASURE("Country Filter Applied", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Country Filter Applied (Slicer Only)</strong></summary>

```DAX
EXTERNALMEASURE("Country Filter Applied (Slicer Only)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Incidents Kidnapped</strong></summary>

```DAX
EXTERNALMEASURE("Incidents Kidnapped", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Incidents Killed</strong></summary>

```DAX
EXTERNALMEASURE("Incidents Killed", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Incidents Wounded</strong></summary>

```DAX
EXTERNALMEASURE("Incidents Wounded", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Show Top Countries Visual</strong></summary>

```DAX
EXTERNALMEASURE("Show Top Countries Visual", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Title — Heatmap (Dynamic)</strong></summary>

```DAX
EXTERNALMEASURE("Title — Heatmap (Dynamic)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Incidents (Selected Outcome)</strong></summary>

```DAX
EXTERNALMEASURE("Total Incidents (Selected Outcome)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Killed (Heatmap)</strong></summary>

```DAX
EXTERNALMEASURE("Total Killed (Heatmap)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>All Countries Selected (Slicer)</strong></summary>

```DAX
EXTERNALMEASURE("All Countries Selected (Slicer)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Attack Dimension Definition</strong></summary>

```DAX
EXTERNALMEASURE("Attack Dimension Definition", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Country Rank (Incidents)</strong></summary>

```DAX
EXTERNALMEASURE("Country Rank (Incidents)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Country Total (TopN Perpetrator Heatmap)</strong></summary>

```DAX
EXTERNALMEASURE("Country Total (TopN Perpetrator Heatmap)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Country Total (TopN)</strong></summary>

```DAX
EXTERNALMEASURE("Country Total (TopN)", INTEGER, "DirectQuery to AS - SM_AWSD")
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
<summary><strong>Dynamic Tree Title 2</strong></summary>

```DAX
EXTERNALMEASURE("Dynamic Tree Title", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>First Data Year</strong></summary>

```DAX
EXTERNALMEASURE("First Data Year", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Gender Female Total</strong></summary>

```DAX
EXTERNALMEASURE("Gender Female Total", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Gender Male Total</strong></summary>

```DAX
EXTERNALMEASURE("Gender Male Total", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Gender Unknown Total</strong></summary>

```DAX
EXTERNALMEASURE("Gender Unknown Total", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>HelpHover</strong></summary>

```DAX
EXTERNALMEASURE("HelpHover", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Kidnapped (3-year avg)</strong></summary>

```DAX
EXTERNALMEASURE("Kidnapped (3-year avg)", DOUBLE, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Kidnapped (line)</strong></summary>

```DAX
EXTERNALMEASURE("Kidnapped (line)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Kidnapped (Previous Year)</strong></summary>

```DAX
EXTERNALMEASURE("Kidnapped (Previous Year)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Kidnapped (Selected Year)</strong></summary>

```DAX
EXTERNALMEASURE("Kidnapped (Selected Year)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Kidnapped Comparison Title</strong></summary>

```DAX
EXTERNALMEASURE("Kidnapped Comparison Title", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Killed (3-year avg)</strong></summary>

```DAX
EXTERNALMEASURE("Killed (3-year avg)", DOUBLE, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Killed (line)</strong></summary>

```DAX
EXTERNALMEASURE("Killed (line)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Killed (Previous Year)</strong></summary>

```DAX
EXTERNALMEASURE("Killed (Previous Year)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Killed (Selected Year)</strong></summary>

```DAX
EXTERNALMEASURE("Killed (Selected Year)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Killed Comparison Title</strong></summary>

```DAX
EXTERNALMEASURE("Killed Comparison Title", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Killed YoY %</strong></summary>

```DAX
EXTERNALMEASURE("Killed YoY %", DOUBLE, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Killed YoY Text</strong></summary>

```DAX
EXTERNALMEASURE("Killed YoY Text", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Last Refresh Text</strong></summary>

```DAX
EXTERNALMEASURE("Last Refresh Text", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Line Title</strong></summary>

```DAX
EXTERNALMEASURE("Line Title", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Map Title</strong></summary>

```DAX
EXTERNALMEASURE("Map Title", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Map Value</strong></summary>

```DAX
EXTERNALMEASURE("Map Value", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Msg — Select one year</strong></summary>

```DAX
EXTERNALMEASURE("Msg — Select one year", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Nationality Group</strong></summary>

```DAX
EXTERNALMEASURE("Nationality Group", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Outcome (3-year avg)</strong></summary>

```DAX
EXTERNALMEASURE("Outcome (3-year avg)", DOUBLE, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Prev Year</strong></summary>

```DAX
EXTERNALMEASURE("Prev Year", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Attack Dimension</strong></summary>

```DAX
EXTERNALMEASURE("Selected Attack Dimension", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Category</strong></summary>

```DAX
EXTERNALMEASURE("Selected Category", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Definition</strong></summary>

```DAX
EXTERNALMEASURE("Selected Definition", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Dictionary Definition</strong></summary>

```DAX
EXTERNALMEASURE("Selected Dictionary Definition", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Dictionary Field</strong></summary>

```DAX
EXTERNALMEASURE("Selected Dictionary Field", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Outcome Total</strong></summary>

```DAX
EXTERNALMEASURE("Selected Outcome Total", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Outcome Total (Gender)</strong></summary>

```DAX
EXTERNALMEASURE("Selected Outcome Total (Gender)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Outcome Total1</strong></summary>

```DAX
EXTERNALMEASURE("Selected Outcome Total1", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Term</strong></summary>

```DAX
EXTERNALMEASURE("Selected Term", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Selected Year</strong></summary>

```DAX
EXTERNALMEASURE("Selected Year", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Show Comparison</strong></summary>

```DAX
EXTERNALMEASURE("Show Comparison", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Show Country Message</strong></summary>

```DAX
EXTERNALMEASURE("Show Country Message", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Show Top Countries Chart</strong></summary>

```DAX
EXTERNALMEASURE("Show Top Countries Chart", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>SortValue_UnknownLast_AttackPage</strong></summary>

```DAX
EXTERNALMEASURE("SortValue_UnknownLast_AttackPage", DOUBLE, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Title — Attack Dimension (Dynamic)</strong></summary>

```DAX
EXTERNALMEASURE("Title — Attack Dimension (Dynamic)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Title — Motive (Dynamic)</strong></summary>

```DAX
EXTERNALMEASURE("Title — Motive (Dynamic)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Title — Organisation Type Distribution</strong></summary>

```DAX
EXTERNALMEASURE("Title — Organisation Type Distribution", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Title — Perpetrator Type (Dynamic)</strong></summary>

```DAX
EXTERNALMEASURE("Title — Perpetrator Type (Dynamic)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Tooltip - Organisation TermDefinition</strong></summary>

```DAX
EXTERNALMEASURE("Tooltip - Organisation TermDefinition", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Tooltip - Perpetrator Definition</strong></summary>

```DAX
EXTERNALMEASURE("Tooltip - Perpetrator Definition", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Tooltip - Perpetrator Term + Definition</strong></summary>

```DAX
EXTERNALMEASURE("Tooltip - Perpetrator Term + Definition", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Tooltip - PerpParam Active</strong></summary>

```DAX
EXTERNALMEASURE("Tooltip - PerpParam Active", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Tooltip - PerpParam TermDefinition</strong></summary>

```DAX
EXTERNALMEASURE("Tooltip - PerpParam TermDefinition", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Tooltip Definition</strong></summary>

```DAX
EXTERNALMEASURE("Tooltip Definition", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Top 10 Share %</strong></summary>

```DAX
EXTERNALMEASURE("Top 10 Share %", DOUBLE, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Top 5 Countries Group</strong></summary>

```DAX
EXTERNALMEASURE("Top 5 Countries Group", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Top Countries Title (Kidnapped)</strong></summary>

```DAX
EXTERNALMEASURE("Top Countries Title (Kidnapped)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Top Countries Title (Killed)</strong></summary>

```DAX
EXTERNALMEASURE("Top Countries Title (Killed)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Top Countries Title (Wounded)</strong></summary>

```DAX
EXTERNALMEASURE("Top Countries Title (Wounded)", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Top Countries Value (Display)</strong></summary>

```DAX
EXTERNALMEASURE("Top Countries Value (Display)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>TopN + Other Value</strong></summary>

```DAX
EXTERNALMEASURE("TopN + Other Value", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>TopN Country Name</strong></summary>

```DAX
EXTERNALMEASURE("TopN Country Name", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>TopN Cumulative %</strong></summary>

```DAX
EXTERNALMEASURE("TopN Cumulative %", DOUBLE, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total affected</strong></summary>

```DAX
EXTERNALMEASURE("Total affected", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Countries (Year + Country only)</strong></summary>

```DAX
EXTERNALMEASURE("Total Countries (Year + Country only)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Detained</strong></summary>

```DAX
EXTERNALMEASURE("Total Detained", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Detained (2025+)</strong></summary>

```DAX
EXTERNALMEASURE("Total Detained (2025+)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Detained (pre-2025)</strong></summary>

```DAX
EXTERNALMEASURE("Total Detained (pre-2025)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Incidents</strong></summary>

```DAX
EXTERNALMEASURE("Total Incidents", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Internationals Detained</strong></summary>

```DAX
EXTERNALMEASURE("Total Internationals Detained", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Internationals Kidnapped</strong></summary>

```DAX
EXTERNALMEASURE("Total Internationals Kidnapped", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Internationals Killed</strong></summary>

```DAX
EXTERNALMEASURE("Total Internationals Killed", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Internationals Wounded</strong></summary>

```DAX
EXTERNALMEASURE("Total Internationals Wounded", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Kidnapped</strong></summary>

```DAX
EXTERNALMEASURE("Total Kidnapped", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Kidnapped (Top Countries Display))</strong></summary>

```DAX
EXTERNALMEASURE("Total Kidnapped (Top Countries Display))", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Killed</strong></summary>

```DAX
EXTERNALMEASURE("Total Killed", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Killed (Top Countries Display)</strong></summary>

```DAX
EXTERNALMEASURE("Total Killed (Top Countries Display)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Nationals Detained</strong></summary>

```DAX
EXTERNALMEASURE("Total Nationals Detained", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Nationals Kidnapped</strong></summary>

```DAX
EXTERNALMEASURE("Total Nationals Kidnapped", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Nationals Killed</strong></summary>

```DAX
EXTERNALMEASURE("Total Nationals Killed", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Nationals Wounded</strong></summary>

```DAX
EXTERNALMEASURE("Total Nationals Wounded", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Victims Females</strong></summary>

```DAX
EXTERNALMEASURE("Total Victims Females", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Victims Gender Unknown</strong></summary>

```DAX
EXTERNALMEASURE("Total Victims Gender Unknown", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Victims IFRC ICRC</strong></summary>

```DAX
EXTERNALMEASURE("Total Victims IFRC ICRC", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Victims INGO</strong></summary>

```DAX
EXTERNALMEASURE("Total Victims INGO", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Victims Males</strong></summary>

```DAX
EXTERNALMEASURE("Total Victims Males", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Victims NNGO</strong></summary>

```DAX
EXTERNALMEASURE("Total Victims NNGO", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Victims NRCS</strong></summary>

```DAX
EXTERNALMEASURE("Total Victims NRCS", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Victims Other Org</strong></summary>

```DAX
EXTERNALMEASURE("Total Victims Other Org", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Victims UN</strong></summary>

```DAX
EXTERNALMEASURE("Total Victims UN", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Wounded</strong></summary>

```DAX
EXTERNALMEASURE("Total Wounded", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Total Wounded (Top Countries Display)</strong></summary>

```DAX
EXTERNALMEASURE("Total Wounded (Top Countries Display)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Update Message</strong></summary>

```DAX
EXTERNALMEASURE("Update Message", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Wounded (3-year avg)</strong></summary>

```DAX
EXTERNALMEASURE("Wounded (3-year avg)", DOUBLE, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Wounded (line)</strong></summary>

```DAX
EXTERNALMEASURE("Wounded (line)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Wounded (Previous Year)</strong></summary>

```DAX
EXTERNALMEASURE("Wounded (Previous Year)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Wounded (Selected Year)</strong></summary>

```DAX
EXTERNALMEASURE("Wounded (Selected Year)", INTEGER, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Wounded Comparison Title</strong></summary>

```DAX
EXTERNALMEASURE("Wounded Comparison Title", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>

<details>
<summary><strong>Tooltip - Motive TermDefinition</strong></summary>

```DAX
EXTERNALMEASURE("Tooltip - Motive TermDefinition", STRING, "DirectQuery to AS - SM_AWSD")
```
</details>
