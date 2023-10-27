~~~ SQL
/*Length of Employment Employee */
CREATE TEMP TABLE t2 AS
SELECT
  Name,
  ROUND(EXTRACT( DAY FROM (CURRENT_DATE() - Employement_Start_Date))/ 365.25,2) AS Years_Employed,
  ROUND(EXTRACT( DAY FROM ('2024-06-30' - Employement_Start_Date))/ 365.25,2) AS FY_24_End_Employment_Length,
  Position,
  CASE
   WHEN position LIKE '%Teacher%' THEN 'Teacher'-- Grouping all Teachers
   WHEN position LIKE '%Culture%' THEN 'LSOC' -- Grouping all Leaders of Student Culture
   WHEN position LIKE '%Substitutes%' THEN 'Substitute' -- Grouping all Subsitute Teachers
   WHEN position LIKE '%School Instructional Dean%' THEN 'Deans of Instruction' -- Grouping all Deans of Instruction
   WHEN position LIKE '%Recruit%' THEN 'Talent Professionals' -- Grouping all Talent Professionals
   WHEN position LIKE '%Paraprofessional%' THEN 'Paraprofessional' -- Groupig all Paraprofessionals 
   WHEN position LIKE '%Social Worker%' THEN 'Social Worker' -- Grouping all social workers
   WHEN position LIKE '%Facilities%' OR position = 'Custodian' THEN 'Facilities' -- Grouping all facilities workers
   WHEN position LIKE '%Technology%' OR position = 'IT Manager' THEN 'Tech Professional'
   WHEN position LIKE '%Chief%' THEN 'Chiefs'
   WHEN position LIKE '%Director%' THEN 'Directors'
   WHEN position LIKE '%Principal in Residence%' THEN 'PIR'
   WHEN position LIKE '%Principal' THEN 'Principal'
   WHEN position LIKE '%Secret%' THEN 'Secretaries'
   WHEN position LIKE '%Enrollment%' THEN 'Enrollment'
   WHEN position LIKE '%College%' THEN 'College & Career'
   WHEN position LIKE '%Counselor%' THEN 'School Counselors'
   WHEN position IN ('Part-Time Bus Driver #1','Part-Time Bus Driver #2') THEN 'P/T Bus Drivers'
   WHEN position IN ('Assistant School Business Administrator','Business Finance Manager', 'Business Finance Senior Manager','Data Analyst Manager') THEN 'Business Office'
   ELSE position 
   END AS Position_Group,
   Campus,
   CASE -- Creating new campus - ES
    WHEN Campus IN ('PS','IS') THEN 'ES'
    ELSE Campus END AS Campus_,
   Actual_Budget_Salary
FROM `single-being-353600.FA_Budget_Analysis.Budget_Analysis_23_24`
WHERE Employement_Start_Date IS NOT NULL -- Filtering out vacancies
ORDER BY Years_Employed;
-- Cameron Smith Projected to start on 9/28/23

/*Projected Add On Pay End of FY 24 -- Chiefs, Teachers, and Princiapls*/
SELECT *
FROM 
(SELECT
  t2.Name,
  t2.Position_Group,
  FY_24_End_Employment_Length,
  CASE
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 1000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 1500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 4 AND 5 THEN 2000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 5 AND 6 THEN 2500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 6 AND 7 THEN 3000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 7 AND 8 THEN 3500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 8 AND 9 THEN 4000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 9 AND 10 THEN 2500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length >= 10 THEN 5000
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 2000
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 3000
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length >= 4 THEN 5000
    WHEN t2.Position_Group = 'Chiefs' AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN t2.Position_Group = 'Chiefs' AND FY_24_End_Employment_Length >= 4 THEN 5000
    END AS Add_On
FROM t2) AS sub
WHERE sub.Add_On IS NOT NULL;

