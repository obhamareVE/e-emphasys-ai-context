# Infor LN Baan 3GL/4GL Quick Reference Guide

## Baan 3GL Syntax Fundamentals

### Character Set
**Allowed characters:**
- Letters: `a–z, A–Z`  
- Numbers: `0–9`
- Special: `# $ ^ & * ( ) - _ + = { } [ ] | \ ; : " , . / < >`

### Key Syntax Rules
✅ **Keywords:** Case-insensitive (MyVar = myvar = MYVAR)  
✅ **Comments:** Use pipe symbol `|` to begin comments  
✅ **Line continuation:** Use `^` to split long statements  
✅ **Separators:** Spaces, tabs, and newlines  

### Comment Examples
```3gl
| This is a single-line comment
| Multi-line comments require
| pipe symbol on each line
```

### Line Continuation Example
```3gl
long.variable.name = very.long.expression.that.needs ^
                     to.be.split.across.multiple.lines
```

## Baan 3GL Constants and Variables

### Constants

#### Numeric Constants
- **Long:** `12345`, `-530`, `0xFFF` (supports hexadecimal)
- **Float:** `123.56`, `0.8743`, `.8743`, `123.5e-5` (supports scientific notation)

#### String Constants  
- **Syntax:** Wrapped in double quotes
- **Quote escaping:** Use `""` to include quotes within strings
```3gl
"This is a string"
"A quote "" inside"
```

#### Symbolic Constants (Predefined)
- `FALSE` → `0`
- `TRUE` → `1`  
- `PI` → `3.141592653589793`

#### Enumerate/Set Constants
- **Source:** Defined via data dictionary
- **Usage:** Access via dot notation
```3gl
box_color = color.green
cf = feature.bold + feature.reverse
```

### Variables

#### Naming Rules
✅ **Must begin with:** Letter (a-z, A-Z)  
✅ **Can include:** Letters, numbers, `_`, `.`  
❌ **Cannot be:** Reserved words  
✅ **Best practice:** Names should indicate purpose  

#### Data Types
- `long` - Integer values
- `double` - Floating-point values  
- `string` - Text values
- `table` - Database table references
- `domain` - Domain-specific data types

#### Default Values
- **Numbers (long/double):** `0`
- **Strings:** `""` (empty string)

## Baan 3GL Data Types and Declarations

### Data Types (Detailed)
- `long` - Integer values
- `double` - Floating-point values  
- `string` - Text values **⚠️ MAX 1024 characters**
- `table` - Database table references (**must start with `t`**)
- `domain` - Domain-specific data types (from data dictionary)

### Declaration Syntax
```3gl
[EXTERN|STATIC] <TYPE> <VARIABLE> [OPTION], ...
```

#### Declaration Keywords
- **EXTERN** - External scope declaration
- **STATIC** - Static scope declaration

#### Declaration Options
- **BASED** - Based variable declaration
- **FIXED** - Fixed declaration
- **MB** - Multibyte support

### Declaration Examples

#### Table Declaration
```3gl
TABLE ttcibd001
```

#### Domain Declaration  
```3gl
DOMAIN tst_long myvar
```

#### Array Declarations
```3gl
LONG arr(10)                    | Single dimension array
STRING str_arr(20, 5)          | Multi-dimensional array
```

#### Variable Declarations
```3gl
LONG counter
DOUBLE price, total_amount
STRING customer_name(50), description(100)
```

## Baan 3GL Arrays and Strings

### Arrays
- **Maximum dimensions**: 4 levels
- **Declaration syntax**: `TYPE array_name(dim1, dim2, dim3, dim4)`

#### Array Examples
```3gl
LONG arr(10)                    | Single dimension - 10 integers
DOUBLE darr(5)                  | Single dimension - 5 doubles  
STRING sarr(10, 5)              | 2D array - 5 strings of 10 chars each
```

### String Operations

#### String Indexing (Unique Syntax)
```3gl
strg(2;3) = "Hel"              | Sets characters 2-4 to "Hel"
```
**⚠️ Uses semicolon (;) not comma for character ranges**

#### FIXED vs Variable Strings
- **FIXED**: Always padded with spaces to declared length
- **Variable**: Length matches content (unless indexed)

#### BASED Strings
```3gl
BASE b AT a(3)                  | Based string declaration
```

#### Multibyte Support
```3gl
STRING mb_str(10,5) MB          | Multibyte string for international chars
```

#### COMMON Strings (Cross-Program)
```3gl
STRING var(10) BASED            | Declare based string
COMMON var                      | Make available across programs
```

