## Review Standards Established

### Key Patterns to Check:
- Functions with 50+ parameters (CRITICAL)
- SQL injection in dynamic queries
- Memory leaks (JSON objects, large datasets)
- Infinite loops (`while process.flag` patterns)
- Transaction management for 10K+ records

### Review Format:
- Excel CSV: Serial No, Component, Line, Category, Major, Remarks
- Verified line numbers only
- Focus on problematic code segments only
- Use grep search to verify line numbers

### Technology-Specific Issues:
- Infor LN 3GL/4GL: Check for excessive revision history
- Credit Card Processing: Security validation requirements
- IDM Integration: Batch processing for large datasets

## Baan 3GL Syntax Standards (FUNDAMENTAL)

### Required 3GL Syntax Elements:
1. **Comments:** Must use pipe symbol `|` (not // or /* */)
2. **Line continuation:** Use `^` symbol for long statements
3. **Case sensitivity:** Keywords are case-insensitive
4. **Character set:** Only use approved characters: `a-z A-Z 0-9 # $ ^ & * ( ) - _ + = { } [ ] | \ ; : " , . / < >`
5. **String constants:** Use double quotes with `""` for quote escaping
6. **Variable naming:** Must begin with letter, can include letters/numbers/_/.
7. **Data types:** Use approved types: long, double, string, table, domain
8. **String limits:** Maximum 1024 characters per string
9. **Table naming:** Must start with 't' (e.g., ttcibd001)
10. **Declaration syntax:** Follow [EXTERN|STATIC] <TYPE> <VARIABLE> [OPTION] pattern
11. **Array limits:** Maximum 4 dimensions
12. **String indexing:** Use semicolon syntax strg(2;3) for character ranges
13. **Multibyte:** Use MB flag for international character support
14. **Operators:** Use Baan-specific operators (<> for !=, & for concatenation, \ for integer division)
15. **Precedence:** Use parentheses to enforce operator precedence
16. **Control flow:** Use proper terminators (ENDIF, ENDCASE, ENDWHILE, ENDFOR)
17. **GOTO:** Allowed in 3GL but should be avoided; forbidden in 4GL
18. **Functions:** Use FUNCTION keyword, RETURN(value) syntax, proper parameter types
19. **Function returns:** Must be explicitly declared and use RETURN(value) format
20. **Preprocessor:** Available in 3GL only (NOT in 4GL script generation)

### Constants and Variables Standards:
- **String escaping:** Use `""` for quotes within strings (not \")
- **Numeric formats:** Support long, float, hexadecimal (0xFFF), scientific notation
- **Symbolic constants:** Use predefined FALSE, TRUE, PI where appropriate
- **Variable naming:** Descriptive names starting with letters
- **Default initialization:** Numbers=0, Strings="" (automatic)

### Data Types and Declarations Standards:
- **String length:** Never exceed 1024 character limit
- **Table names:** Must follow 't' prefix convention
- **Array syntax:** Use proper dimension syntax: `TYPE name(size)` or `TYPE name(dim1, dim2)`
- **Declaration options:** Use BASED, FIXED, MB appropriately
- **Scope modifiers:** Use EXTERN/STATIC when needed
- **Domain types:** Reference data dictionary domains correctly

### Arrays and Strings Standards:
- **Array dimensions:** Never exceed 4 levels maximum
- **String indexing:** Use semicolon syntax `strg(2;3)` for character ranges (not comma)
- **FIXED strings:** Understand auto-padding behavior with spaces
- **Variable strings:** Length matches content unless specifically indexed
- **Multibyte support:** Use MB flag for international character sets
- **BASED strings:** Use for advanced memory management scenarios
- **COMMON variables:** Use for cross-program shared variables

### Expressions and Operators Standards:
- **String concatenation:** Use `&` operator (not `+`)
- **Not equal:** Use `<>` operator (not `!=`)
- **Integer division:** Use `\` operator for integer division
- **Logical operators:** Use AND, OR, NOT (not &&, ||, !)
- **Operator precedence:** Use parentheses for clarity and correct evaluation
- **Conditional expressions:** Use `?:` ternary operator correctly
- **Division types:** Understand difference between `/` and `\` operators

### Control Flow Standards:
- **Conditional blocks:** Always use proper terminators (ENDIF, ENDCASE, ENDWHILE, ENDFOR)
- **CASE statements:** Include DEFAULT case for completeness
- **Loop types:** Choose appropriate loop construct (WHILE, FOR, REPEAT-UNTIL)
- **Loop control:** Use BREAK/CONTINUE for early exit/skip
- **FOR loops:** Use STEP parameter for custom increments
- **GOTO usage:** Allowed in 3GL but discouraged; forbidden in 4GL scripts
- **Nested structures:** Ensure proper nesting and termination

### Functions Standards:
- **Function syntax:** Use FUNCTION keyword with optional return type
- **Return statements:** Always use RETURN(value) with parentheses
- **Parameter types:** Use REF for reference parameters, CONST for read-only
- **Static variables:** Use STATIC for persistent function-level data
- **Prototypes:** Declare forward references when functions used before definition
- **Recursion:** Avoid due to limitations (no local vars/args in recursive functions)
- **Return types:** Explicitly declare return types for clarity
- **Function naming:** Follow standard variable naming conventions

### Preprocessor Standards (3GL Only):
- **Directive syntax:** Use `#` at start of line for preprocessor directives
- **Include files:** Use `<>` for system, `""` for local files
- **Macro definitions:** Use #define for constants, #undef to remove
- **Include guards:** Prevent multiple inclusions
- **4GL restriction:** Preprocessor NOT available in 4GL script generation
- **Compiler requirement:** Requires bic6.2 compiler support

## Architecture Standards: 3GL vs 4GL Environments

### 3GL Environment (Full Programming Language)
**Capabilities:**
- ✅ Complete language features including GOTO
- ✅ Preprocessor directives (#include, #define, #undef)
- ✅ Full function support with recursion (limited)
- ✅ Complete control flow constructs
- ✅ Advanced memory management (BASED, COMMON)

**Use Cases:** Complex business logic, system-level programming, full applications

### 4GL Environment (Restricted Programming Model)
**Limitations:**
- ❌ No GOTO statements (structured programming only)
- ❌ No preprocessor directives
- ❌ No db.commit() in DAL scripts
- ❌ Simplified function model
- ❌ Limited advanced features

**Use Cases:** Database access layers, simple business rules, report generation

### 3GL Code Review Checklist:
- [ ] All comments use `|` syntax
- [ ] Line continuations use `^` symbol appropriately
- [ ] Only approved character set used
- [ ] Proper separators (spaces, tabs, newlines)
- [ ] String constants use `""` for quote escaping
- [ ] Variable names follow naming rules
- [ ] Appropriate data types used (long, double, string, table, domain)
- [ ] Symbolic constants used where applicable
- [ ] String variables within 1024 character limit
- [ ] Table names start with 't'
- [ ] Array declarations use proper syntax
- [ ] Declaration options used appropriately
- [ ] Arrays do not exceed 4 dimensions
- [ ] String indexing uses semicolon syntax
- [ ] MB flag used for multibyte strings when needed
- [ ] BASED and COMMON used appropriately
- [ ] String concatenation uses `&` operator
- [ ] Not equal comparisons use `<>` operator
- [ ] Logical operators use AND/OR/NOT syntax
- [ ] Parentheses used for operator precedence clarity
- [ ] Control flow blocks properly terminated
- [ ] CASE statements include DEFAULT
- [ ] GOTO avoided (forbidden in 4GL, discouraged in 3GL)
- [ ] Appropriate loop types used
- [ ] Functions use FUNCTION keyword
- [ ] Return statements use RETURN(value) syntax
- [ ] Parameter types specified correctly (REF/CONST)
- [ ] Function return types explicitly declared
- [ ] Recursive functions avoided or used very carefully
- [ ] Preprocessor directives used correctly (3GL only)
- [ ] Include files use proper syntax and paths

## Infor LN 4GL Syntax Limitations (CRITICAL)

### FORBIDDEN Constructs - Will Cause Compilation Errors:
1. **No struct declarations** - Use individual variables instead
2. **No mod operator** - Use integer division logic: `remainder = value - ((value / divisor) * divisor)`
3. **No db.commit() calls in DAL scripts** - Forbidden in Data Access Layer scripts only
4. **No goto statements** - Use structured control flow (WHILE, FOR, IF-THEN-ELSE, ON CASE)
5. **No index() function** - Use pos() function for string searching
6. **No complex string functions** - No isalnum(), isdigit(), etc. Use len() and basic checks
7. **No modern array operations** - Use element-by-element logic for comparisons
8. **No preprocessor directives** - No #include, #define, #undef in 4GL script generation

### Required 4GL Syntax:
- SQL count queries: Use `count(*)` syntax
- Function return types: Must be explicitly declared with RETURN(value) format
- Memory cleanup: Keep functions simple without conditional logic
- String validation: Use `len()` and basic character checking
- Array processing: Use traditional loop structures
- Structured programming: Use only IF-THEN-ELSE, loops, and functions (no GOTO)

# 4GL Programming Standards and Patterns

## 4GL Runtime-Oriented Programming Standards

### Core Structure Requirements:
1. **PROGRAM structure:** Use PROGRAM as top-level container with runtime event management
2. **Event flow:** Use before.program, after.form.read, and explicit event calls
3. **Choice processing:** Use WHILE TRUE loop with input.choice() and on case choice
4. **Database binding:** Use db.bind() for runtime database connection
5. **Form processing:** Use read.form() and field iteration patterns

### Runtime Event Standards:

#### Program Initialization:
- **before.program:** Use for program setup and read.form() call
- **after.form.read:** Use for form initialization, references, and database binding
- **Background processing:** Include IF background THEN logic for background operations
- **Job processing:** Include IF job.process THEN logic with execute() calls

#### Choice Processing Standards:
- **Main loop:** Use WHILE TRUE for continuous user interaction
- **Input handling:** Use input.choice() to get user actions
- **Case structure:** Use on case choice with appropriate case statements
- **Event pattern:** Use before.xxx(), on.xxx(), after.xxx() for each operation
- **Update handling:** Check update.status before database operations

#### Database Operation Standards:
- **Runtime binding:** Use db.bind(tmain) for table binding
- **Update logic:** Use IF update.status AND choice <> delete pattern
- **Record operations:** Include get.ref.var(parent) and read record for background
- **DAL restriction:** NO db.commit() calls allowed in DAL scripts

#### Field Processing Standards:
- **Field iteration:** Use FOR EACH field ON form pattern
- **Field initialization:** Call init.field() and put.attributes() for each field
- **Index management:** Use change.to.start.index() appropriately
- **Form preparation:** Call init.form(), execute(start.event), before.form()

### Runtime Programming Patterns:

#### Standard Program Flow:
```baan
PROGRAM
{
    before.program
        read.form()

    after.form.read
        init.references()
        create.sql.queries()
        db.bind(tmain)
        
        FOR EACH field ON form
            init.field()
            put.attributes()
        ENDFOR
        
        init.form()
        before.form()

    WHILE TRUE
        input.choice()
        
        on case choice
            case operation
                before.operation()
                on.operation()
                after.operation()
                break
        endcase
    ENDWHILE
}
```

#### Background Processing Pattern:
```baan
after.form.read
    IF background THEN
        get.ref.var(parent)
        read record
    ENDIF
```

#### Job Processing Pattern:
```baan
after.form.read
    IF job.process THEN
        before.choice.run.job()
        execute(cont.process)
        after.choice.run.job()
        execute(end.program)
    ENDIF
```

### 4GL Runtime Programming Review Checklist:
- [ ] PROGRAM structure used as top-level container
- [ ] before.program includes read.form() call
- [ ] after.form.read includes init.references() and create.sql.queries()
- [ ] db.bind(tmain) used for database binding
- [ ] FOR EACH field ON form pattern used for field initialization
- [ ] WHILE TRUE loop used for main user interaction
- [ ] input.choice() used to get user actions
- [ ] on case choice structure used with appropriate cases
- [ ] before.xxx(), on.xxx(), after.xxx() pattern used for operations
- [ ] Background processing logic included where appropriate
- [ ] Job processing logic included where appropriate
- [ ] update.status checking implemented correctly
- [ ] DAL scripts do not contain db.commit() calls
- [ ] zoom.from.on.entry() called where appropriate
- [ ] execute() functions used for job processing

### Runtime Architecture Best Practices:
- **Explicit control:** Manage program flow explicitly through runtime calls
- **Event consistency:** Use consistent before/on/after pattern for all operations
- **Error handling:** Implement proper error handling in runtime event calls
- **Performance:** Optimize field iteration and database operations
- **Maintainability:** Structure runtime logic clearly and consistently
- **Documentation:** Document runtime event flow and choice handling logic

## Database Handling Standards (CRITICAL)

### Database Operation Requirements:
1. **Read operations:** Must use proper `read` syntax with direction keywords
2. **Performance:** Use `select` for specific fields instead of full record reads
3. **Status checking:** Always check `found` variable after read operations
4. **Index usage:** Specify appropriate indexes for performance
5. **Data modification:** Read record first before update/delete operations
6. **Transaction control:** Use begin/commit/rollback for atomic operations
7. **Concurrency:** Use `lock` keyword for exclusive access when needed
8. **Error handling:** Implement proper error handling with transaction rollback

### Baan 3GL SQL Syntax Standards:
- **General format:** Always use `tablename.fieldname` format (NO semicolons anywhere)
- **Select structure:** `select...from...where...order by...with retry` format
- **Required blocks:** `selectdo...selectempty...endselect` structure (mandatory)
- **Field references:** Always use full `tablename.fieldname`, never just `fieldname`
- **Transaction start:** `db.retry.point()` BEFORE any SQL operations
- **Retry detection:** `if db.retry.hit() then` for logging/messaging
- **Delayed locking:** `from tablename for update` in FROM clause
- **Ordering safety:** `order by field with retry [repeat last row]` when retry points present
- **Updates:** `db.update(tablename, DB.RETRY)` for all update operations
- **Retry check:** `if db.retry then return` after every update
- **Transaction end:** Explicit `commit.transaction()` or `abort.transaction()`
- **WHERE options:** Support for `=`, `<>`, `IN()`, `LIKE`, `INRANGE`, `EXISTS`
- **Aggregation:** `group by` required with aggregate functions, `having` for filtering

### Database Review Checklist:
- [ ] All field references use full `tablename.fieldname` format (never just `fieldname`)
- [ ] NO semicolons (`;`) after any SQL lines (SELECT, WHERE, COMMIT, UPDATE)
- [ ] `selectdo...selectempty...endselect` structure used correctly for all queries
- [ ] `selectempty` block handles no-results cases appropriately
- [ ] `db.retry.point()` placed BEFORE any SQL operations (never inside `selectdo`)
- [ ] `if db.retry.hit()` used for retry detection and logging
- [ ] `for update` used in FROM clause for delayed locking
- [ ] `order by...with retry` used when retry points are present
- [ ] `repeat last row` used for consistent retry behavior when needed
- [ ] `db.update(tablename, DB.RETRY)` used for all update operations
- [ ] `if db.retry then return` checked after every update
- [ ] Explicit `commit.transaction()` or `abort.transaction()` used to end transactions
- [ ] Aggregate functions used with required `group by` clause
- [ ] `having` clause used for filtering grouped results
- [ ] Advanced WHERE options used correctly (`IN`, `LIKE`, `INRANGE`, `EXISTS`)
- [ ] Wildcard selections use `tablename.*` format
- [ ] No standard SQL blocks (`BEGIN TRANSACTION`, `ROLLBACK`, `END`)
- [ ] No `using` clause unless explicitly required
- [ ] Database operations comply with script type restrictions
- [ ] No `db.commit()` calls in DAL scripts
- [ ] Proper WHERE conditions to avoid full table scans

### Database Performance Standards:
- **Index strategy:** Always specify indexes for large table access
- **Field selection:** Use SELECT for subset of fields to reduce I/O
- **Join optimization:** Structure joins to minimize record retrieval
- **Loop efficiency:** Use proper read direction and index ordering
- **Transaction size:** Keep transactions appropriately sized
- **Lock duration:** Minimize lock time to reduce contention

### Database Error Patterns to Flag:
❌ **Partial field references** using just `fieldname` instead of `tablename.fieldname`  
❌ **Semicolons (`;`)** after any SQL lines (SELECT, WHERE, COMMIT, UPDATE)  
❌ **Missing `selectdo...endselect`** structure for queries  
❌ **No `selectempty`** block for handling no-results cases  
❌ **Missing `db.retry.point()`** before transaction SQL operations  
❌ **No `if db.retry.hit()`** for retry detection and logging  
❌ **Wrong locking syntax** - not using `for update` in FROM clause  
❌ **Missing ordering safety** - no `order by...with retry` when retry points present  
❌ **Wrong update syntax** - not using `db.update(tablename, DB.RETRY)`  
❌ **Missing `if db.retry then return`** after updates  
❌ **Standard SQL transaction blocks** - using `BEGIN TRANSACTION`, `ROLLBACK`, `END`  
❌ **Missing transaction endings** - not using `commit.transaction()` or `abort.transaction()`  
❌ **Retry points inside `selectdo`** or SQL loops  
❌ **Aggregate functions without `group by`** clause  
❌ **Wrong wildcard syntax** - using `*` instead of `tablename.*`  
❌ **Using `using` clause** when not explicitly required  
❌ **Mixing standard SQL with Baan syntax** inconsistently  
❌ **`db.commit()` in DAL scripts** (forbidden - use `commit.transaction()`)  
❌ **Not handling `repeat last row`** when using consistent retry behavior  

### Database Best Practice Examples:

#### Correct Read-Only Query:
```baan
| Read-only operations don't need transactions
select tccom100.cust, tccom100.name, tccom100.city
from tccom100
where tccom100.stat = 10 and tccom100.balance > 1000.0

selectdo
    message("Customer: " & tccom100.name & " from " & tccom100.city)
selectempty
    message("No customers found")
endselect
```

#### Correct Transaction with Update:
```baan
| ✅ CORRECT: Complete Baan transaction structure
db.retry.point()

if db.retry.hit() then
    message("Retrying customer update transaction")
endif

select tccom100.cust, tccom100.name, tccom100.balance
from tccom100 for update
where tccom100.cust = "CUST001"
order by tccom100.cust with retry

selectdo
    tccom100.balance = tccom100.balance + 100.0
    tccom100.last_updated = current.date()
    
    db.update(tccom100, DB.RETRY)
    
    if db.retry then
        return
    endif
    
    commit.transaction()

selectempty
    abort.transaction()
endselect
```

#### Correct Aggregate Query with Grouping:
```baan
| ✅ CORRECT: Aggregation with required group by
select tccom100.city, count(*), sum(tccom100.balance), avg(tccom100.balance)
from tccom100
where tccom100.stat = 10
group by tccom100.city
having count(*) > 5

selectdo
    message("City: " & tccom100.city & 
            " Customers: " & str$(count(*)) &
            " Total: $" & str$(sum(tccom100.balance)))
selectempty
    message("No city data found")
endselect
```

#### Correct Advanced WHERE Clauses:
```baan
| ✅ CORRECT: Using IN, LIKE, and INRANGE
select tccom100.cust, tccom100.name
from tccom100
where tccom100.cust in ("C001", "C002", "C003")
and tccom100.name like "[Tt]est.*"
and tccom100.balance inrange 1000.0 and 10000.0

selectdo
    message("Found customer: " & tccom100.cust & " - " & tccom100.name)
selectempty
    message("No matching customers found")
endselect
```

## Integration Standards: 3GL Language + 4GL Programming

### Combined Review Approach:
1. **Language compliance:** Validate 3GL syntax and limitations
2. **Programming patterns:** Ensure proper 4GL event-driven structure
3. **Script type rules:** Verify appropriate script type usage
4. **Architecture alignment:** Check 3GL vs 4GL capability usage
5. **Performance optimization:** Consider both language and pattern efficiency

### Comprehensive Development Standards:
- Use 3GL language features within 4GL programming constraints
- Respect script type limitations (especially DAL restrictions)
- Implement event-driven patterns using proper 3GL syntax
- Maintain clear separation between language capabilities and programming models
- Follow both syntax rules and architectural patterns consistently

# Baan Code Review Standards

## SQL Standards

### Field Reference Requirements
- **ALWAYS** use full table.field references: `tccom100.name`
- **NEVER** use bare field names: `name` ❌
- **NO** semicolons after SQL statements
- **NO** `db.bind()` unless specifically mentioned

### SQL Query Structure
- Use `selectdo...selectempty...endselect` structure
- **NO** `using` clause unless explicitly told
- Handle empty results with `selectempty` block

### Transaction Management
- Start with `db.retry.point()` before SQL operations
- Use `for update` for record locking
- Use `db.update(table, DB.RETRY)` for modifications
- Check `if db.retry then return` after updates
- End with `commit.transaction()` or `abort.transaction()`
- **NEVER** use `begin transaction`, `rollback`, or semicolons

## Baan 4GL Structure Standards

### Event Block Structure ✅
**ALWAYS use proper event blocks, NOT function handlers:**

#### Field Event Handling
```baan
field.tablename.fieldname:
when.field.changes:
    |* Field change logic here
    table.calculated.field = table.field1 + table.field2
```

#### Table I/O Lifecycle Events
```baan
main.table.io:
before.write:
    |* Validation before database write
    if table.field < 0 then
        message("Validation error")
    endif

before.read:
    |* Logic before reading record

after.read:
    |* Logic after reading record
```

### Common 4GL Structure Mistakes ❌

#### DON'T Use Function-Style Handlers
```baan
|* ❌ WRONG - Don't write custom functions for events
function on.field.changed()
function before.save.record()
function after.field.update()
```

#### DON'T Use Procedural Main Program Structure
```baan
|* ❌ WRONG - Don't use procedural approach
function main()
{
    while true do
        input.choice()
        on case choice
        ...
    endwhile
}
```

### Correct 4GL Patterns ✅

#### Domain Declarations at Top Level
```baan
domain tcamnt total.amount
domain tccom.bpid business.partner.id
```

#### Direct SQL Result Assignment
```baan
select sum(tccom100.crdt):total.crdt.limit
from tccom100
where tccom100.stat = 1
selectdo
selectempty
    message("No results found")
endselect
```

## Variable and Data Type Standards

### Variable Naming
- Use descriptive names with dots: `total.credit.limit`
- Case-insensitive but use consistent casing
- Use underscores for readability: `business.partner.id`

### Data Type Usage
- Use `domain` for business-specific types
- Use `long` for integers, `double` for decimals
- Use `string(n)` with appropriate length
- Declare variables at appropriate scope level

## Control Flow Standards

### Conditional Logic
- Always use `if...then...endif` structure
- Use `elsif` for multiple conditions
- Include `else` clause when appropriate

### Loop Structures  
- Use `while...do...endwhile` for conditional loops
- Use `for...to...do...endfor` for counting loops
- Always include proper loop terminators

### Case Statements
- Use `on case...case...default...endcase` structure
- Include `default` case for completeness
- Use meaningful case values

## Error Handling Standards

### Message Display
- Use descriptive error messages
- Include context in error messages
- Use consistent message formatting

### Validation Patterns
- Perform validation in `before.write:` events
- Check business rules before database operations
- Provide clear feedback for validation failures

## Documentation Standards

### Comment Requirements
- Use `|*` for block comments
- Use `|` for line comments  
- Comment complex business logic
- Include purpose and assumptions

### Code Organization
- Group related functionality together
- Separate event blocks clearly
- Use consistent indentation
- Include meaningful section headers

---

## DLL (General Library) Review Standards

### DLL Implementation Standards:
1. **Naming Convention**: Use `<package><module>dll<dllName>` format (NO dots in script name)
2. **Function Naming**: Use `<package>.<module>.dll<dllName>.<function.name>` format (WITH dots in function name)
3. **Global Functions**: Must use `extern function` syntax for external accessibility
4. **Local Functions**: Use standard `function` syntax for internal helpers only
5. **Include Statement**: Must use `#pragma used dll <scriptname>` syntax
6. **Return Types**: Always specify explicit return types and use RETURN(value)

### DLL Review Checklist:
- [ ] Script name follows `<package><module>dll<dllName>` convention without dots
- [ ] Global function names use `<package>.<module>.dll<dllName>.<function.name>` format with dots
- [ ] `extern function` keyword used for all globally accessible functions
- [ ] Standard `function` keyword used for local helper functions only
- [ ] `#pragma used dll <scriptname>` included in consuming scripts
- [ ] Function parameters use appropriate types (REF/CONST)
- [ ] Return types explicitly declared for all functions
- [ ] `RETURN(value)` syntax used consistently
- [ ] Comprehensive error handling implemented in DLL functions
- [ ] DLL purpose and interface thoroughly documented
- [ ] Single responsibility principle followed (each DLL focuses on one area)
- [ ] No business logic specific to individual sessions in shared DLLs
- [ ] Proper separation between global and local functions maintained

### DLL Function Standards:
```baan
|* ✅ CORRECT: Global function with proper naming
extern function tcyesno tsext.dllvalidationUtils.validate.customer.credit(tccom.cust customer.id, tcamnt credit.amount)
{
    |* Function implementation
    return(tcyesno.yes)
}

|* ✅ CORRECT: Local helper function
function string format.timestamp(long timestamp.value) : string
{
    |* Local implementation
    return(formatted.date)
}

|* ❌ WRONG: Missing extern keyword for global function
function tcyesno tsext.dllvalidationUtils.validate.customer.credit()

|* ❌ WRONG: Wrong naming convention (no dots in script name, wrong function name format)
extern function tcyesno tsext_dll_validation_utils_validate_customer_credit()
```

### DLL Usage Standards:
```baan
|* ✅ CORRECT: Proper DLL include and usage
#pragma used dll tsextdllvalidationUtils

field.tccom100.crdt:
when.field.changes:
    if not tsext.dllvalidationUtils.validate.customer.credit(tccom100.cust, tccom100.crdt) then
        message("Credit limit validation failed")
    endif

|* ❌ WRONG: Missing pragma or wrong function call format
tsext_dll_validation_utils_validate_customer_credit(tccom100.cust, tccom100.crdt)
```

### DLL Architecture Review Points:
- [ ] Clear separation between validation, calculation, integration, and utility DLLs
- [ ] No circular dependencies between DLLs
- [ ] Proper error handling with meaningful error messages
- [ ] Resource cleanup in complex DLL functions
- [ ] Performance considerations for frequently called DLL functions
- [ ] Version compatibility maintained when updating DLLs
- [ ] Dependencies clearly documented
- [ ] Testing coverage for all DLL functions

---

## Standard Commands and Form Commands Review Standards

### Standard Commands Review Standards:
1. **Built-in Recognition**: Verify standard commands (ADD.SET, MODIFY.SET, DELETE.SET) used without definition
2. **Runtime Control**: Check proper use of `enable.commands()` and `disable.commands()` functions
3. **Timing Restrictions**: Ensure commands not controlled in `before.program` section
4. **Authorization Integration**: Verify DAL `method.is.allowed()` hook implementation
5. **User Feedback**: Confirm appropriate user messaging for command state changes

### Form Commands Review Standards:
1. **Manual Definition**: Verify form commands defined in session configuration, not code
2. **Function Implementation**: Check `extern function` with exact name match to form command
3. **Type Validation**: Confirm proper "Call Function" vs "Start Session" type usage
4. **Business Logic**: Validate appropriate business logic in form command functions
5. **Command Integration**: Check proper integration with `business.method.is.allowed()` DAL hook

### Command Control Review Checklist:

#### Standard Command Control:
- [ ] `enable.commands()` and `disable.commands()` functions used correctly
- [ ] Multiple commands enabled/disabled in single call where appropriate
- [ ] Standard command constants used (ADD.SET, MODIFY.SET, DELETE.SET)
- [ ] Command control logic based on data state and user permissions
- [ ] No command control attempted in `before.program` section
- [ ] Commands disabled/enabled in response to field changes and record state
- [ ] User authorization checked before enabling sensitive commands

#### Form Command Implementation:
- [ ] Form commands defined in session configuration (not in code)
- [ ] `extern function` defined with exact name matching form command
- [ ] Function name follows `command.<action>.<object>` convention
- [ ] Proper business logic validation before command execution
- [ ] User permission checking implemented where appropriate
- [ ] Error handling with meaningful user messages
- [ ] UI state updates after command execution (display() calls)
- [ ] Transaction management for database operations in commands

### DAL Integration Review Standards:

#### method.is.allowed() Implementation:
```baan
|* ✅ CORRECT: Standard command authorization
function extern long method.is.allowed(long method)
{
    on case method
    case DAL_NEW:
        if not check.user.permission("CREATE_CUSTOMERS") then
            return(0)  |* Disable ADD command
        endif
    case DAL_UPDATE:
        if not check.user.permission("MODIFY_CUSTOMERS") then
            return(0)  |* Disable MODIFY command
        endif
    case DAL_DELETE:
        if not check.user.permission("DELETE_CUSTOMERS") then
            return(0)  |* Disable DELETE command
        endif
    endcase
    
    return(1)  |* Allow operation
}
```

#### business.method.is.allowed() Implementation:
```baan
|* ✅ CORRECT: Form command authorization
function extern long business.method.is.allowed()
{
    if business.method = "command.post.transaction" then
        if tccom200.status <> "DRAFT" then
            return(0)  |* Disable if not draft
        endif
        
        if not check.user.permission("POST_TRANSACTIONS") then
            return(0)  |* Disable if no permission
        endif
    endif
    
    return(1)  |* Allow operation
}
```

### Command Usage Pattern Review:

#### Security-Based Command Control:
- [ ] Role-based command availability implemented
- [ ] User permissions checked before command execution
- [ ] Sensitive operations protected with appropriate authorization
- [ ] Audit logging implemented for critical command actions
- [ ] Error messages don't expose sensitive system information

#### Data-Driven Command Control:
- [ ] Command availability based on record status/state
- [ ] Field-dependent command enabling/disabling implemented
- [ ] Workflow state properly managed through command actions
- [ ] Business rule validation before command execution
- [ ] Proper state transitions maintained

#### UI Integration Review:
- [ ] Commands properly integrated with field change events
- [ ] Display updates after command execution
- [ ] Consistent user experience across command actions
- [ ] Proper error handling and user feedback
- [ ] Form state maintained correctly after command execution

### Command Implementation Patterns to Flag:

#### ❌ Problematic Patterns:
```baan
|* ❌ WRONG: Trying to control commands in before.program
before.program:
    enable.commands(ADD.SET)  |* Not allowed here

|* ❌ WRONG: Form command function name mismatch
|* Form command named "command.post.transaction" but function named differently
function extern command.post.trans()  |* Name must match exactly

|* ❌ WRONG: Missing extern keyword for form command function
function command.post.transaction()  |* Must be extern

|* ❌ WRONG: No error handling in form command
function extern command.post.transaction()
{
    tccom200.status = "POSTED"  |* No validation or error handling
}
```

#### ✅ Correct Patterns:
```baan
|* ✅ CORRECT: Proper command control in field events
field.tccom200.status:
when.field.changes:
    on case tccom200.status
    case "DRAFT":
        enable.commands("command.post.transaction", MODIFY.SET)
        disable.commands("command.approve.invoice")
    case "POSTED":
        enable.commands("command.approve.invoice")
        disable.commands("command.post.transaction", MODIFY.SET)
    endcase

|* ✅ CORRECT: Proper form command implementation
function extern command.post.transaction()
{
    |* Validation
    if not validate.transaction.data() then
        message("Transaction validation failed")
        return
    endif
    
    |* Business logic
    tccom200.status = "POSTED"
    tccom200.posted.by = logname$
    tccom200.posted.date = date.num()
    
    |* UI updates
    display("tccom200.status")
    display("tccom200.posted.by")
    display("tccom200.posted.date")
    
    |* User feedback
    message("Transaction posted successfully")
}
```

### Integration Review Standards:
- [ ] Commands properly integrated with DAL authorization hooks
- [ ] UI command control coordinated with DAL business logic
- [ ] Consistent command behavior across different session types
- [ ] Proper separation between UI command control and business authorization
- [ ] Form commands complement but don't duplicate standard commands
- [ ] Command naming follows consistent conventions
- [ ] Command functionality aligns with business requirements
- [ ] Error handling consistent across all command implementations

### Performance and Maintenance Review:
- [ ] Command functions optimized for quick execution
- [ ] Minimal database operations in command control logic
- [ ] Command state changes batched where possible
- [ ] Resource cleanup in complex command functions
- [ ] Command logic maintainable and well-documented
- [ ] Command dependencies clearly identified
- [ ] Testing coverage for all command scenarios