/*Associate Add-On Pay*/
SELECT
  sub.name,
  sub.position_group,
  sub.FY_24_End_Employment_Length,
  CASE
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 1000
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 6 AND 7 THEN 2500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 7 AND 8 THEN 3000
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 8 AND 9 THEN 3500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 9 AND 10 THEN 4000
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 10 AND 11 THEN 4500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length >= 11 THEN 5000
    END AS Add_On
FROM
(SELECT
  Name,
  Position,
  CASE
     WHEN Position LIKE '%Associate%' OR Position LIKE '%Paraprofessional%' OR Position = 'Custodian' OR Position LIKE '%Secret%' THEN 'Associate' END AS Position_Group, -- Grouping Associates
  FY_24_End_Employment_Length
FROM t2
ORDER BY FY_24_End_Employment_Length) AS sub
WHERE sub.Position_Group = 'Associate';

/*Projected Add On Pay End of FY 24 -- Managers and Higher Career Level*/
SELECT
  t2.name,
  CASE WHEN t2.position_group = t2.position_group THEN 'Manager_Or_Higher' END AS Position_Group,
  t2.FY_24_End_Employment_Length,
  CASE 
    WHEN t2.FY_24_End_Employment_Length < 2 THEN 0
    WHEN t2.FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 1000
    WHEN t2.FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 1500
    WHEN t2.FY_24_End_Employment_Length BETWEEN 4 AND 5 THEN 2000
    WHEN t2.FY_24_End_Employment_Length BETWEEN 5 AND 6 THEN 2500
    WHEN t2.FY_24_End_Employment_Length BETWEEN 6 AND 7 THEN 3000
    WHEN t2.FY_24_End_Employment_Length BETWEEN 7 AND 8 THEN 3500
    WHEN t2.FY_24_End_Employment_Length BETWEEN 8 AND 9 THEN 4000
    WHEN t2.FY_24_End_Employment_Length BETWEEN 9 AND 10 THEN 4500
    WHEN t2.FY_24_End_Employment_Length >= 10 THEN 5000
    END AS Add_On 
FROM t2
WHERE t2.name NOT IN -- Nested Query
(
SELECT U.Name 
FROM 
(SELECT 
  DISTINCT sub.Name,
  sub.Position_Group,
  sub.FY_24_End_Employment_Length,
  sub.Add_On
FROM
(SELECT
  t2.Name,
  t2.Position_Group,
  FY_24_End_Employment_Length,
  CASE
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 1000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 1500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 4 AND 5 THEN 2000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 5 AND 6 THEN 2500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 6 AND 7 THEN 3000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 7 AND 8 THEN 3500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 8 AND 9 THEN 4000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 9 AND 10 THEN 2500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length >= 10 THEN 5000
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 2000
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 3000
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length >= 4 THEN 5000
    WHEN t2.Position_Group = 'Chiefs' AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN t2.Position_Group = 'Chiefs' AND FY_24_End_Employment_Length >= 4 THEN 5000
    END AS Add_On
FROM t2) AS sub
WHERE sub.Add_On IS NOT NULL
UNION ALL
-- Associate Add-On Pay
SELECT
  sub.name,
  sub.position_group,
  sub.FY_24_End_Employment_Length,
  CASE
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 1000
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 6 AND 7 THEN 2500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 7 AND 8 THEN 3000
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 8 AND 9 THEN 3500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 9 AND 10 THEN 4000
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 10 AND 11 THEN 4500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length >= 11 THEN 5000
    END AS Add_On
FROM
(SELECT
  Name,
  Position,
  CASE
     WHEN Position LIKE '%Associate%' OR Position LIKE '%Paraprofessional%' OR Position = 'Custodian' OR Position LIKE '%Secret%' THEN 'Associate' END AS Position_Group, -- Grouping Associates
  FY_24_End_Employment_Length
FROM t2
ORDER BY FY_24_End_Employment_Length) AS sub
WHERE sub.Position_Group = 'Associate') AS U); -- wrapping nested query