### String and Array Best Practices
✅ **Array dimensions**: Never exceed 4 levels  
✅ **String indexing**: Use semicolon syntax for character ranges  
✅ **Multibyte**: Use MB flag for international applications  
✅ **Memory management**: Use BASED for advanced memory control  
✅ **Cross-program**: Use COMMON for shared variables  

## Baan 3GL Expressions and Operators

### Operator Categories

#### Arithmetic Operators
- `+` Addition
- `-` Subtraction  
- `*` Multiplication
- `/` Division
- `\` Integer division (backslash)
- `&` String concatenation AND arithmetic

#### Relational Operators
- `=` Equal
- `<>` Not equal (**⚠️ NOT `!=`**)
- `>` Greater than
- `<` Less than
- `>=` Greater than or equal
- `<=` Less than or equal

#### Logical Operators
- `AND` Logical AND
- `OR` Logical OR  
- `NOT` Logical NOT

#### Conditional Operator
- `?:` Ternary conditional (condition ? true_value : false_value)

### Operator Precedence (Highest to Lowest)
```3gl
Highest    - (negation)
           * / \ 
           & 
           + - 
           = > < >= <= <> 
           NOT 
           AND 
Lowest     OR
```

**⚠️ Use parentheses `()` to enforce specific precedence**

### Expression Examples
```3gl
doub = 45 / 30.0               | Result: 1.5 (float division)
strg = "hel" & "lo"            | Result: "hello" (concatenation)
result = age >= 18 ? "adult" : "minor"  | Conditional expression
valid = (x > 0) AND (y < 100)  | Logical combination
```

### Operator Best Practices
✅ **String concatenation**: Use `&` operator (not `+`)  
✅ **Not equal**: Use `<>` operator (not `!=`)  
✅ **Precedence**: Use parentheses for clarity  
✅ **Division**: Be aware of integer vs float division  
✅ **Logical operators**: Use AND/OR (not &&/||)  

## Baan 3GL Control Flow

### Conditional Statements

#### IF-THEN-ELSE
```3gl
IF condition THEN
    | statements
ELSE
    | statements  
ENDIF
```

#### ON CASE (Switch-like)
```3gl
ON CASE expression
CASE value1:
    | statements
    BREAK
CASE value2:
    | statements
    BREAK
DEFAULT:
    | default statements
ENDCASE
```

### Loop Constructs

#### WHILE Loop (Pre-condition)
```3gl
WHILE condition
    | statements
ENDWHILE
```

#### FOR Loop (Counter-based)
```3gl
FOR i = 1 TO 10 STEP 2
    | statements
ENDFOR
```

#### REPEAT-UNTIL Loop (Post-condition)
```3gl
REPEAT
    | statements
UNTIL condition
```

### Jump Statements

#### GOTO (3GL Only)
```3gl
GOTO label
| ...
label:
    | statements
```
**⚠️ GOTO allowed in 3GL but forbidden in 4GL scripts**

#### Loop Control
- `BREAK` - Exit loop immediately
- `CONTINUE` - Skip to next iteration

### Control Flow Best Practices
✅ **Termination**: Always use proper terminators (ENDIF, ENDCASE, ENDWHILE, ENDFOR)  
✅ **CASE structure**: Always include DEFAULT case for completeness  
✅ **FOR loops**: Use STEP parameter for custom increments  
✅ **Loop control**: Use BREAK/CONTINUE for early exit/skip  
⚠️ **GOTO usage**: Allowed in 3GL but avoid in 4GL scripts  

### Control Flow Examples
```3gl
| Conditional example
IF age >= 18 THEN
    status = "adult"
ELSE
    status = "minor"
ENDIF

| Case example  
ON CASE day_type
CASE 1:
    day_name = "weekday"
    BREAK
CASE 2:
    day_name = "weekend"
    BREAK
DEFAULT:
    day_name = "unknown"
ENDCASE

| Loop examples
FOR counter = 1 TO 100 STEP 5
    | Process every 5th item
ENDFOR

WHILE records_remaining > 0
    | Process records
    records_remaining = records_remaining - 1
ENDWHILE
```

## Baan 3GL Functions

### Function Declaration Syntax
```3gl
FUNCTION [TYPE] function_name([arguments])
{
    [STATIC] variable_declarations
    | function body
    RETURN(value)
}
```

### Parameter Types

#### Value Parameters (Pass by Value)
```3gl
FUNCTION LONG calculate(LONG input_value)
```

#### Reference Parameters (Pass by Reference)
```3gl
FUNCTION VOID update_buffer(REF STRING buffer())
```

#### Constant Parameters (Read-only)
```3gl
FUNCTION VOID display_message(CONST STRING message())
```

### Static Variables in Functions
```3gl
FUNCTION LONG get_counter()
{
    STATIC LONG counter = 0
    counter = counter + 1
    RETURN(counter)
}
```

### Function Prototypes (Forward Declarations)
```3gl
| Prototype declaration (if function used before definition)
FUNCTION LONG calculate_total(LONG amount)

