# Baan ERP Architecture & Session Management

## 🚨 **CRITICAL ARCHITECTURAL RULE**

### **Default Development Approach: DAL-First**

When working in Baan 4GL sessions or UI scripts, your **default behavior** should **ALWAYS** be:

🔹 **Use DAL functions** (`dal.insert`, `dal.update`, `dal.delete`, etc.) to perform **ALL** database operations like reading, inserting, updating, or deleting records.

```baan
|* ✅ CORRECT - Default approach
choice.save:
    dal.save.customer(tccom100)           |* Use DAL
    
choice.delete:
    dal.delete.customer(tccom100.cust)    |* Use DAL
    
field.tccom100.cust:
when.field.changes:
    dal.read.customer.details(tccom100.cust)  |* Use DAL
```

### **Why DAL-First Approach?**

Using DAL ensures:
- ✅ **Consistent validation** across all applications
- ✅ **Centralized business rules** in one location
- ✅ **Better maintainability** and code reuse
- ✅ **Proper use of DAL events** (before.insert, on.update, after.delete, etc.)
- ✅ **Unified error handling** and logging
- ✅ **Transaction consistency** across operations

## ⛔ **Exception: When to Use Direct `db.*` Operations**

You can use direct `db.*` methods (e.g., `db.insert()`, `db.update()`, `db.read()`, `db.delete()`) **ONLY when:**

### **1. No DAL Defined for Table**
```baan
|* ✅ ACCEPTABLE - No DAL exists for tccom999 utility table
db.retry.point()

select tccom999.temp.data
from tccom999
where tccom999.session.id = session$
selectdo
    process.temp.data()
selectempty
endselect

commit.transaction()
```

### **2. Utility or Tool-Type Sessions**
```baan
|* ✅ ACCEPTABLE - System utility session
|* Reading system configuration where DAL is not practical
select ttsys001.parameter.value
from ttsys001  
where ttsys001.parameter.name = "SYSTEM_MODE"
selectdo
    system.mode = ttsys001.parameter.value
selectempty
    system.mode = "PRODUCTION"  |* Default
endselect
```

### **3. Internal Operations Where DAL Logic Not Relevant**
```baan
|* ✅ ACCEPTABLE - Internal logging where business rules don't apply
db.retry.point()

ttlog100.timestamp = date.num()
ttlog100.user = logname$
ttlog100.action = "USER_LOGIN"
ttlog100.session = session$

db.insert(ttlog100)

if db.retry then
    return
endif

commit.transaction()
```

### **4. Explicitly Instructed by Business Rules**
```baan
|* ✅ ACCEPTABLE - When business specifically requires direct access
|* Example: Emergency override procedures, data migration, etc.
```

### **Required Direct `db.*` Safety Pattern**
When using direct `db.*` operations, **ALWAYS** use retry-safe logic:

```baan
|* ✅ CORRECT - Complete retry-safe pattern
db.retry.point()

if db.retry.hit() then
    message("Retrying database operation...")
endif

select tccom100.cust, tccom100.name
from tccom100 for update  
where tccom100.cust = "CUST001"

selectdo
    tccom100.name = "Updated Name"
    db.update(tccom100, DB.RETRY)
    
    if db.retry then
        return  |* Exit and retry
    endif
    
    commit.transaction()
    message("Customer updated successfully")

selectempty
    abort.transaction()
    message("Customer not found")
endselect
```

## 📋 **Decision Matrix: DAL vs Direct DB**

| Scenario | Use DAL | Use Direct DB | Reason |
|----------|---------|---------------|---------|
| **Customer maintenance** | ✅ | ❌ | DAL exists with business rules |
| **Order processing** | ✅ | ❌ | Complex validation in DAL |
| **System logging** | ❌ | ✅ | Simple utility, no DAL needed |
| **Temp table cleanup** | ❌ | ✅ | Internal operation, no business logic |
| **Configuration read** | ❌ | ✅ | System parameter, no DAL |
| **Financial transactions** | ✅ | ❌ | Critical business rules in DAL |
| **Data migration** | Depends | Depends | Business decision |

