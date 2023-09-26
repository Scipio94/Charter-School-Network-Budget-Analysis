~~~ SQL
/*Employee Metrics*/
SELECT 
  COUNT(*) AS Total_Employee_Count,
  COUNT(CASE WHEN name LIKE '%VACANT%' THEN 'VACANT' END) AS Vacant_Count,
  COUNT(*) - COUNT(CASE WHEN name LIKE '%VACANT%' THEN 'VACANT' END) AS Filled_Count,
  ROUND((COUNT(*) - COUNT(CASE WHEN name LIKE '%VACANT%' THEN 'VACANT' END))/ COUNT(*),3) * 100 AS Pct_Filled
FROM `single-being-353600.FA_Budget_Analysis.Budget_Analysis_23_24`
WHERE name <> 'ELIMINATED';
-- 186 Employees overall
-- 27 Vacancies 
-- 159 Filled Positions
-- 85.6% Filled

/*Salary Statistical Measures*/
SELECT
  MIN(Actual_Budget_Salary) AS _Min,
  MAX(Actual_Budget_Salary) AS _Max,
  MAX(Actual_Budget_Salary) - MIN(Actual_Budget_Salary) AS _Range
FROM `single-being-353600.FA_Budget_Analysis.Budget_Analysis_23_24`
WHERE name NOT LIKE '%VACANT%' AND name <> 'ELIMINATED' AND Actual_Budget_Salary > 0; -- Filtering out all vacant and eliminated positions and actual salaries w/ a value of $0

/*Distribution*/
SELECT 
 DISTINCT ROUND(AVG(Actual_Budget_Salary) OVER (),2) AS Mean,
  PERCENTILE_DISC(Actual_Budget_Salary,0.50) OVER () AS Median
FROM `single-being-353600.FA_Budget_Analysis.Budget_Analysis_23_24`
WHERE name NOT LIKE '%VACANT%' AND name <> 'ELIMINATED' AND Actual_Budget_Salary > 0; -- Filtering out all vacant and eliminated positions and actual salaries w/ a value of $0;

/*Budget Aggregate w/ vacancies*/
SELECT 
  ROUND(SUM(Finance_Budget_Salary),2) AS Projected_Salaries,
  ROUND(SUM(Actual_Budget_Salary),2) AS Actual_Salaries,
  ROUND(SUM(Finance_Budget_Salary) - SUM(Actual_Budget_Salary),2) AS Difference -- w/ vacacies
FROM `single-being-353600.FA_Budget_Analysis.Budget_Analysis_23_24`;

/*Department Actual and Projected Salaries*/
SELECT 
  DISTINCT Department,
  SUM(Finance_Budget_Salary) OVER (PARTITION BY Department) AS Projected,
  SUM(Actual_Budget_Salary) OVER (PARTITION BY Department) AS Actual,
 ROUND((SUM(Finance_Budget_Salary) OVER (PARTITION BY Department)) - (SUM(Actual_Budget_Salary) OVER (PARTITION BY Department)),2) AS Difference -- Projected - Actual
FROM `single-being-353600.FA_Budget_Analysis.Budget_Analysis_23_24`
ORDER BY Difference;

/*Campus Actual and Projected Salaries*/
SELECT 
  DISTINCT Campus,
  SUM(Finance_Budget_Salary) OVER (PARTITION BY Campus) AS Projected,
  SUM(Actual_Budget_Salary) OVER (PARTITION BY Campus) AS Actual,
 ROUND((SUM(Finance_Budget_Salary) OVER (PARTITION BY Campus)) - (SUM(Actual_Budget_Salary) OVER (PARTITION BY Campus)),2) AS Difference -- Projected - Actual
FROM `single-being-353600.FA_Budget_Analysis.Budget_Analysis_23_24`
ORDER BY Difference;

/*COLA Individual*/
SELECT 
  Name,
  Actual_Budget_Salary,
  ROUND(Actual_Budget_Salary + (Actual_Budget_Salary * 0.03),2) AS _3PCT, -- 3% COLA
  ROUND(Actual_Budget_Salary + (Actual_Budget_Salary * 0.035),2) AS _3_5PCT, -- 3.5% COLA
  ROUND(Actual_Budget_Salary + (Actual_Budget_Salary * 0.05),2) AS _5PCT-- 5% COLA
FROM `single-being-353600.FA_Budget_Analysis.Budget_Analysis_23_24`
WHERE Actual_Budget_Salary > 0 AND name <> 'Filled' -- Filtering for filled positions
ORDER BY Actual_Budget_Salary;

/*Metrics for Position*/
SELECT 
  DISTINCT sub.position_group,
  ROUND(AVG(Actual_Budget_Salary) OVER (PARTITION BY sub.position_group),2) AS Avg_Salary,
  PERCENTILE_DISC(Actual_Budget_Salary, 0.50) OVER (PARTITION BY sub.position_group) AS Median_Salary
FROM
(SELECT 
  DISTINCT Position,
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
   Actual_Budget_Salary
FROM `single-being-353600.FA_Budget_Analysis.Budget_Analysis_23_24`
WHERE Actual_Budget_Salary > 0) AS sub; -- Filtering out vacancies
-- 173 positions overall
-- 90 Teaching Positions