| Later in code...
FUNCTION LONG calculate_total(LONG amount)
{
    RETURN(amount * 1.1)
}
```

### Recursive Functions ⚠️ **LIMITED SUPPORT**
- **Restriction**: No local variables or arguments allowed in recursive functions
- **Use carefully**: Very limited compared to modern languages

### Function Examples
```3gl
| Simple function with return value
FUNCTION LONG add_numbers(LONG a, LONG b)
{
    RETURN(a + b)
}

| Function with reference parameter
FUNCTION VOID format_string(REF STRING output(), CONST STRING template())
{
    | Modify output string using template
    output = template & " formatted"
    RETURN()
}

| Function with static variable
FUNCTION LONG sequence_number()
{
    STATIC LONG sequence = 0
    sequence = sequence + 1
    RETURN(sequence)
}
```

### Function Best Practices
✅ **Return statements**: Always use `RETURN(value)` with parentheses  
✅ **Parameter types**: Use REF for modifiable parameters, CONST for read-only  
✅ **Static variables**: Use for persistent function-level data  
✅ **Prototypes**: Declare before use if function called early  
⚠️ **Recursion**: Avoid or use very carefully due to limitations  
✅ **Return types**: Explicitly declare return types for clarity  

## Baan 3GL Preprocessor and Compiler

### Preprocessor Directives (3GL Only)
⚠️ **NOT AVAILABLE in 4GL script generation**

#### Include Files
```3gl
#include "local_file.h"        | Local file with standard redirection
#include <system_file.h>       | System file from $BSE/include<rel.number>
```

#### Macro Definitions
```3gl
#define MAX_SIZE 1000          | Define constant macro
#define DEBUG_MODE 1           | Define debug flag
#undef DEBUG_MODE              | Undefine macro
```

### Include Search Paths
- **`<>`**: System includes from `$BSE/include<rel.number>`
- **`""`**: Local files following standard file redirection rules

### Preprocessor Requirements
- **Directive syntax**: Use `#` at the start of line
- **Compiler support**: Active in bic6.2 compiler only
- **3GL restriction**: Available in 3GL scripts only
- **4GL limitation**: ⚠️ **NOT AVAILABLE in 4GL script generation**

### Preprocessor Examples
```3gl
| System include
#include <standard_defs.h>

| Local include  
#include "my_constants.h"

| Macro definitions
#define PI 3.141592653589793
#define MAX_RECORDS 10000
#define VERSION "1.0"

| Conditional compilation
#define DEBUG 1
#ifdef DEBUG
    | Debug code here
#endif

| Cleanup
#undef DEBUG
```

### Preprocessor Best Practices
✅ **Include guards**: Use to prevent multiple inclusions  
✅ **Macro naming**: Use UPPERCASE for constants  
✅ **Local vs system**: Use `""` for project files, `<>` for system files  
⚠️ **4GL incompatibility**: Remember preprocessor not available in 4GL  
✅ **Compiler version**: Ensure bic6.2 compiler support  

## Architecture Summary: 3GL vs 4GL

### 3GL Capabilities (Full Language)
✅ **GOTO statements** - Jump control available  
✅ **Preprocessor directives** - #include, #define, #undef  
✅ **Full function support** - Including recursion (limited)  
✅ **Complete control flow** - All constructs available  

### 4GL Limitations (Restricted Environment)
❌ **No GOTO statements** - Must use structured control flow  
❌ **No preprocessor** - No #include, #define, #undef  
❌ **No db.commit() in DAL scripts** - Transaction restrictions  
❌ **Limited function features** - Simplified programming model  

## Critical "DO NOT USE" List - Will Cause Compilation Errors
❌ **struct declarations** → Use individual variables  
❌ **mod operator** → Use: `remainder = value - ((value / divisor) * divisor)`  
❌ **db.commit() in DAL scripts** → Forbidden only in Data Access Layer scripts  
❌ **goto statements** → Use structured error handling  
❌ **index() function** → Use pos() function instead  
❌ **isalnum(), isdigit()** → Use len() and basic checks  
❌ **Complex array operations** → Use element-by-element logic  

## Important Distinction: DAL vs Other Scripts
**DAL Scripts (Data Access Layer):**
- ❌ Cannot use `db.commit()` - Transaction management handled externally
- Usually have specific naming patterns or are in DAL directories
- Focus on data access operations only

**Other 4GL Scripts:**
- ✅ Can use `db.commit()` for transaction management
- Business logic, reports, interfaces, etc.
- Full transaction control allowed