## 🎯 **Summary Rule**

> **"Use DAL for all data operations unless there's a clear, necessary reason to use `db.*` directly — and that reason must be justified by the absence or inapplicability of a DAL layer for the target table."**

### **Development Workflow**
```
1. Need database operation?
   ↓
2. Does DAL exist for this table?
   ↓ YES                    ↓ NO
3. Use DAL method       3. Use direct db.* with retry-safe pattern
   ↓                       ↓
4. Handle DAL result    4. Handle db.* result with transactions
```

### **Quick Reference**
```baan
|* ✅ DEFAULT APPROACH - Use DAL
dal.save.customer(tccom100)
dal.read.order.details(order.number)
dal.delete.invoice(invoice.number)

|* ⛔ EXCEPTION ONLY - Direct DB (when no DAL)
db.retry.point()
select/insert/update/delete operations
commit.transaction() / abort.transaction()
```

---

## 🏗️ Baan ERP Architecture Overview

Baan ERP uses a **modular architecture** built around **sessions** and **scripts** that work together to implement business processes.

### Core Components
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│    SESSION      │───▶│   UI SCRIPT     │───▶│   DAL LAYER     │
│ Business Logic  │    │  (4GL Code)     │    │ Database Access │
│ Module          │    │  Form Control   │    │   (4GL Code)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
        │                       │                       │
        │                       │                       │
        ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   USER FORMS    │    │  FIELD EVENTS   │    │  TABLE BUFFERS  │
│ Screen Layout   │    │  Validation     │    │   tccom100      │
│ Input Fields    │    │  Calculations   │    │   tcord100      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 🧩 What is a Session?

A **session** is the fundamental modular unit of business logic in Baan ERP.

### Session Characteristics
- **Modular Design**: Each session represents one business activity
- **Self-Contained**: Complete business logic in one unit
- **User Interface**: Has attached form for user interaction
- **Reusable**: Can be called from other sessions or processes
- **Secure**: Built-in authorization and access control

### Session Types & Usage

| Type | Purpose | Example Session | When to Use |
|------|---------|----------------|-------------|
| **1** | **Master Data Maintenance** | `tccom0101m000` - Maintain Customers | Creating/updating reference data |
| **2** | **Transactional Processing** | `tdsls4101m000` - Sales Orders | Daily business transactions |
| **4** | **Background Jobs** | `tfacp2521m000` - Auto Settlement | Batch processing, scheduled tasks |
| **5** | **Reporting** | `tfacp4401r000` - A/R Aging Report | Data analysis and reporting |

### Session Naming Convention
```
tssss9999m000
│││││││││││││
│││││││││││└─ Sequence (000-999)
│││││││││└── Module type (m=maintain, r=report, etc.)
│││││└──────── Sequence number (0000-9999)
│└───────────── Functional area (com=common, sls=sales, etc.)
└────────────── Package identifier (t=table)
```

## 🎨 UI Scripts (4GL Programs)

Each session has an attached **UI Script** written in **Baan 4GL** that controls the user interface and form behavior.

### UI Script Responsibilities
```baan
|* UI Script Structure for Session tccom0101m000
table ttccom100 tccom100

|* 1. Form Initialization
after.form.read:
    init.references()
    create.sql.queries() 
    setup.field.properties()

|* 2. Field-Level Logic
field.tccom100.cust:
when.field.changes:
    validate.customer.code()
    update.dependent.fields()

|* 3. User Interaction Handling
choice.save:
    if validate.all.fields() then
        dal.save.customer(tccom100)  |* ✅ USE DAL BY DEFAULT
        message("Customer saved successfully")
    endif

|* 4. Navigation Control
zoom.from.tccom100.cust:
on.entry:
    setup.customer.zoom.filter()
```

### Form Lifecycle Events
```baan
|* Complete form lifecycle in UI script
before.form:
    |* Initialize form before display
    setup.default.values()
    
after.form.read:
    |* After form structure loaded
    init.references()
    create.sql.queries()
    setup.field.attributes()
    
before.display.object:
    |* Before showing form to user
    apply.security.rules()
    set.field.states()
    
after.form:
    |* Form cleanup
    cleanup.resources()
```

