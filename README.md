**University Course Enrollment & Performance Analysis**

**Course:** INSY 8311 - Database Development with PL/SQL  
**Student:** Ornella Marie Sandra Joy Ishimwe | 29598 | Group A  
**Instructor:** Eric Maniraguha  
**Submission Date:** February,07, 2026


  **Business Problem**
Business Context
The organization is a public university operating in the education sector.
The analysis is conducted for the Academic Affairs and Institutional Research Department.


**Data Challenge**

The university collects data on student enrollments and course registrations each semester, but
lacks structured insights into student participation, course demand, and enrollment trends over
time. Decision-makers struggle to identify inactive students, courses with low enrollment, and
changes in enrollment patterns across semesters.


**Expected Outcome**


The analysis aims to provide actionable insights on course popularity, student engagement,
and enrollment trends, enabling better academic planning, resource allocation, and curriculum
improvement.

**Success Criteria (Measurable Goals Using Window Functions)**

The analysis will achieve the following five measurable objectives:
1. Identify the top 5 most enrolled courses per semester using RANK()
2. Calculate running total enrollments per semester using SUM() OVER()
3. Measure semester-to-semester enrollment growth using LAG()
4. Segment students into four enrollment-activity quartiles using NTILE(4)
5. Compute three-semester moving averages of enrollments using AVG() OVER()

  **Database Schema**
1. **Students Table**

 
CREATE TABLE students (
student_id INT PRIMARY KEY,
full_name VARCHAR(100) NOT NULL,
faculty VARCHAR(50),
enrollment_year INT
);
Description:
Stores student demographic and academic information


2.**Courses Table**

CREATE TABLE courses (
course_id INT PRIMARY KEY,
course_name VARCHAR(100) NOT NULL,
department VARCHAR(50),
credit_units INT
);
Description:
Contains course details offered by the university


 3.**Enrollments Table**

 
CREATE TABLE enrollments (
enrollment_id INT PRIMARY KEY,
student_id INT,
course_id INT,
semester VARCHAR(20),
enrollment_date DATE,
CONSTRAINT fk_student
FOREIGN KEY (student_id)
REFERENCES students(student_id),
CONSTRAINT fk_course
FOREIGN KEY (course_id)
REFERENCES courses(course_id)
);
Description:
Represents course registrations and acts as the transaction table linking students and
courses.


 **Entity Relationship Diagram**


<img width="1080" height="586" alt="TogethaDrawing" src="https://github.com/user-attachments/assets/7ab7654e-bbde-4623-9240-47ca21e59a1e" />

**Tables**
1. **students** 
2. **courses** 
3. **enrollments** 



 SQL JOINs Implementation

 1. INNER JOIN - Valid Transactions**
```sql
SELECT
s.full_name,
c.course_name,
e.semester,
e.enrollment_date
FROM enrollments e
INNER JOIN students s ON e.student_id = s.student_id
INNER JOIN courses c ON e.course_id = c.course_id;
```
**Result:**
<img width="506" height="188" alt="1 join" src="https://github.com/user-attachments/assets/12acb3af-ee24-4e58-94ac-94961fd76781" />

**Insight:** 
Business Interpretation:
This query displays all confirmed enrollments where both student and course records exist.
It helps the university analyze active participation across courses and semesters.

 2. LEFT JOIN - Inactive Customers
```sql
SELECT
c.course_id,
c.course_name
FROM enrollments e
RIGHT JOIN courses c ON e.course_id = c.course_id
WHERE e.enrollment_id IS NULL;
```
**Result:** 
<img width="491" height="118" alt="2 join" src="https://github.com/user-attachments/assets/f2894d73-fcd4-4005-b1a7-6eae90d347a8" />

**Insight:** 
Business Interpretation:
These students may be inactive or at risk of dropping out and should be targeted for academic
advising

3. RIGHT JOIN
Purpose: Detect courses with no student enrollments
```sql
SELECT
c.course_id,
c.course_name
FROM enrollments e
RIGHT JOIN courses c ON e.course_id = c.course_id
WHERE e.enrollment_id IS NULL;
```
<img width="289" height="164" alt="4 join" src="https://github.com/user-attachments/assets/cea7397c-c339-4fa3-854f-37be86b4d809" />