## Required 4GL Syntax Patterns
✅ **SQL count:** `count(*)` syntax  
✅ **Function returns:** Must be explicitly declared  
✅ **Memory cleanup:** Simple functions without conditionals  
✅ **String validation:** Use `len()` and basic character checking  
✅ **Array processing:** Traditional loop structures  

## Pre-Implementation Checklist
- [ ] All solutions validated against 4GL limitations
- [ ] Memory cleanup functions planned
- [ ] Database operations checked for compatibility
- [ ] Function parameter counts reasonable (<10)
- [ ] No forbidden constructs in proposed code

## Error Resolution Priority
1. **Syntax Errors** - Remove forbidden constructs
2. **Database Operations** - Fix SQL and database calls
3. **Function Logic** - Simplify complex operations
4. **Validation** - Test against 4GL standards

## Memory Management Pattern
```4gl
function cleanup.allocated.memory()
{
    if (memory.allocated) {
        mem.free(memory.ptr)
        memory.allocated = false
    }
}
```

## String Operations Pattern
```4gl
| Instead of: if (isalnum(string.value))
if (len(string.value) > 0) {
    |* Basic string validation
}

| Instead of: index(string.value, "pattern")
position = pos(string.value, "pattern")
```

## Updated Files for Consistent Results
- **review-standards.md** - Contains all 4GL limitations and checklist
- **workflow-guide.md** - Includes 4GL-specific workflow steps
- **ai-prompts.md** - Has 4GL-aware prompts for consistent AI responses

## Key Success Factors
1. **Conservative Approach** - Use proven 4GL patterns only
2. **Incremental Testing** - Validate syntax in small sections
3. **Memory Management** - Always plan cleanup functions
4. **Documentation** - Keep revision history separate from active code
5. **Compilation Validation** - Always verify code compiles without errors

*Last Updated: Based on pext3440.bc compilation error resolution* 

# 4GL Programming Features and Patterns

## 4GL Programming Overview

### Programming Model
4GL (Fourth Generation Language) in BaanERP uses a **runtime-oriented architecture** that:
- Uses explicit procedural event management rather than declarative sections
- Manages program flow through runtime functions and loops
- Handles user interaction through choice input and case processing
- Integrates database operations through runtime binding

### Primary Capabilities
✅ **Runtime Event Management** - Explicit before/after event calls  
✅ **Form Processing** - Runtime form reading and field initialization  
✅ **Database Binding** - Runtime database binding and record management  
✅ **Choice Processing** - User input through choice loops and case handling  

## 4GL Runtime-Oriented Structure

### Core Program Structure
```baan
PROGRAM
{
    before.program
        | Program initialization
        read.form()

    after.form.read
        | Form setup and initialization
        init.references()
        create.sql.queries()
        db.bind(tmain)
        
        | Field initialization
        FOR EACH field ON form
            init.field()
            put.attributes()
        ENDFOR
        
        | Form preparation
        init.form()
        execute(start.event)
        before.form()

    | Main user interaction loop
    WHILE TRUE
        input.choice()

        IF update.status AND choice <> delete THEN
            on.update.db()
        ENDIF

        on case choice
            case new
                before.new()
                on.new()
                after.new()
                break

            case modify
                before.modify()
                on.modify()
                after.modify()
                break

            case delete
                before.delete()
                on.delete()
                after.delete()
                break

            case exit
                on.exit()
                break
        endcase
    ENDWHILE
}
```

## Runtime Event Management

### Core Runtime Events
- **before.program** - Program initialization and setup
- **after.form.read** - Post-form reading initialization
- **before.xxx()** - Pre-operation event handling
- **on.xxx()** - Main operation processing
- **after.xxx()** - Post-operation cleanup and updates

### Runtime Functions
- **read.form()** - Read form definition and structure
- **init.references()** - Initialize reference data and lookups
- **create.sql.queries()** - Prepare SQL queries for execution
- **db.bind(tmain)** - Bind database table to program
- **input.choice()** - Get user choice/action input
- **execute()** - Execute specific program functions

### Background and Job Processing
```baan
after.form.read
    IF background THEN
        get.ref.var(parent)
        read record
    ENDIF

    IF job.process THEN
        before.choice.run.job()
        execute(cont.process)
        after.choice.run.job()
        execute(end.program)
    ENDIF
```

## Choice Processing Pattern

### Standard Choice Handling
```baan
WHILE TRUE
    input.choice()

    on case choice
        case new
            before.new()
            on.new()
            after.new()
            break

        case modify
            before.modify()
            on.modify()
            after.modify()
            break

        case delete
            before.delete()
            on.delete()
            after.delete()
            break

        case save
            before.save()
            on.save()
            after.save()
            break

        case exit
            on.exit()
            break

        default
            | Handle unexpected choices
            break
    endcase
ENDWHILE
```