/*UNION ALL Add-On Tables -- Chiefs, Teachers, Associates, Principals, and Managers or Higher*/
-- Projected Add On Pay End of FY 24 -- Chiefs, Teachers, and Princiapls
CREATE TEMP TABLE Add_On AS
SELECT 
  DISTINCT sub.Name,
  sub.Position_Group,
  sub.FY_24_End_Employment_Length,
  sub.Add_On
FROM
(SELECT
  t2.Name,
  t2.Position_Group,
  FY_24_End_Employment_Length,
  CASE
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 1000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 1500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 4 AND 5 THEN 2000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 5 AND 6 THEN 2500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 6 AND 7 THEN 3000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 7 AND 8 THEN 3500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 8 AND 9 THEN 4000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 9 AND 10 THEN 2500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length >= 10 THEN 5000
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 2000
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 3000
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length >= 4 THEN 5000
    WHEN t2.Position_Group = 'Chiefs' AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN t2.Position_Group = 'Chiefs' AND FY_24_End_Employment_Length >= 4 THEN 5000
    END AS Add_On
FROM t2) AS sub
WHERE sub.Add_On IS NOT NULL
UNION ALL
-- Associate Add-On Pay
SELECT
  sub.name,
  sub.position_group,
  sub.FY_24_End_Employment_Length,
  CASE
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 1000
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 6 AND 7 THEN 2500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 7 AND 8 THEN 3000
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 8 AND 9 THEN 3500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 9 AND 10 THEN 4000
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 10 AND 11 THEN 4500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length >= 11 THEN 5000
    END AS Add_On
FROM
(SELECT
  Name,
  Position,
  CASE
     WHEN Position LIKE '%Associate%' OR Position LIKE '%Paraprofessional%' OR Position = 'Custodian' OR Position LIKE '%Secret%' THEN 'Associate' END AS Position_Group, -- Grouping Associates
  FY_24_End_Employment_Length
FROM t2
ORDER BY FY_24_End_Employment_Length) AS sub
WHERE sub.name <> 'VACANT' AND sub.Position_Group = 'Associate'
UNION ALL
-- Projected Add On Pay End of FY 24 -- Managers and Higher Career Level
SELECT 
  t2.name,
  CASE WHEN t2.position_group = t2.position_group THEN 'Manager_Or_Higher' END AS Position_Group,
  t2.FY_24_End_Employment_Length,
  CASE 
    WHEN t2.FY_24_End_Employment_Length < 2 THEN 0
    WHEN t2.FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 1000
    WHEN t2.FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 1500
    WHEN t2.FY_24_End_Employment_Length BETWEEN 4 AND 5 THEN 2000
    WHEN t2.FY_24_End_Employment_Length BETWEEN 5 AND 6 THEN 2500
    WHEN t2.FY_24_End_Employment_Length BETWEEN 6 AND 7 THEN 3000
    WHEN t2.FY_24_End_Employment_Length BETWEEN 7 AND 8 THEN 3500
    WHEN t2.FY_24_End_Employment_Length BETWEEN 8 AND 9 THEN 4000
    WHEN t2.FY_24_End_Employment_Length BETWEEN 9 AND 10 THEN 4500
    WHEN t2.FY_24_End_Employment_Length >= 10 THEN 5000
    END AS Add_On 
