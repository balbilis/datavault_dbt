

the structure contains:

Hubs: HUB_Emp, HUB_Project, HUB_Dep
Links: LINK_Emp_Project, LINK_Emp_Dep
Satellites: SAT_Emp, SAT_Address, SAT_Project, SAT_Dep
Sources: SRC_Emp, SRC_Project, SRC_Dep, SRC_WorkOn



Step 1: Model for Hub Table: HUB_Emp

-- models/hub_emp.sql
WITH source_data AS (
    SELECT
        Emp_ID AS Emp_ID,
        CURRENT_TIMESTAMP() AS LOAD_DTS,
        'SRC_Emp' AS REC_SRC
    FROM {{ ref('src_emp') }} -- source table
)

SELECT
    MD5(Emp_ID) AS HK_Emp_ID, -- Hash key
    Emp_ID,
    LOAD_DTS,
    REC_SRC
FROM source_data
GROUP BY Emp_ID, LOAD_DTS, REC_SRC


Purpose: Hubs are used to store the unique business keys (Emp_ID) and metadata such as LOAD_DTS (Load Date Timestamp) and REC_SRC 

Step 2: Model for Satellite Table: SAT_Emp

-- models/sat_emp.sql
WITH source_data AS (
    SELECT
        MD5(Emp_ID) AS HK_Emp_ID, -- Hash key
        CURRENT_TIMESTAMP() AS LOAD_DTS,
        Name,
        Gender,
        Birthdate,
        'SRC_Emp' AS REC_SRC
    FROM {{ ref('src_emp') }} -- source table
)

SELECT
    HK_Emp_ID,
    LOAD_DTS,
    Name,
    Gender,
    Birthdate,
    REC_SRC
FROM source_data

Purpose: Satellites store descriptive attributes (Name, Gender, Birthdate) associated with a business entity in the hub, and are linked via HK_Emp_ID.

Step 3: Model for Link Table: LINK_Emp_Project

-- models/link_emp_project.sql
WITH source_data AS (
    SELECT
        MD5(Emp_ID || Project_ID) AS HK_L_Emp_Project, -- Hash key for the link
        MD5(Emp_ID) AS HK_Emp_ID,
        MD5(Project_ID) AS HK_Project_ID,
        CURRENT_TIMESTAMP() AS LOAD_DTS,
        'SRC_WorkOn' AS REC_SRC
    FROM {{ ref('src_workon') }} -- source table for employee and project work relationships
)

SELECT
    HK_L_Emp_Project,
    HK_Emp_ID,
    HK_Project_ID,
    LOAD_DTS,
    REC_SRC
FROM source_data

Purpose: Links are used to establish relationships between two hubs. This link connects HUB_Emp and HUB_Project.

Step 4: Model for Hub Table: HUB_Project

-- models/hub_project.sql
WITH source_data AS (
    SELECT
        Project_ID AS Project_ID,
        CURRENT_TIMESTAMP() AS LOAD_DTS,
        'SRC_Project' AS REC_SRC
    FROM {{ ref('src_project') }} -- source table
)

SELECT
    MD5(Project_ID) AS HK_Project_ID, -- Hash key
    Project_ID,
    LOAD_DTS,
    REC_SRC
FROM source_data
GROUP BY Project_ID, LOAD_DTS, REC_SRC

Step 5: Model for Satellite Table: SAT_Project
-- models/sat_project.sql
WITH source_data AS (
    SELECT
        MD5(Project_ID) AS HK_Project_ID, -- Hash key
        CURRENT_TIMESTAMP() AS LOAD_DTS,
        Project_Name,
        Project_Code,
        'SRC_Project' AS REC_SRC
    FROM {{ ref('src_project') }} -- source table
)

SELECT
    HK_Project_ID,
    LOAD_DTS,
    Project_Name,
    Project_Code,
    REC_SRC
FROM source_data

Step 6: Model for Link Table: LINK_Emp_Dep

-- models/link_emp_dep.sql
WITH source_data AS (
    SELECT
        MD5(Emp_ID || Dep_ID) AS HK_L_Emp_Dep, -- Hash key for the link
        MD5(Emp_ID) AS HK_Emp_ID,
        MD5(Dep_ID) AS HK_Dep_ID,
        CURRENT_TIMESTAMP() AS LOAD_DTS,
        'SRC_Emp' AS REC_SRC
    FROM {{ ref('src_emp') }} -- source table for employee and department relationships
)

SELECT
    HK_L_Emp_Dep,
    HK_Emp_ID,
    HK_Dep_ID,
    LOAD_DTS,
    REC_SRC
FROM source_data

Step 7: Model for Hub Table: HUB_Dep

-- models/hub_dep.sql
WITH source_data AS (
    SELECT
        Dep_ID AS Dep_ID,
        CURRENT_TIMESTAMP() AS LOAD_DTS,
        'SRC_Dep' AS REC_SRC
    FROM {{ ref('src_dep') }} -- source table
)

SELECT
    MD5(Dep_ID) AS HK_Dep_ID, -- Hash key
    Dep_ID,
    LOAD_DTS,
    REC_SRC
FROM source_data
GROUP BY Dep_ID, LOAD_DTS, REC_SRC

Step 8: Model for Satellite Table: SAT_Dep

-- models/sat_dep.sql
WITH source_data AS (
    SELECT
        MD5(Dep_ID) AS HK_Dep_ID, -- Hash key
        CURRENT_TIMESTAMP() AS LOAD_DTS,
        Dep_Name,
        Dep_Code,
        'SRC_Dep' AS REC_SRC
    FROM {{ ref('src_dep') }} -- source table
)

SELECT
    HK_Dep_ID,
    LOAD_DTS,
    Dep_Name,
    Dep_Code,
    REC_SRC
FROM source_data



Key Concepts:
Hubs store unique business keys.
Links establish relationships between business keys (Hubs).
Satellites store descriptive attributes related to Hubs or Links.
Hash Keys (MD5) are used to uniquely identify records and avoid collisions.
LOAD_DTS is used to track when data was loaded.