## Field Processing Pattern

### Runtime Field Initialization
```baan
FOR EACH field ON form
    init.field()
    put.attributes()
ENDFOR

change.to.start.index()
```

## Database Operations

### Database Access Fundamentals
All database operations access tables defined in the **Baan Data Dictionary**. The system automatically sets the `found` variable after read operations.

### Baan 3GL SQL Complete Specification

#### General SQL Rules
✅ **Always use `tablename.fieldname` format** - e.g., `tccom100.name`  
✅ **NO semicolons (`;`)** at the end of any SQL line  
✅ **Use `selectdo`/`selectempty`/`endselect` structure** for all queries  
✅ **Retry points BEFORE SQL** - never inside `selectdo`  
✅ **NO standard SQL blocks** - no `BEGIN TRANSACTION`, `ROLLBACK`, `END`  

#### Complete SQL Structure Template
```baan
db.retry.point()

if db.retry.hit() then
    message("Retrying transaction")
endif

select tablename.field1, tablename.field2
from tablename [for update]
where condition
[order by field [asc|desc] [with retry [repeat last row]]]
[group by field]
[having condition]
[as prepared set] [with n rows]

selectdo
    | Update logic here
    tablename.field = new_value
    db.update(tablename, DB.RETRY)
    
    if db.retry then
        return
    endif
    
    commit.transaction()

selectempty
    abort.transaction()
endselect
```

#### SELECT Clause Options
```baan
| Individual fields
select tccom100.cust, tccom100.name, tccom100.city
from tccom100
where tccom100.stat = 10

| Wildcard with table scope
select tccom100.*
from tccom100
where tccom100.cust = "CUST001"

| Aggregate functions
select tccom100.city, count(*), sum(tccom100.balance), avg(tccom100.balance)
from tccom100
group by tccom100.city
having count(*) > 5
```

#### WHERE Clause Advanced Options
```baan
| Comparison operators
select tccom100.cust, tccom100.balance
from tccom100
where tccom100.balance >= 1000.0 and tccom100.stat <> 99

| IN sets
select tccom100.cust, tccom100.name
from tccom100
where tccom100.cust in ("C001", "C002", "C003")

| LIKE with wildcards
select tccom100.cust, tccom100.name
from tccom100
where tccom100.name like "[Tt]est.*"

| BETWEEN / INRANGE
select tccom100.cust, tccom100.name
from tccom100
where tccom100.cust inrange "C001" and "C100"

| EXISTS subqueries
select tccom100.cust, tccom100.name
from tccom100
where exists (select tcord100.cust 
              from tcord100 
              where tcord100.cust = tccom100.cust)
```

#### Ordering and Retry Safety
```baan
| Required when retry points present
select tccom100.cust, tccom100.name
from tccom100
where tccom100.stat = 10
order by tccom100.cust with retry

selectdo
    message("Processing: " & tccom100.name)
selectempty
    message("No customers found")
endselect
```

```baan
| Repeat last row for consistent retry behavior
select tccom100.cust, tccom100.balance
from tccom100 for update
where tccom100.stat = 10
order by tccom100.cust with retry repeat last row

selectdo
    tccom100.balance = tccom100.balance * 1.05
    db.update(tccom100, DB.RETRY)
    
    if db.retry then
        return
    endif
    
    | Commit inside SELECTEOS to avoid rebuilding set
    commit.transaction()

selectempty
    abort.transaction()
endselect
```

### Data Modification

#### Insert Records
```baan
tccom100.cust = "CUST001"
tccom100.name = "New Customer"
tccom100.city = "New York"
insert tccom100
```

#### Update Records (with Transaction)
```baan
| Must use transaction for updates
db.retry.point()

if db.retry.hit() then
    message("Retrying update...")
endif

select tccom100.cust, tccom100.name
from tccom100 for update
where tccom100.cust = "CUST001"

selectdo
    tccom100.name = "Updated Name"
    db.update(tccom100, DB.RETRY)
    
    if db.retry then
        return
    endif
    
    commit.transaction()
selectempty
    abort.transaction()
endselect
```

#### Delete Records (with Transaction)
```baan
| Must use transaction for deletes
db.retry.point()

if db.retry.hit() then
    message("Retrying delete...")
endif

select tccom100.cust
from tccom100 for update
where tccom100.cust = "CUST001"

selectdo
    delete tccom100
    commit.transaction()
selectempty
    abort.transaction()
endselect
```

### Transaction Management (Baan Specific)