FROM t2
WHERE t2.name NOT IN -- Nested Query
(
SELECT U.Name 
FROM 
(SELECT 
  DISTINCT sub.Name,
  sub.Position_Group,
  sub.FY_24_End_Employment_Length,
  sub.Add_On
FROM
(SELECT
  t2.Name,
  t2.Position_Group,
  FY_24_End_Employment_Length,
  CASE
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 1000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 1500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 4 AND 5 THEN 2000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 5 AND 6 THEN 2500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 6 AND 7 THEN 3000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 7 AND 8 THEN 3500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 8 AND 9 THEN 4000
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length BETWEEN 9 AND 10 THEN 4500
    WHEN t2.Position_Group = 'Teacher' AND FY_24_End_Employment_Length >= 10 THEN 5000
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 2000
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 3000
    WHEN t2.Position_Group IN ('Principal','PIR') AND FY_24_End_Employment_Length >= 4 THEN 5000
    WHEN t2.Position_Group = 'Chiefs' AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN t2.Position_Group = 'Chiefs' AND FY_24_End_Employment_Length >= 4 THEN 5000
    END AS Add_On
FROM t2) AS sub
WHERE sub.Add_On IS NOT NULL
UNION ALL
-- Associate Add-On Pay
SELECT
  sub.name,
  sub.position_group,
  sub.FY_24_End_Employment_Length,
  CASE
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length < 2 THEN 0
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 1000
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 6 AND 7 THEN 2500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 7 AND 8 THEN 3000
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 8 AND 9 THEN 3500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 9 AND 10 THEN 4000
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length BETWEEN 10 AND 11 THEN 4500
    WHEN sub. Position_Group = 'Associate' AND FY_24_End_Employment_Length >= 11 THEN 5000
    END AS Add_On
FROM
(SELECT
  Name,
  Position,
  CASE
     WHEN Position LIKE '%Associate%' OR Position LIKE '%Paraprofessional%' OR Position = 'Custodian' OR Position LIKE '%Secret%' THEN 'Associate' END AS Position_Group, -- Grouping Associates
  FY_24_End_Employment_Length
FROM t2
ORDER BY FY_24_End_Employment_Length) AS sub
WHERE sub.Position_Group = 'Associate') AS U); -- wrapping nested query


/*Conditional Add-On Data*/
SELECT *
FROM Add_On;


/*Total of Conditional Add-Ons*/
SELECT 
  SUM(Add_On.Add_On) AS Total
FROM Add_On;
-- Total projected cost $212500

/*Uniform Add-Ons*/
SELECT 
  DISTINCT t2.Name,
  t2.position_group,
  FY_24_End_Employment_Length,
  CASE
    WHEN FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 1000
    WHEN FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 1500
    WHEN FY_24_End_Employment_Length BETWEEN 4 AND 5 THEN 2000
    WHEN FY_24_End_Employment_Length BETWEEN 5 AND 6 THEN 2500
    WHEN FY_24_End_Employment_Length BETWEEN 6 AND 7 THEN 3000
    WHEN FY_24_End_Employment_Length BETWEEN 7 AND 8 THEN 3500
    WHEN FY_24_End_Employment_Length BETWEEN 8 AND 9 THEN 4000
    WHEN FY_24_End_Employment_Length BETWEEN 9 AND 10 THEN 4500
    WHEN FY_24_End_Employment_Length >= 10 THEN 5000
    ELSE 0
    END AS Uniform_Add_On
FROM t2;

/*Total of Uniform Add-Ons*/
SELECT
  SUM(sub.Uniform_Add_On) AS Total
  FROM
(SELECT 
  DISTINCT t2.Name,
  t2.position_group,
  FY_24_End_Employment_Length,
  CASE
    WHEN FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 1000
    WHEN FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 1500
    WHEN FY_24_End_Employment_Length BETWEEN 4 AND 5 THEN 2000
    WHEN FY_24_End_Employment_Length BETWEEN 5 AND 6 THEN 2500
    WHEN FY_24_End_Employment_Length BETWEEN 6 AND 7 THEN 3000
    WHEN FY_24_End_Employment_Length BETWEEN 7 AND 8 THEN 3500
    WHEN FY_24_End_Employment_Length BETWEEN 8 AND 9 THEN 4000
    WHEN FY_24_End_Employment_Length BETWEEN 9 AND 10 THEN 4500
    WHEN FY_24_End_Employment_Length >= 10 THEN 5000
    ELSE 0
    END AS Uniform_Add_On
FROM t2) AS sub;
-- Total projected cost $215,500

/*Total of the 3% COLA increase*/
SELECT
   ROUND(SUM(Actual_Budget_Salary + (Actual_Budget_Salary * 0.03)),2) AS Total
