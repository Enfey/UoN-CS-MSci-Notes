# Intro
- OS sec, focus on who can access what files, treats data at higher leel
- Database security, concerned with the precise content and meaning of data, whilst also providing restrictions
- Rmember,. info can be inferred where direct access is denied, makes db security difficult

### CIA triad applied to databases
- **Confidentiality**
	- Sensitive data, protect, from unauth disclosure
	- Includes limitation of inference
- **Integrity**
	- Ensure DB is accurate, complete, consistent, valid
	- Integrity has **two** sub-dimensions:
		- **Internal Consistency**
			- Database obeys its own rules - foreign keys valid, values within ranges, constraints not violated
		- **External Consistency**
			- Harder; means data reflects reality
			- Salary figure may be internally consistent but not externally consistent
			- Usually require human process+auditing
- **Availability**
	- Data must be accessible when needed
	- E.g., ransomware, DoS, poorly written queries

## SQL Security
- SQL implements access control based on **three entities**
	- **Users**
	- **Actions(SELECT, UPDATE, DELETE)**
	- **Objects(tables, views, columns)**
- Users invoke actions on objects
- Newly created objects are owned by creator
- Privileges can be granted via `GRANT` statement
	- **Five** elements define any privilege granted:
		1. **Granter**
			- Entity issuing privilege; can only grant privs you possess
			- Original owner of obj has full privileges
			- Usually have superuser like root.
		2. **Grantee**
			- Who receives privilege. 
			- Can be user, role (group of privs), or everyone
		3. **Object**
			- What privilege applies to
			- Can be very granular, permits SELECT of only specific columns for example, on a DB view
		4. **Action**
			- What the grantee is permitted to do
			- Can be combined
		5. **Grantable**
			- Adding `WITH GRANT` allows grantee to further pass on this privilege.


### View-Based Security
- Powerful tool offered by SQL
- **Stored query** that **looks** and **behaves** like a table
- Rather than granting access to raw table, expose 2 columns perhaps, stored query becomes view that is then accessable
- People accessing the view never even know underlying table structure exists
- Can apply grant privileges regarding view as an object
- In many cases, writes to view are written back to underlying base table(integrity), acts as window to OG table. 
	- DB engine translates operations on views into a write on base table.
	- However, view only writes back if it is simple enough such that the derived computation for base table write is unambiguous about where data should go
		- View is usually wriable is derived from single base table, does not use aggregate functions or set operations, does not use GROUP BY. etc
- Can use `WITH CHECK OPTION` for a writable view; means any INSERT or UPDATE applied to the view, must produce a row that the view itself can see, blocked otherwise.


### Why use views
- **Principle of least privilege**
	- Enforce via grants and WITH CHECK, only read and modify what need to be able to
- **Confidentiality**
	- Views make it so that users who edit them and observe them are not aware of the underlying table structure(s), let alone able to access it. 
- **Writable views ensure data integrity**
	- Consistent

### Why not use views
- **Can quickly become inefficient**
- **Data inconsistency**
	- Complex SQL commands or views will not write back to main table automatically, inventing operational strain where there was none. 
- INSERT/UPDATE depends on WITH CHECK OPTION, otherwise have blind writes, where cannot see the result. 


## Statistical database security
- Used for statistical analysis and generating aggregate data
- Where access to specific data is restricted, access to aggregates permitted, allow useful statistical queries whilst preventing individual level inference

### Inference attacks
- Individual items are sensitive; we cannot permit direct access
- Statistical queries are useful indeed, but aggregates like `COUNT`, `SUM``MAX` min, if combined correctly can reveal info about the data the result is predicated on
- E.g., sum of all salaries in department query, sum of all salaries for the department except for department head, then subtract
	- **Do not permit singleton sets to be returned**
	- **Do not allow conditions/operations that would uniquely identify someone**
### Defenses
- Any partition of dataset into groups, where one person falls into unique combination of those two groups, can leak persons data through aggregate queries
	- Prevent ingleton ets being returned, prevent queries from being composed in a manner such that a person's information could be yielded.
- **Data swapping**
	- Swap values between similar records; stats = identical, but trying to reconstruct individual value gets wrong data.
	- Swaps need to be between records that are demographically similar to not compromise statistical significance and jeopardise external consistency.