/*Unbudgeted Positions*/
CREATE TEMP TABLE t1 AS
SELECT *
FROM `single-being-353600.FA_Budget_Analysis.Budget_Analysis_23_24`
WHERE Budgeted = 'Unbudgeted';

/*Salary Total*/
SELECT
  ROUND(SUM(t1.Actual_Budget_Salary),2) AS Total_Unbudgeted_Salaries
FROM t1;
-- 878,822.56

/*Department Unbudgeted Hires Metrics*/
SELECT 
  DISTINCT t1.Department,
  COUNT(*) OVER (PARTITION BY t1.Department) AS _Count,
  ROUND(COUNT(*) OVER (PARTITION BY t1.Department)/ (COUNT(*) OVER ()),2) AS Pct
FROM t1;

/*Campus Unbudgeted Hires Metrics */
SELECT 
  DISTINCT t1.Campus,
  COUNT(*) OVER (PARTITION BY t1.Campus) AS _Count,
  ROUND(COUNT(*) OVER (PARTITION BY t1.Campus)/ (COUNT(*) OVER ()),2) AS Pct
FROM t1;


/*Length of Employment Employee */
CREATE TEMP TABLE t2 AS
SELECT
  Name,
  ROUND(EXTRACT( DAY FROM (CURRENT_DATE() - Employement_Start_Date))/ 365.25,2) AS Years_Employed,
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

/*Statistical Metrics*/
SELECT
  DISTINCT ROUND(AVG(t2.years_employed) OVER (),2) AS Mean,
  PERCENTILE_DISC(t2.years_employed,0.50) OVER () AS Median
FROM t2
WHERE t2.years_employed >= 0 ; --Filtering out projected hires

/*Range*/
SELECT
  MIN(t2.years_employed) AS _Min,
  MAX(t2.years_employed) AS _Max,
  MAX(t2.years_employed) - MIN(t2.years_employed) AS _Range
FROM t2
WHERE t2.years_employed >= 0; --Filtering out projected hires

/*Employee Length Tiers*/
SELECT 
  DISTINCT sub.Employee_Length_Tiers,
  COUNT(*) OVER (PARTITION BY sub.Employee_Length_Tiers) AS Employee_Tier_Length_Ct,
  ROUND(COUNT(*) OVER (PARTITION BY sub.Employee_Length_Tiers) / COUNT(*) OVER (),2) AS Employee_Tier_Length_Pct
FROM
(SELECT 
  t2.years_employed, 
  CASE 
    WHEN t2.years_employed <= 2.00 THEN 'Tier 1' 
    WHEN t2.years_employed BETWEEN 2.01 AND 5.00 THEN 'Tier 2'
    WHEN t2.years_employed > 5.00 THEN 'Tier 3' END AS Employee_Length_Tiers
FROM t2
WHERE t2.years_employed >= 0) AS sub
ORDER BY sub.Employee_Length_Tiers; --Filtering out projected hires
-- includes part-time and full-time staff
-- Approximatey half of the staff members at FA have been employed at FA for less than 2 years


/*First Year Staff Metrics*/
SELECT
  COUNT(CASE WHEN t2.years_employed <= 1.00 THEN 'First Year' END) AS First_Year_Ct,
  ROUND(COUNT(CASE WHEN t2.years_employed <= 1.00 THEN 'First Year' END)/ COUNT(*),2) AS First_Year_Pct
FROM t2
WHERE t2.years_employed >= 0;
-- Approximately one fourth of staff members have been employed at FA for less than a year

/*Less than 5 years*/
SELECT
  COUNT(CASE WHEN t2.years_employed <= 5.00 THEN 'Fifth Years' END) AS Fifth_Year_Ct,
  ROUND(COUNT(CASE WHEN t2.years_employed <= 5.00 THEN 'Fifth Years' END)/ COUNT(*),2) AS Fifth_Year_Pct
FROM t2
WHERE t2.years_employed >= 0;

/*Length of Employment Position Group */
SELECT
  DISTINCT t2.Position_Group,
  ROUND(AVG(t2.years_employed) OVER (PARTITION BY t2.position_group),2) AS Mean,
  PERCENTILE_DISC(t2.years_employed,0.50) OVER (PARTITION BY t2.position_group) AS Median,
  ROUND(AVG(t2.Actual_Budget_Salary) OVER (PARTITION BY t2.Position_Group),2) AS Avg_Position_Group_Salary,
   PERCENTILE_DISC(t2.Actual_Budget_Salary,0.50) OVER (PARTITION BY t2.position_group) AS Median_Position_Group_Salary
FROM t2
WHERE t2.years_employed >= 0
ORDER BY Median;

/*Length of Employement Campus*/
SELECT
  DISTINCT t2.Campus_,
  ROUND(AVG(t2.years_employed) OVER (PARTITION BY t2.Campus_),2) AS Mean,
  PERCENTILE_DISC(t2.years_employed,0.50) OVER (PARTITION BY t2.Campus_) AS Median
FROM t2
WHERE t2.years_employed >= 0
ORDER BY Median

~~~