#### Correct Transaction Structure
```baan
| Step 1: Start transaction with retry point (BEFORE any SQL)
db.retry.point()

| Step 2: Check for retry and provide logging
if db.retry.hit() then
    message("Retrying transaction...")
endif

| Step 3: Select with delayed lock for updates
select tccom100.cust, tccom100.name, tccom100.balance
from tccom100 for update
where tccom100.cust = "CUST001"

selectdo
    | Step 4: Perform updates with retry enabled
    tccom100.balance = tccom100.balance + 100.0
    db.update(tccom100, DB.RETRY)
    
    | Step 5: Check for retry after update
    if db.retry then
        return
    endif
    
    | Step 6: Commit successful transaction
    commit.transaction()

selectempty
    | Step 6: Abort if record not found
    abort.transaction()
endselect
```

#### Multi-Table Transaction
```baan
db.retry.point()

if db.retry.hit() then
    message("Retrying multi-table transaction...")
endif

| Update customer record
select tccom100.cust, tccom100.balance
from tccom100 for update
where tccom100.cust = "CUST001"

selectdo
    tccom100.balance = tccom100.balance - 500.0
    db.update(tccom100, DB.RETRY)
    
    if db.retry then
        return
    endif
    
    | Insert order record
    tcord100.order_num = "ORD-001"
    tcord100.cust = tccom100.cust
    tcord100.amount = 500.0
    tcord100.status = "ACTIVE"
    
    insert tcord100
    
    commit.transaction()

selectempty
    abort.transaction()
endselect
```

### Baan 3GL SQL Best Practices
✅ **Always use `tablename.fieldname` format** - never just `fieldname`  
✅ **NO semicolons (`;`)** at the end of any SQL line  
✅ **Use `selectdo...selectempty...endselect`** structure for all queries  
✅ **Place `db.retry.point()` BEFORE SQL** - never inside `selectdo`  
✅ **Check `if db.retry.hit()`** for retry detection and logging  
✅ **Use `for update` in FROM clause** for delayed locking  
✅ **Use `db.update(table, DB.RETRY)`** for safe updates  
✅ **Check `if db.retry then return`** after every update  
✅ **Always end with `commit.transaction()` or `abort.transaction()`**  
✅ **Use `order by...with retry`** when retry points are present  
✅ **Use `repeat last row`** for consistent retry behavior  
✅ **Use aggregate functions with `group by`** for summaries  
✅ **Use advanced WHERE options** - `IN`, `LIKE`, `INRANGE`, `EXISTS`  
✅ **Handle empty results** with `selectempty` block  
✅ **Commit inside SELECTEOS** when using `repeat last row`  

### Runtime Database Integration
```baan
after.form.read
    db.bind(tmain)  | Bind main table
    
    IF update.status AND choice <> delete THEN
        on.update.db()
    ENDIF
```

### Baan 3GL SQL Restrictions
❌ **No semicolons (`;`)** after SELECT, WHERE, COMMIT, or UPDATE  
❌ **No standard SQL blocks** - NO `BEGIN TRANSACTION`, `ROLLBACK`, `END`  
❌ **No partial field references** - Always use `tablename.fieldname` format  
❌ **No retry points inside `selectdo`** - Place `db.retry.point()` before SQL  
❌ **No `db.commit()`** in DAL scripts (use `commit.transaction()` instead)  
❌ **No skipping transaction endings** - Must use `commit.transaction()` or `abort.transaction()`  
❌ **No `using` clause** unless explicitly instructed  
❌ **No `db.bind()` unless specifically mentioned**  
❌ **No mixing standard SQL with Baan syntax** - Use Baan 3GL format consistently

## Runtime-Oriented Best Practices
✅ **Explicit Event Management**: Use before/on/after function calls  
✅ **Runtime Binding**: Use db.bind() for database connections  
✅ **Choice Loop**: Use WHILE TRUE with input.choice() for user interaction  
✅ **Case Processing**: Use on case choice for action handling  
✅ **Background Support**: Include background and job processing logic  
✅ **Field Iteration**: Use FOR EACH field ON form for field processing  

## Script Type Selection Guide
- **UI Scripts**: Use runtime-oriented structure for screen behavior and user interactions
- **Reports**: Use runtime processing with report-specific choice handling  
- **DAL Scripts**: Use runtime database operations (no db.commit() allowed)
- **STP Scripts**: Use runtime job processing with execute() functions 

---

## Standard Commands and Form Commands

### Standard Commands Overview
**Standard Commands** are built-in commands available by default in session definitions. These include common operations like `ADD.SET`, `MODIFY.SET`, `DELETE.SET` that users can trigger from the UI.

#### Key Characteristics:
- ✅ **Pre-defined**: Exist by default, no definition required
- ✅ **Built-in functionality**: Automatically handled by the system
- ✅ **Runtime control**: Can be enabled/disabled dynamically
- ✅ **User triggered**: Activated through UI buttons or menu items