Business Interpretation:
This query highlights an alignment or a mismatch between student faculties and the course
departments.
```sql
SELECT
a.student_id AS student_1,
b.student_id AS student_2,
a.semester
FROM enrollments a
JOIN enrollments b
ON a.semester = b.semester
AND a.student_id <> b.student_id;
```
<img width="290" height="176" alt="5 join" src="https://github.com/user-attachments/assets/753fb6bd-d6ae-4ba3-bd71-02098b196517" />

Business Interpretation:
This comparison helps analyze peer enrollment patterns within the same academic period

5.1 Ranking Functions
Use Case: Top-enrolled courses per semester.
 Rank courses by enrollment count per semester
```sql
SELECT
semester,
course_id,
COUNT(student_id) AS total_enrollments,
RANK() OVER (
PARTITION BY semester
ORDER BY COUNT(student_id) DESC
) AS course_rank
FROM enrollments
GROUP BY semester, course_id;
```
<img width="349" height="197" alt="1 group" src="https://github.com/user-attachments/assets/95139914-efeb-4f52-9453-2b36ebf7815d" />

Interpretation:
Courses are ranked based on enrollment volume within each semester, helping identify the
most popular courses.

5.2 Aggregate Window Functions
Use Case: Running total of enrollments.
 Running total of enrollments by semester
```sql
SELECT
semester,
COUNT(student_id) AS semester_enrollments,
SUM(COUNT(student_id)) OVER (
ORDER BY semester
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
) AS running_total_enrollments
FROM enrollments
GROUP BY semester;
```
<img width="399" height="172" alt="2 group" src="https://github.com/user-attachments/assets/7a29b765-260e-45a0-b3d3-8a41bd3fb032" />

Interpretation:
This shows cumulative enrollment growth over semesters, supporting long-term academic
planning

5.3 Navigation Functions
Use Case: Semester-to-semester enrollment growth.
 Compare enrollment changes between semesters  
```sql
SELECT
semester,
COUNT(student_id) AS total_enrollments,
COUNT(student_id) - LAG(COUNT(student_id))
OVER (ORDER BY semester) AS enrollment_growth
FROM enrollments
GROUP BY semester;
```
<img width="331" height="157" alt="3 groupe" src="https://github.com/user-attachments/assets/ff2c4a4f-02a1-456d-9f61-70bac52bed9e" />

Interpretation:
Positive values indicate enrollment growth, while negative values highlight declining interest.

5.4 Distribution Functions
Use Case: Student segmentation by enrollment activity.
 Segment students into quartiles based on enrollment count
```sql
SELECT
student_id,
COUNT(course_id) AS total_courses,
NTILE(4) OVER (ORDER BY COUNT(course_id) DESC) AS
activity_quartile
FROM enrollments
GROUP BY student_id;
```
<img width="386" height="167" alt="4group" src="https://github.com/user-attachments/assets/a4a13236-63b3-485a-a664-14cc5dcd024d" />

Interpretation:

Students are segmented into quartiles, enabling targeted academic support and engagement
strategies.


 **Recommendations**

 
Descriptive Analysis
Enrollment activity varies significantly by course and semester, with a small number of courses
attracting the majority of students.
Diagnostic Analysis
Low-enrollment courses and inactive students contribute to inefficient resource utilization and
reduced academic impact.
Prescriptive Analysis
The university should promote high-demand courses, revise underperforming courses, and
Provide targeted academic interventions for low-activity students.


 **References**

 

 
1. PostgreSQL Official Documentation – Window Functions


2. W3Schools SQL JOINs


3. Elmasri & Navathe (2016), Fundamentals of Database Systems


 **Academic Integrity Statement**

All sources were correctly referenced. The implementations and analyses are original contributions. No AI-generated content was used without proper attribution or modification.

**Signature:** Ishimwe Ornella Marie Sandra Joy  
**Date:** 07/02/2026

 **Contact**
**Email:** dukundanesandra51@gmail.com 
**GitHub:** Ishimwe12-ALT