## 🧠 When to Use Baan 4GL (UI Scripts)

### ✅ **CORRECT Uses of 4GL in UI Scripts**

#### 1. **Field Validation & User Input**
```baan
field.tccom100.crdt:
check.input:
    if tccom100.crdt < 0 then
        set.input.error("Credit limit cannot be negative")
    endif
    
when.field.changes:
    tccom100.available.credit = tccom100.crdt - tccom100.used.credit
    display("tccom100.available.credit")
```

#### 2. **User Interface Control**
```baan
field.customer.type:
when.field.changes:
    on case customer.type
    case "CORPORATE":
        enable.fields("tax.number", "credit.rating")
        show.group(2)  |* Corporate information group
    case "INDIVIDUAL":
        disable.fields("tax.number", "credit.rating")
        hide.group(2)
    endcase
```

#### 3. **User Interaction & Choices**
```baan
choice.save:
    if not validate.required.fields() then
        return
    endif
    
    if ask.enum("Save customer changes?", tcokca.yes, tcokca.no) = tcokca.yes then
        dal.save.customer(tccom100)  |* ✅ DEFAULT: Use DAL
        message("Customer saved successfully")
    endif

choice.delete:
    if ask.enum("Delete customer " & tccom100.cust & "?", 
                tcokca.yes, tcokca.no) = tcokca.yes then
        dal.delete.customer(tccom100.cust)  |* ✅ DEFAULT: Use DAL
        message("Customer deleted")
    endif
```

#### 4. **Navigation & Zoom Logic**
```baan
zoom.from.tccom100.country:
on.entry:
    |* Filter zoom to show only active countries
    query.extend.where.in.zoom("tcmcs010.stat = 1")

zoom.to.customer.orders:
    |* Navigate to customer's order history
    export("zoom.customer", tccom100.cust)
    start.session("tdsls4101m000", "zoom.mode")
```

#### 5. **Calling DAL Methods (Default Approach)**
```baan
choice.save:
    |* ✅ DEFAULT: UI script calls DAL for database operations
    dal.start.business.method("tccom100", "save.customer", 
                             tccom100.cust, tccom100.name, tccom100.crdt)
    
    if dal.get.error.message() <> "" then
        message("Error: " & dal.get.error.message())
    else
        message("Customer saved successfully")
    endif
```

### ⛔ **INCORRECT Uses - Do NOT Use 4GL For**

#### ❌ **Direct Database Operations (When DAL Available)**
```baan
|* ❌ WRONG - Don't do direct SQL in UI scripts when DAL exists
choice.save:
    db.retry.point()
    select tccom100.cust from tccom100 for update
    where tccom100.cust = tccom100.cust
    selectdo
        tccom100.name = "Updated Name"
        db.update(tccom100, DB.RETRY)
        commit.transaction()
    selectempty
    endselect
```

#### ✅ **CORRECT - Use DAL Instead**
```baan
choice.save:
    |* ✅ CORRECT - Delegate database operations to DAL
    dal.save.customer(tccom100)
```

#### ❌ **Complex Business Logic (Belongs in DAL)**
```baan
|* ❌ WRONG - Complex validation should be in DAL
field.tccom100.crdt:
when.field.changes:
    |* Don't put complex business rules in UI
    select sum(tcord100.amnt) from tcord100 where...
    |* Calculate credit exposure...
    |* Check payment history...
    |* Apply credit scoring...
```

#### ❌ **Batch Processing (Use Background Sessions)**
```baan
|* ❌ WRONG - Don't do batch processing in UI scripts
choice.process.all:
    select tccom100.cust from tccom100
    selectdo
        |* Process thousands of customers in UI
        update.customer.data()
    selectempty
    endselect
```

## 📋 UI Script Patterns & Best Practices