#### Common Standard Commands:
```baan
ADD.SET         |* Add new record
MODIFY.SET      |* Modify existing record  
DELETE.SET      |* Delete record
SAVE.SET        |* Save changes
CANCEL.SET      |* Cancel operation
REFRESH.SET     |* Refresh display
PRINT.SET       |* Print current view
```

### Form Commands Overview
**Form Commands** are custom UI commands that you define manually in the session configuration (not in code). They extend the standard functionality with business-specific operations.

#### Key Characteristics:
- ❌ **Not pre-defined**: Must be created manually in session config
- 🔧 **Two types**: Start Session or Call Function
- 📝 **Function requirement**: If "Call a Function" type, must define `extern` function in 4GL script
- 🎯 **Business specific**: Custom operations for specific requirements

#### Form Command Types:
1. **Start a Session**: Triggers a new session when clicked
2. **Call a Function**: Triggers a function in your 4GL script

### Runtime Command Control

#### Enabling Commands
```baan
function void enable.commands(const string command, const string ...)
```

**Usage Examples:**
```baan
|* Enable single command
enable.commands(ADD.SET)

|* Enable multiple commands
enable.commands("command.post.transaction", "ttadv3500m000", ADD.SET)

|* Enable form command by function name
enable.commands("command.approve.invoice")
```

#### Disabling Commands
```baan
function void disable.commands(const string command, const string ...)
```

**Usage Examples:**
```baan
|* Disable single command
disable.commands(DELETE.SET)

|* Disable multiple commands
disable.commands("command.post.transaction", MODIFY.SET, DELETE.SET)

|* Disable based on business logic
if customer.type = "RESTRICTED" then
    disable.commands(ADD.SET, MODIFY.SET)
endif
```

### Form Command Implementation

#### Step 1: Define Form Command Function
```baan
|* Function name must match form command name exactly
function extern command.post.transaction()
{
    message("Processing transaction...")
    
    |* Business logic here
    if validate.transaction.data() then
        process.transaction()
        message("Transaction posted successfully")
    else
        message("Transaction validation failed")
    endif
}

function extern command.approve.invoice()
{
    |* Check user permissions
    if not check.approval.authority() then
        message("Insufficient authorization for approval")
        return
    endif
    
    |* Update invoice status
    tccom200.status = "APPROVED" 
    tccom200.approved.by = logname$
    tccom200.approved.date = date.num()
    
    display("tccom200.status")
    display("tccom200.approved.by")
    display("tccom200.approved.date")
    
    message("Invoice approved successfully")
}
```

#### Step 2: Conditional Command Control
```baan
main.table.io:
after.read:
    |* Enable/disable commands based on record status
    on case tccom200.status
    case "DRAFT":
        enable.commands("command.post.transaction", MODIFY.SET, DELETE.SET)
        disable.commands("command.approve.invoice")
    case "POSTED":
        enable.commands("command.approve.invoice", MODIFY.SET)
        disable.commands("command.post.transaction", DELETE.SET)
    case "APPROVED":
        disable.commands("command.post.transaction", "command.approve.invoice", MODIFY.SET, DELETE.SET)
    default:
        disable.commands("command.post.transaction", "command.approve.invoice")
    endcase

field.tccom200.status:
when.field.changes:
    |* Update command availability when status changes
    on case tccom200.status
    case "DRAFT":
        enable.commands("command.post.transaction")
        disable.commands("command.approve.invoice")
    case "POSTED": 
        enable.commands("command.approve.invoice")
        disable.commands("command.post.transaction")
    case "APPROVED":
        disable.commands("command.post.transaction", "command.approve.invoice")
    endcase
```

### DAL-Side Command Authorization

#### Standard Command Control in DAL
```baan
|* In DAL script
function extern long method.is.allowed(long method)
{
    |* Control standard commands (ADD, MODIFY, DELETE)
    on case method
    case DAL_NEW:
        |* Check if user can add records
        if not check.user.permission("CREATE_CUSTOMERS") then
            return(0)  |* Disable ADD command
        endif
    case DAL_UPDATE:
        |* Check if user can modify records
        if not check.user.permission("MODIFY_CUSTOMERS") then
            return(0)  |* Disable MODIFY command
        endif
    case DAL_DELETE:
        |* Check if user can delete records
        if not check.user.permission("DELETE_CUSTOMERS") then
            return(0)  |* Disable DELETE command
        endif
    endcase
    
    return(1)  |* Allow operation
}
```

