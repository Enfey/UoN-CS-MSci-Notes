## Intro
- OS security tends to focus on *who can access what files*, largely treating files/data as opaque blobs at a higher level
- Database security on the other hand is concerned with the precise content and meaning of that data, whilst also providing restrictions. 
- Crucially, sensitive information can often be inferred even where direct access is denied, which is what makes database security a uniquely tricky problem. 

## CIA Triad Applied to Databases
- **Confidentiality**
	- Sensitive data must be protected from unauthorised disclosure
	- This also includes the limitation of inference; indirect methods of reading or understanding data one is forbidden to access must also be blocked.
- **Integrity**
	- Ensuring the data in a DB is accurate, complete, consistent, and valid.
	- Integrity has two sub-dimensions:
		- **Internal Consistency**
			- The database obeys its own rules - foreign keys are valid, values fall within allowed ranges, constraints aren't violated. This is enforced internally.
		- **External Consistency**
			- This is harder; it means that the data reflects reality.
			- A salary figure may be internally consistent, but externally wrong (doesn't match what employee is paid but is a valid value).
			- Ensuring external consistency generally requires human processes and auditing and cannot be fully automated.
- **Availability**
	- The data must be accessible when needed.
	- Naturally, this can be threatened by denial-of-service attacks, ransomware, or even poorly written queries that lock tables.

### SQL Security - Privileges and Access Control 
- SQL implements access control based on three entities:
	- **Users**
	- **Actions** (SELECT, INSERT, UPDATE, DELETE etc)
	- **Objects**(tables, views, columns)
- Users can invoke actions on objects
- Newly created objects are owned by the creator.
- Privileges can be granted, via the `GRANT` statement, for example:
```SQL
GRANT SELECT(EMPID, name)
ON Employees
TO Office_Staff
```
- Five elements define any privilege grant:
	1. **Granter**
		- The entity issuing the privileges
		- Can only grant privileges one actually possesses.
		- The original owner of an object has full privileges and can grant anything
		- Usually superuser who can grant anything like `root` in MySQL.
	2. **Grantee**
		- Who receives the privilege. Can be a specific user, role (named group of privileges) or in some systems `PUBLIC` which means everyone
	3. **Object**
		- What privilege applies to 
		- Can be very granular, as seen in the example above, permits `SELECT` only of specific columns
		- Can also grant stored procedures, and actions on views (as opposed to base table).
	4. **Action**
		- What the grantee is permitted to do e.g., `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `REFERENCES`, `EXECUTE`
		- Can be combined
	5. **Grantable**
		- Adding the `WITH GRANT OPTION` permits the grantee to pass that privilege onto others.
```SQL
GRANT SELECT ON Employees TO alice WITH GRANT OPTION; --Now alice can do: 
GRANT SELECT ON Employees TO bob;
```

### View-Based Security
- One of the most powerful security tools offered by SQL
- A view is essentially a stored query that looks like and behaves like a table:
```sql
CREATE VIEW IT_EMPLOYEES AS
	SELECT EmpID, NAME
	FROM Employees
	WHERE Department = 'IT'
WITH CHECK OPTION
```
- Rather than granting access to the raw `Employees` table which may contain salary, medical info, etc, we only expose 2 columns: `EmpID` and `Name`
- Users querying `IT_Employees` never even know the underlying table structure exists, let alone access it.
- Granting of privileges can be made over views as the object.
- In many cases, writes to a view are written back to the underlying base table; acting as a window into the original table. The DB engine translates operations on views into a write on the base table directly.
	- However, a view only writes back if it is simple enough such that the derived computation for the base table write is unambiguous about where the data being written should go.
	- A view is generally writable if it:
		- Draws from a single base table
		- Does not use aggregate functions (`SUM`, `COUNT`)
		- Does not use set operations
		- Does not use `GROUP BY` or `HAVING`
- The `WITH CHECK OPTION` for a writable view means that any `INSERT` or `UPDATE` through the vview must produce a row that the view itself could see:
	- `INSERT INTO IT_EMPLOYEES VALUES(99, 'Sarah', 'Finance)`
	- Would write back, but immediately disappear from the `IT_EMPLOYEES` table because the view filters for `Department = IT`
	- The `WITH CHECK OPTION` prevents this.

#### Why use views
- Principle of least privilege can be enforced via grants whilst also ensuring that sensitive data is not disclosed to those without the permissions to view it.
- Users querying views may never even know the underlying table structure exists, let alone access it.
- Writable views ensure data integrity
- Can adjust the visibility of data easily.

#### Why not use views
- `INSERT`/`UPDATE` actions depend on `WITH CHECK OPTION`, otherwise may be 'blind writes' where the result may not be visible in the local view.
- Can very quickly become inefficient.
- Completeness of data and its consistency are not achieved automatically, especially for complex views, where write-backs have to be done manually; this threatens data integrity, as there may be some data present on one view that is not written to the table it is derived from. 


### Statistical Database Security
- Statistical databases are useful for statistical analysis and generating aggregate data
- Where access to specific data is restricted, access to aggregates may still be permitted, to allow genuinely useful statistical queries whilst preventing individual-level inference.

#### Inference Attacks
- Since individual items are sensitive, we cannot permit direct access
- Statistical queries are indeed useful, using aggregates like `COUNT, SUM, AVG, MAX, MIN` etc but these queries can nonetheless reveal information about the data their results are predicated on. 
#### Salary example 1
- $S$ = the sum of all salaries in the department
- $T$ = the sum of all salaries for the department except for those who have "Department Head" as "Position"
- Boss' salary = $S-T$ 
- **Do not permit singleton sets to be returned** - it should be noted it is relatively impossible to fully restrict inference, but it should be mitigated.
#### Salary example 2
- $S$ = sum of all salaries
- $T$ = sum of all salaries of women, and anyone whose first name is albert
- $U$ = sum of all men's department salaries
- Albert's salary = $T+U - S$ 
	- Isolate albert as appear in both sets via this algebra
- **Don't allow conditions/operations that would uniquely identify someone**
#### Salary example 3
- $S$ = sum of all salaries
- Number of department heads named Albert is not allowed
- $T$ = sum of all salaries for those named Albert
- $U$ = sum of all salaries for department heads
- $V$ = sum of all salaries for those who are not department heads
- Albert's salary = $V + T + U - S$ 


### Defenses
- Any partition of a dataset into groups where one person falls into a uniquely identifying combination of those groups, can leak that persons data through aggregate queries. It is important to prevent the return of singleton sets and maintain this global awareness.
- **Data swapping**
	- Swap values between similar records such that the statistics of the dataset remain identical but any attempt to reconstruct an individual's value gets the wrong person's data instead.
	- Swaps need to be between records that are demographically similar e.g., same department, similar job title, similar seniority, otherwise aggregate queries that combine say, salary with those attributes start returning wrong answers, threatening external consistency/data integrity.
- **Noise addition**
	- Rather than swapping true values around, noise addition keeps the underlying data intact but perturbs the query results before returning them. 
	- Every aggregate output gets a small random adjustment
	- These errors compound algebraically, making complex queries which attempt to reconstruct an individual's data return information 'wrong enough' to be useless.
- **Table splitting**
	- Architectural defence. The idea is to separate identifying information from sensitive information at the database level. 
		![](Pasted%20image%2020260326152909.png)
	- Where `AnonID` is a random identifier that has no meaning outside the database. The mapping between `AnonID` and real identify it itself kept separately.
	- Better than views, because if someone can break the view e.g., SQL injection, privilege escalation, or a misconfigured grant, the full underlying data is exposed.
	- Full separation means that an attacker that gets info only against anonymous IDs and only gets half the data anyway. 
- **User tracking/query logging**
	- Monitor sequences of queries for patterns that look like inference attacks.
	- Computationally expensive but can detect recognisable query patterns e.g., operations that return singleton sets and flag them. 
	- Detects co-ordinated attacks e.g., an attacker repeatedly trying to uncover the perturbation applied to specific columns for $x$ demographic to recover specific data.


### SQL Injection
- It is common for user input to be read e.g., in web or application form as part of search, and then used within an SQL query
- Unexpected and potentially malicious user input is able to therefore able to completely malform the query, which can have catastrophic results.
- An SQL injection attack bears striking similarity to XSS attacks.
- A typical vulnerable search query may look like:
```SQL
$query = "SELECT * FROM Products WHERE pName LIKE '%" . $searchterm . "%'";
```
- If a user submits `; DROP TABLE Products; --` as their search term, the resulting query becomes:
```SQL
SELECT * FROM Products WHERE pName LIKE '%'; DROP TABLE Products;-- %'
```
- Comments out the trailing `%` leaving a valid two-state query.
- An application or website is vulnerable to injection if it doesn't filter specific/control characters that can be used to 'break out' of the query:
	- `'` - Represents the beginning or end of a string; by adding to input field e.g., username, terminates the string early such that the next word is read as a command instead of data.
	- `;` - Represents the end of a command. Tells the database the first command is finished, allowing the attacker to start a new, malicious one inside the original query.
	- `/*...*/` - Comment
	- `--` - Comment, both used to nullify the remaining parts of an SQL query so it doesn't cause a syntax error.

#### In-Band SQL Injection
- Most common form of SQL injection attack
- In-band SQL Injection is a type of SQL injection where the attacker receives the result as a direct response using the same communication channel to launch the attack. 
- The simplest type of in-band SQL injection is when the attacker is able to modify the original query and receive the direct results of the modified query. 
	![](Pasted%20image%2020260327011607.png)
- An error-based in-band SQL injection is a subtype of in-band SQL injections where the result returned to the attacker is a database error rather than just raw data/results. 
	- The consequences of this may be that the attacker can use the received error string to get information about type and version of database to try different attack techniques for specific schemas
	- Get information about the specific schema
	- May be able to manipulate the errors to exfiltrate data from the database
- A Union-based SQL Injection is a subtype of in-band SQL injection where the attacker uses the UNION SQL clause to receive a result that combines legitimate information with sensitive data.
	- The attacker must first determine the number and type of columns in the original query, then craft a matching `UNION SELECT` to extract arbitrary table data.
		- `UNION` permits another SELECT queries and appends the results to the original query
		- For a `UNION` query to work, naturally the individual queries must return the same number of columns, and the data types in each column must be compatible


#### Blind SQL Injection
- No data is returned directly to the application, so the attacker instead infers information by observing application behaviour. 
- **Boolean-based blind injection** sends queries that evaluate to true or false, watching for differences in the page response. 
	![](Pasted%20image%2020260327011753.png)
- **Time-based blind injection** is used where the only observable side-channel is response time, where conditional expressions such as `1 AND IF(SUBSTRING(password,1,1) > 'm', SLEEP(5), 0)` "Is the first char of the admin password > 'm'"
	- Usually need to know database being used and version.


#### Second Order SQL Injection
- Distributes the attack across separate moments in time
- An attacker utilises the database to store malicious data which is then intended to be used in further queries, typically used when know a lot about the processes, and data entry points are fairly sanitised for things like search
- E.g.,
	![](Pasted%20image%2020260327020207.png)
	![](Pasted%20image%2020260327020218.png)
	Then use update password on this:
	![](Pasted%20image%2020260327021043.png)
	The quote nullifies the original and ensures uniqueness, as well as the comment, so the actual password for the admin account is what is reset. This shows that sanitisation at entry is not the only thing necessary; one needs sanitisation regardless where input comes from. 

### Database Fingerprinting
- This refers to how an attacker figures out which database engine they're talking to.
- If injection is possible, an attacker can `UNION_SELECT` the version string (which each engine exposes via a different function e.g., @@version).
- Watching for errors triggered with certain syntax specific to DBMS will slowly reveal the DBMS and version in use.
- Format of error messages and the wording, and even error numbers, can point directly to the DBMS being used.
#### Preventative measures
- Parameterised queries
- Stored procedures
- Input validation
- Least privilege database accounts
 