### Standard UI Script Structure
```baan
|* Standard UI script template
|* Session: tccom0101m000 - Maintain Customers

table ttccom100 tccom100

|* ===== FORM LIFECYCLE EVENTS =====
before.form:
    setup.session.defaults()

after.form.read:
    init.references()
    create.sql.queries()
    apply.user.permissions()

|* ===== FIELD EVENTS =====
field.tccom100.cust:
when.field.changes:
    validate.customer.code()
    dal.read.customer.details(tccom100.cust)  |* ✅ Use DAL

field.tccom100.crdt:
check.input:
    validate.credit.limit()

|* ===== USER CHOICES =====
choice.save:
    if validate.customer.data() then
        dal.save.customer(tccom100)  |* ✅ Use DAL
        handle.save.result()
    endif

choice.delete:
    confirm.and.delete.customer()

|* ===== ZOOM NAVIGATION =====
zoom.from.tccom100.cust:
on.entry:
    setup.customer.zoom()

|* ===== UTILITY FUNCTIONS =====
functions:

function validate.customer.data() : long
{
    |* UI-level validation
    if isspace(tccom100.cust) then
        set.input.error("Customer code required")
        return(0)
    endif
    
    if isspace(tccom100.name) then
        set.input.error("Customer name required") 
        return(0)
    endif
    
    return(1)
}
```

### Common UI Script Patterns

#### 1. **Conditional Field States**
```baan
function set.field.states.by.status()
{
    on case tccom100.stat
    case "ACTIVE":
        enable.fields("tccom100.crdt", "tccom100.payment.terms")
        set.field.color("tccom100.stat", "green")
    case "BLOCKED":
        disable.fields("tccom100.crdt", "tccom100.payment.terms")
        set.field.color("tccom100.stat", "red")
    case "PROSPECT":
        enable.fields("tccom100.crdt")
        disable.fields("tccom100.payment.terms")
        set.field.color("tccom100.stat", "yellow")
    endcase
}
```

#### 2. **Progressive Data Loading**
```baan
field.tccom100.cust:
when.field.changes:
    if not isspace(tccom100.cust) then
        |* ✅ Load related customer data via DAL
        dal.read.customer.details(tccom100.cust)
        
        |* Update dependent fields
        display("tccom100.name")
        display("tccom100.city")
        display("tccom100.crdt")
        
        |* Set field states based on loaded data
        set.field.states.by.status()
    endif
```

#### 3. **Multi-Step Validation**
```baan
function validate.customer.save() : long
{
    long validation.result
    
    validation.result = 1
    
    |* Step 1: Required field validation
    if not validate.required.fields() then
        validation.result = 0
    endif
    
    |* Step 2: Business rule validation
    if validation.result and not validate.business.rules() then
        validation.result = 0
    endif
    
    |* Step 3: External validation (via DAL)
    if validation.result then
        validation.result = dal.validate.customer(tccom100)
    endif
    
    return(validation.result)
}
```

## 🎯 Summary: 4GL in Baan ERP Context

### **Component Responsibilities**

| Component | Purpose | Language | Database Access |
|-----------|---------|----------|----------------|
| **Session** | Business logic unit | N/A | Via UI Script + DAL |
| **UI Script** | Form behavior, user interaction | **Baan 4GL** | **Via DAL calls (DEFAULT)** |
| **DAL** | Database operations, business validation | **Baan 4GL** | **Direct database access** |
| **Table Buffer** | Data structure | N/A | Used by both UI + DAL |

### **4GL Usage Guidelines**

#### ✅ **Use 4GL in UI Scripts For:**
- Field validation and user input handling
- Form navigation and zoom logic  
- User interface control (enable/disable fields)
- User interaction (choices, confirmations)
- **Calling DAL methods for database operations (DEFAULT)**
- Display formatting and field updates

#### ⛔ **Do NOT Use 4GL in UI Scripts For:**
- Direct database read/write operations (use DAL by default)
- Complex business logic validation (belongs in DAL)
- Batch processing (use background sessions)
- Heavy computational tasks (use dedicated sessions)

### **Architecture Flow**
```
User Input → UI Script (4GL) → DAL Method (4GL) → Database
    ↑              ↓              ↓              ↓
User Display ← Field Events ← Business Logic ← Table Operations
```

This architecture ensures **separation of concerns** and **maintainable code structure** in Baan ERP development! 🚀 