- **Noise addition**
	- Rather than swapping true values, noise keeps underlying data intact, but perturbs query results.
	- Every aggregate output gets small random adjustment
	- Errors compound; large complex queries attempting to reconstruct info would be 'wrong enough'
- **Table splitting**
	- Separate identifying info, from sensitive info, at DB level
	- E.g., employees table with real IDs, instaead make 2 sep tables, with IDs just for the DB, one has queryable, non-identifying info like salaryband, YoE, other one has name, address, salary etc
	- Better than views, as if someone can break the view e.g., misconfigured grant, privilege escalation, full underlying data is exposed
		- Only get half here; and its against anon ids
- **Query loggin**
	- Monitor sequences of queries for patterns that look like inference attacks e.g., return singleton set, and flag
	- Can detect co-ordinated attacks, but is computationally expensive.


## SQL injection
- Common for user input to be read as part of search, then used, within SQL query
- Malicious user input can subvert the intention of OG query, which can have catastrophic results
- An SQL injection attack is similar in nature to XSS attacks. 
- A typical vulnerable search query allows a user to submit SQL syntax as part of their query unsanitised, and it is processed as part of, or in place of the original query. 
- An application is vulnerable to SQL injection if it doesn't filter specific control characters that can be used to **break out** of the query
	- `/**/`, `--` **Comments**
		- Nullify remaining part of query after injection, so no syntax error
	- `'` Represent beginning or end of string, terminate string early so next word is read as command
	- `;` same for commands, tells DB first command is over, allows attacker to insert their own, processed sequentially.

### In-band SQL injection
- Most common form of SQL injection
- In-band SQL injection = attacker receives the result as direct response in the same communication channel used to launch the attack
- Simplest type of in-band is where the attacker modifies the original query and receives the direct results of said modified query
#### Error-based In-band injection
- Subtype of in-band injection where the result returned is an error rather than raw data
	- Consequences may be that attacker can use error string to get info about type and version of database to try different attack techniques
	- Get info about schema
	- May be able to learn enough via errors to perform regular -in-band

#### Union-based in band injection
- Subtype of in-band injectiion where an attacker uses the UNION clause to receive a result that combines legit info with sensitive data.
	- Attacker must determine number + type of cols in OG query, then craft a matching `UNION` query to extract arbitrary table data
		- UNION permits other SELECT queries and appends the results
		- Individual queries must return same number of cols and data types must be compatible

### Blind SQL injection
- No data is returned to the communication channel so the attacker infers info by observing the application behaviour
	- **Boolean-based blind injection**
		- Sends queries that evaluate to true or false, watching for differences in response
	- **Time-based blind injection**
		- Used where the only observable side-channel is response time
		- Can craft queries to say determine first char of admin password if can craft injection, take certain amount off time with if statement, eventually enumerate password
			- Usually need to know DB version.


### Second order SQL injection
- Distributes the attack across separate moments in time
- An attacker utilises the database to store malicious data, which is then intended to be used in further queries
	- Used when know a lot about DB structure and processes, but data entry points are heavily sanitised

## Database fingerprinting
- Refers to how attacker figures out which database engine possible
- If union-based in-band injection possible, can do UNION_SELECT the version string, which each engine exposes via a function
- Watching for errors triggered with certain syntax, slowly reveal DBMS and version
- Format of error messages, error numbers, wording. 

## Prevention
- **Parameterised queries**
	- Most effective defence against SQL injection
	- Core idea, separate SQL structure from data injected into it
	- Query template is received by database engine, compiled, then user-supplied vals are slotted in afterward so they are not treated as SQL logic. 
	- Different SQL dialects, diff syntax for placeholders in query.
	- Attacks can only change what data the query operates on, and thus, this does not prevent **second-order SQL injection**
- **Stored procedures**
	- Named block of SQL, lives inside DB, called by application code, SQL logic pre-compiled and store server side, params passed separately; similar to the above. 
	- Additional benefit of abstraction layer between application and schema; don't see SQL query in full. 
- **Input validation**
	- Stringent set of rules, before reach query
	- **Whitelist** - define what is acceptable, reject everything else
	- **Blacklist** - less preferred, detect and block dangerous patterns. Best used as a secondary layer
		- Never used on own, easy to miss attack vector
- **Least privilege database accounts**
	- Web app doesn't need to DROP tables; have multiple DB accounts