#### Business Method Control in DAL
```baan
|* In DAL script  
function extern long business.method.is.allowed()
{
    |* Control form commands/business methods
    
    |* Check if post transaction method is allowed
    if business.method = "command.post.transaction" then
        if tccom200.status <> "DRAFT" then
            return(0)  |* Disable if not draft
        endif
        
        if not check.user.permission("POST_TRANSACTIONS") then
            return(0)  |* Disable if no permission
        endif
    endif
    
    |* Check if approve invoice method is allowed
    if business.method = "command.approve.invoice" then
        if tccom200.status <> "POSTED" then
            return(0)  |* Disable if not posted
        endif
        
        if not check.user.permission("APPROVE_INVOICES") then
            return(0)  |* Disable if no permission
        endif
    endif
    
    return(1)  |* Allow operation
}
```

### Command Control Patterns

#### Security-Based Control
```baan
main.table.io:
after.read:
    |* Role-based command control
    string user.role(20)
    
    user.role = get.user.role(logname$)
    
    on case user.role
    case "CLERK":
        enable.commands(ADD.SET, MODIFY.SET)
        disable.commands(DELETE.SET, "command.approve.invoice")
    case "SUPERVISOR":
        enable.commands(ADD.SET, MODIFY.SET, DELETE.SET, "command.approve.invoice")
        disable.commands("command.post.transaction")
    case "MANAGER":
        enable.commands(ADD.SET, MODIFY.SET, DELETE.SET, "command.approve.invoice", "command.post.transaction")
    default:
        disable.commands(ADD.SET, MODIFY.SET, DELETE.SET, "command.approve.invoice", "command.post.transaction")
    endcase
```

#### Data-Driven Control
```baan
field.tccom100.cust.type:
when.field.changes:
    |* Enable different commands based on customer type
    on case tccom100.cust.type
    case "RETAIL":
        enable.commands("command.process.retail.order")
        disable.commands("command.process.wholesale.order", "command.credit.check")
    case "WHOLESALE":
        enable.commands("command.process.wholesale.order", "command.credit.check")
        disable.commands("command.process.retail.order")
    case "CORPORATE":
        enable.commands("command.process.wholesale.order", "command.credit.check", "command.corporate.pricing")
        disable.commands("command.process.retail.order")
    default:
        disable.commands("command.process.retail.order", "command.process.wholesale.order", "command.credit.check")
    endcase
```

### Command Integration with Business Logic

#### Workflow-Based Commands
```baan
function extern command.submit.for.approval()
{
    |* Validate before submission
    if not validate.document.completeness() then
        message("Document incomplete - cannot submit for approval")
        return
    endif
    
    |* Update workflow status
    tccom300.workflow.status = "PENDING_APPROVAL"
    tccom300.submitted.by = logname$
    tccom300.submitted.date = date.num()
    
    |* Send notification
    send.approval.notification(tccom300.approver.id)
    
    |* Update command availability
    disable.commands(MODIFY.SET, DELETE.SET, "command.submit.for.approval")
    enable.commands("command.cancel.submission")
    
    |* Refresh display
    display("tccom300.workflow.status")
    display("tccom300.submitted.by")
    display("tccom300.submitted.date")
    
    message("Document submitted for approval")
}

function extern command.cancel.submission()
{
    |* Confirm cancellation
    if ask.user.confirmation("Cancel submission?") = tcyesno.yes then
        tccom300.workflow.status = "DRAFT"
        tccom300.submitted.by = ""
        tccom300.submitted.date = 0
        
        |* Update command availability
        enable.commands(MODIFY.SET, DELETE.SET, "command.submit.for.approval")
        disable.commands("command.cancel.submission")
        
        |* Refresh display
        display("tccom300.workflow.status")
        message("Submission cancelled")
    endif
}
```

### Command Control Summary

| Control Level | Method | Use Case |
|--------------|--------|----------|
| **UI Script** | `enable.commands()`/`disable.commands()` | Dynamic runtime control |
| **DAL** | `method.is.allowed()` | Standard command authorization |
| **DAL** | `business.method.is.allowed()` | Form command authorization |
| **Session Config** | Form Command Definition | Custom command creation |

### Command Best Practices

#### Design Guidelines:
1. **Consistent naming**: Use clear, descriptive command names
2. **Security first**: Always implement proper authorization checks
3. **User feedback**: Provide clear messages for command actions
4. **State management**: Update UI state after command execution
5. **Error handling**: Handle command failures gracefully

#### Performance Considerations:
1. **Minimal validation**: Keep command validation lightweight
2. **Efficient updates**: Update only necessary UI elements
3. **Batch operations**: Group related command actions
4. **Resource cleanup**: Clean up resources after command execution

#### Restrictions:
- ⚠️ **Cannot use in `before.program` section**
- ⚠️ **Cannot override user authorization or UI-level disablement**
- ✅ **Can be called from field events and lifecycle events**
- ✅ **Function names must match form command names exactly** 