FROM `single-being-353600.FA_Budget_Analysis.Budget_Analysis_23_24`
WHERE Actual_Budget_Salary > 0 AND name <> 'Filled'; -- Filtering for filled positions
-- Total projectd 3% COLA cost $11,803,920.12

/*Total of the 3% COLA increase + Total of Conditional Add-Ons */
SELECT ROUND(SUM(Total),2) AS Overall_Total
FROM
(SELECT
   ROUND(SUM(Actual_Budget_Salary + (Actual_Budget_Salary * 0.03)),2) AS Total
FROM `single-being-353600.FA_Budget_Analysis.Budget_Analysis_23_24`
WHERE Actual_Budget_Salary > 0 AND name <> 'Filled'
UNION ALL
SELECT 
  SUM(Add_On.Add_On) AS Total
FROM Add_On);
-- Projected FY 25 Budget with Conditional Add-On $12016420.12

/*Total of the 3% COLA increase + Total of Uniform Add-Ons */
SELECT ROUND(SUM(Total),2) AS Overall_Uniform_Total_Cost
FROM
(SELECT
   ROUND(SUM(Actual_Budget_Salary + (Actual_Budget_Salary * 0.03)),2) AS Total
FROM `single-being-353600.FA_Budget_Analysis.Budget_Analysis_23_24`
WHERE Actual_Budget_Salary > 0 AND name <> 'Filled'
UNION ALL
SELECT
  SUM(sub.Uniform_Add_On) AS Total
  FROM
(SELECT 
  DISTINCT t2.Name,
  t2.position_group,
  FY_24_End_Employment_Length,
  CASE
    WHEN FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 1000
    WHEN FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 1500
    WHEN FY_24_End_Employment_Length BETWEEN 4 AND 5 THEN 2000
    WHEN FY_24_End_Employment_Length BETWEEN 5 AND 6 THEN 2500
    WHEN FY_24_End_Employment_Length BETWEEN 6 AND 7 THEN 3000
    WHEN FY_24_End_Employment_Length BETWEEN 7 AND 8 THEN 3500
    WHEN FY_24_End_Employment_Length BETWEEN 8 AND 9 THEN 4000
    WHEN FY_24_End_Employment_Length BETWEEN 9 AND 10 THEN 4500
    WHEN FY_24_End_Employment_Length >= 10 THEN 5000
    ELSE 0
    END AS Uniform_Add_On
FROM t2) AS sub);
-- Projected FY 25 Budget with Uniform Add-On $12024920.12

/*Anonymized Data Conditional Add On*/
SELECT 
  RPAD(LEFT(name,1),10,'#') AS Name, 
  Position_Group,
  FY_24_End_Employment_Length,
  Add_On
FROM Add_On;

/*Anonymized Data Uniform Add On*/
SELECT
  RPAD(LEFT(sub.name,1),10,'#') AS Name,
  sub.position_group,
  sub.FY_24_End_Employment_Length,
  sub.Uniform_Add_On
  FROM
(SELECT 
  DISTINCT t2.Name,
  t2.position_group,
  FY_24_End_Employment_Length,
  CASE
    WHEN FY_24_End_Employment_Length BETWEEN 2 AND 3 THEN 1000
    WHEN FY_24_End_Employment_Length BETWEEN 3 AND 4 THEN 1500
    WHEN FY_24_End_Employment_Length BETWEEN 4 AND 5 THEN 2000
    WHEN FY_24_End_Employment_Length BETWEEN 5 AND 6 THEN 2500
    WHEN FY_24_End_Employment_Length BETWEEN 6 AND 7 THEN 3000
    WHEN FY_24_End_Employment_Length BETWEEN 7 AND 8 THEN 3500
    WHEN FY_24_End_Employment_Length BETWEEN 8 AND 9 THEN 4000
    WHEN FY_24_End_Employment_Length BETWEEN 9 AND 10 THEN 4500
    WHEN FY_24_End_Employment_Length >= 10 THEN 5000
    ELSE 0
    END AS Uniform_Add_On
FROM t2) AS sub;
~~~
