# Baan DAL (Data Access Layer) Architecture

## �� **DAL Overview**

**DAL (Data Access Layer)** is an architecture in Infor ERP Enterprise used to **centralize and standardize all data integrity logic**. It replaces traditional direct database operations (`db.insert()`, `db.update()`, `db.delete()`) with **rule-driven hooks** that are executed automatically by the 4GL engine.

### **Critical Engine Behavior**
🚨 **If a DAL script exists for a table, the engine IGNORES related 4GL event sections in UI scripts and instead executes the DAL hooks. If no DAL is present, the 4GL logic executes as usual.**

## 🔶 **Why DAL?**

The goal of DAL is to:
- **Centralize logic** for validation and field control
- **Reuse logic** across:
  - UI sessions
  - Integrations (e.g., Baan OpenWorld)
  - Batch jobs
- **Ensure conformity** - all inserts, updates, and deletes conform to business rules
- **Enable consistent enforcement** of security and data integrity

## 📋 **What DAL Replaces**

| 4GL Function or Event | DAL Replacement Method |
|----------------------|------------------------|
| `field.<x>.check.input` | `function extern long <x>.check()` |
| `before.read` | `function extern long before.get.object()` |
| `after.read` | `function extern long after.get.object()` |
| `before.write` | `function extern long before.save.object()` |
| `after.write` | `function extern long after.save.object()` |
| `before.delete` | `function extern long before.destroy.object()` |
| `after.delete` | `function extern long after.destroy.object()` |

## 🏗️ **DAL vs UI Scripts**

### **DAL Logic (Database-Driven)**
- DAL runs when **database operations occur**
- Triggered by UI, integrations, or batch operations
- **Centralized business rules** and validation
- **Cannot be bypassed** by any application

### **UI Scripts (User-Driven)** 
- Scripts respond to **user input**, navigation, or screen logic
- **Must delegate all data operations to DAL**
- Handle user interface and form behavior only
- **No business logic** when DAL exists

## 🔧 **DAL Implementation Requirements**

### **Mandatory Include File**
Every DAL script **must** include:
```baan
#include <bic_dal>
```

### **Function Signature Pattern**
All DAL hooks use `function extern` with specific signatures:
```baan
function extern long hook.name()
{
    |* DAL logic here
    return(0)  |* 0 = success, DALHOOKERROR = error
}
```

## 🔄 **Complete DAL Object Hook Lifecycle**

DAL object hooks define validation and processing logic at the object (record) level. These hooks are **automatically called by the 4GL engine** during lifecycle events.

### **DAL Hook Lifecycle Table**

| Hook Name | When It's Called | Purpose |
|-----------|------------------|---------|
| `before.open.object.set()` | When opening the object set | Dependency setup |
| `before.new.object()` | Before a new record is initialized | Check if creation allowed |
| `after.new.object()` | After a new record is initialized | Initialize defaults |
| `set.object.defaults()` | Used to set default values | Default value logic |
| `before.change.object()` | Before any field change | Pre-change validation |
| `after.change.object()` | After any field change | Post-change processing |
| `before.get.object()` | Before retrieving from database | Access control |
| `after.get.object()` | After retrieving from database | Data preparation |
| `before.save.object(mode)` | Before insert/update | Pre-save validation |
| `after.save.object(mode)` | After insert/update | Post-save processing |
| `before.destroy.object()` | Before deleting a record | Delete validation |
| `after.destroy.object()` | After deleting a record | Cleanup operations |
| `method.is.allowed()` | To check if operation allowed | Dynamic permissions |
| `after.commit.transaction()` | After successful commit | Transaction finalization |
| `after.abort.transaction()` | After transaction abort | Rollback cleanup |

## 📝 **DAL Hook Implementation Details**

### **1. before.get.object()**
Called just before the object is fetched from the database. Prevent retrieval based on constraints.

```baan
function extern long before.get.object()
{
    if ttccom100.status = "ARCHIVED" then
        dal.set.error.message("Record is archived and cannot be retrieved.")
        return(DALHOOKERROR)
    endif
    return(0)
}
```

### **2. after.get.object()**
Executed after the object has been fetched. Used to prepare or format additional data.

```baan
function extern long after.get.object()
{
    |* Calculate derived fields after reading
    tccom100.available.credit = tccom100.crdt - tccom100.used.credit
    return(0)
}
```

### **3. before.save.object(long mode)**
Called before insert or update. Perform validations here.

```baan
function extern long before.save.object(long mode)
{
    if mode = DAL_NEW then
        |* Validation for new records
        if isspace(tccom100.cust) then
            dal.set.error.message("Customer code is required")
            return(DALHOOKERROR)
        endif
    endif
    
    if mode = DAL_UPDATE then
        |* Validation for updates
        if tccom100.crdt < 0 then
            dal.set.error.message("Credit limit cannot be negative")
            return(DALHOOKERROR)
        endif
    endif
    
    return(0)
}
```

### **4. after.save.object(long mode)**
Used to perform follow-up actions after save.

```baan
function extern long after.save.object(long mode)
{
    |* Update parent totals after saving order line
    long ret
    domain tcamnt old.amnt
    
    old.amnt = 0
    
    if mode = DAL_NEW then
        old.amnt = 0
    endif
    
    |* Update order header total
    select tdsls400.*
    from   tdsls400 for update
    where  tdsls400.orno = tccom100.orno
    selectdo
        dal.set.property("tdsls400", ttdsls400, "tdsls400.amnt",
            tdsls400.amnt - old.amnt + tccom100.amnt, DAL_UPDATE)
        dal.update("tdsls400", ttdsls400, ret, true, db.retry)
    selectempty
    endselect
    
    return(ret)
}
```

### **5. before.new.object()**
Check whether a new object is allowed to be created.

```baan
function extern long before.new.object()
{
    |* Check user permissions for creating customers
    if not check.user.permission("CUSTOMER_CREATE") then
        dal.set.error.message("Access denied: Cannot create customers")
        return(DALHOOKERROR)
    endif
    
    return(0)
}
```

### **6. after.new.object()**
Initialize default field values for new records.

```baan
function extern long after.new.object()
{
    |* Set default values for new customer
    tccom100.stat = "ACTIVE"
    tccom100.created.date = date.num()
    tccom100.created.by = logname$
    tccom100.payment.terms = 30
    
    return(0)
}
```

### **7. before.destroy.object()**
Validate whether a delete is allowed.

```baan
function extern long before.destroy.object()
{
    |* Check for dependent records
    select count(*)
    from tcord100
    where tcord100.cust = tccom100.cust
    selectdo
        if count(*) > 0 then
            dal.set.error.message("Cannot delete customer with existing orders")
            return(DALHOOKERROR)
        endif
    selectempty
    endselect
    
    return(0)
}
```

### **8. after.destroy.object()**
Perform cleanup after deletion.

```baan
function extern long after.destroy.object()
{
    |* Log deletion for audit trail
    ttaudit100.table.name = "tccom100"
    ttaudit100.record.key = tccom100.cust
    ttaudit100.action = "DELETE"
    ttaudit100.timestamp = date.num()
    ttaudit100.user = logname$
    
    dal.insert("ttaudit100", ttaudit100)
    
    return(0)
}
```

### **9. before.open.object.set()**
Key place to declare field dependencies and setup.

```baan
function extern long before.open.object.set()
{
    |* Declare field dependencies
    dal.field.depends.on("tccom100.available.credit", "tccom100.crdt")
    dal.field.depends.on("tccom100.available.credit", "tccom100.used.credit")
    
    |* Disable never applicable checks if needed
    dal.skip.never.applicable.checks()
    
    return(0)
}
```

### **10. method.is.allowed(string method)**
Dynamically enable/disable operations.

```baan
function extern long method.is.allowed(string method)
{
    if method = "Delete" then
        if tccom100.stat = "BLOCKED" then
            return(false)  |* Cannot delete blocked customers
        endif
    endif
    
    if method = "Update" then
        if tccom100.stat = "ARCHIVED" then
            return(false)  |* Cannot update archived customers
        endif
    endif
    
    return(true)
}
```

### **11. set.object.defaults()**
Initialize default values from parameters or static data.

```baan
function extern long set.object.defaults()
{
    |* Set defaults from system parameters
    select ttccom000.default.payment.terms
    from ttccom000
    selectdo
        tccom100.payment.terms = ttccom000.default.payment.terms
    selectempty
        tccom100.payment.terms = 30  |* Fallback default
    endselect
    
    return(0)
}
```

### **12. Transaction Hooks**
```baan
function extern long after.commit.transaction()
{
    |* Send notifications after successful save
    if tccom100.crdt > 100000 then
        send.high.credit.notification(tccom100.cust)
    endif
    
    return(0)
}

function extern long after.abort.transaction()
{
    |* Cleanup after failed transaction
    cleanup.temp.data()
    
    return(0)
}
```

## 🎯 **DAL Field-Level Hooks**

DAL also provides field-level validation hooks that replace individual field validation:

```baan
|* Replace field.tccom100.cust.check.input
function extern long cust.check()
{
    if isspace(tccom100.cust) then
        dal.set.error.message("Customer code is required")
        return(DALHOOKERROR)
    endif
    
    if len(tccom100.cust) < 3 then
        dal.set.error.message("Customer code must be at least 3 characters")
        return(DALHOOKERROR)
    endif
    
    return(0)
}

|* Field applicability control
function extern long crdt.is.applicable()
{
    |* Credit limit only applicable for active customers
    if tccom100.stat = "ACTIVE" then
        return(true)
    endif
    
    return(false)
}

|* Field readonly control
function extern long name.is.readonly()
{
    |* Name readonly for archived customers
    if tccom100.stat = "ARCHIVED" then
        return(true)
    endif
    
    return(false)
}
```

## 🚨 **Critical DAL Rules**

### **1. Engine Behavior**
- **If DAL exists**: Engine ignores UI script event sections and uses DAL hooks
- **If NO DAL**: UI script 4GL logic executes as normal
- **Cannot bypass**: DAL logic executes for ALL database operations

### **2. Return Values**
- **`return(0)`**: Success, continue operation
- **`return(DALHOOKERROR)`**: Error, abort operation
- **Error messages**: Use `dal.set.error.message("text")`

### **3. Implementation Requirements**
- **Must use `function extern`** for all DAL hooks
- **Must include `#include <bic_dal>`** at top of DAL script
- **Compiled automatically** by 4GL engine
- **Cannot be disabled** or bypassed

## 📊 **Complete Architecture Flow**

```
UI Action (Save) → DAL Hook Chain → Database
     │                    │             │
     │         before.save.object()     │
     │         field.check()            │
     │         method.is.allowed()      │
     │                    │             │
     │         → Database Operation     │
     │                    │             │
     │         after.save.object()      │
     │         after.commit.transaction() │
     │                    │             │
     ← Result Message ← DAL Result ← Success/Error
```

## 🎯 **Summary: Complete DAL Understanding**

### **DAL Is:**
- **Hook-based integrity layer** bound to database tables
- **Automatically triggered** by 4GL engine during database operations
- **Single source of truth** for all business logic and validation
- **Cannot be bypassed** by any application (UI, integration, batch)

### **DAL Replaces:**
- All `field.*.check.input` events in UI scripts
- All `before.read`, `after.read`, `before.write`, `after.write` events
- All direct `db.insert()`, `db.update()`, `db.delete()` operations
- All table-level business logic and validation

### **DAL Ensures:**
- **Centralized business rules** used by all applications
- **Data integrity** maintained automatically
- **Consistent behavior** across UI, integrations, and batch processing
- **Security enforcement** at the data layer

This DAL architecture is the **foundation of data integrity** in Baan ERP, ensuring that business logic cannot be circumvented and data remains consistent across all access methods! 🚀

---

## 💡 **DAL Overview (Original)**

In Infor ERP Enterprise, there is a **clear separation of responsibilities**:
- **UI Logic**: Written in Baan 4GL (handles user interface, form behavior)
- **Data Logic**: Handled by DAL (Data Access Layer) scripts (handles business rules, validation, database operations)

## 🔶 **What is DAL? (Original)**

**DAL (Data Access Layer)** is a **hook-based integrity layer** that provides a central place for defining business logic and integrity checks on database tables.

### **Key Characteristics**
- **NOT written in 4GL**: Uses different syntax and structure than UI scripts
- **Hook-based system**: Automatically triggered by the 4GL engine during database operations
- **Table-bound**: Each DAL is bound to specific database tables (object sets)
- **Compiled scripts**: DAL scripts are compiled and triggered automatically
- **Engine-managed**: The 4GL engine calls DAL hooks at appropriate times

### **DAL Purpose**
```
┌─────────────────────────────────────────────────────────────┐
│                    DAL CENTRAL PURPOSE                      │
│                                                             │
│  Replace direct database operations with reusable,         │
│  structured business logic that ensures data integrity     │
│  and consistency across ALL applications                   │
└─────────────────────────────────────────────────────────────┘
```

## 🏗️ **DAL Architecture (Original)**

### **How DAL Works**
```
UI Script (4GL)           DAL Layer               Database
     │                       │                       │
     │ dal.save.customer()   │                       │
     ├──────────────────────▶│ before.save.object()  │
     │                       │ field.is.mandatory()  │
     │                       │ field.is.valid()      │
     │                       │ field.set.defaults()  │
     │                       ├──────────────────────▶│ db.insert()
     │                       │ after.save.object()   │ db.update()
     │ ◀──────────────────────┤                       │ db.delete()
     │ Success/Error Result   │                       │
```

### **DAL Replaces Direct Database Operations**
```baan
|* ❌ OLD WAY - Direct database operations in UI
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

|* ✅ NEW WAY - DAL handles everything
choice.save:
    dal.save.customer(tccom100)  |* DAL handles validation, business logic, DB operations
```

## 🛠️ **DAL Responsibilities (Original)**

### ✅ **What DAL Handles**

#### **1. Field-Level Logic**
- **`field.is.mandatory()`** - Required field checks
- **`field.is.applicable()`** - Field applicability rules
- **`field.is.readonly()`** - Field access control
- **`field.update()`** - Field value updates and calculations
- **`field.set.defaults()`** - Default value logic
- **`field.check()`** - Field validation rules
- **`field.is.valid()`** - Consistency validation

#### **2. Object Lifecycle Hooks**
- **`before.save.object()`** - Pre-save validation and preparation
- **`after.save.object()`** - Post-save processing and updates
- **`before.new.object()`** - New record initialization
- **`after.new.object()`** - Post-creation processing
- **`before.delete.object()`** - Pre-delete validation
- **`after.delete.object()`** - Post-delete cleanup

#### **3. Database Operations**
- **Insert operations**: Handles `db.insert()` with validation
- **Update operations**: Handles `db.update()` with business rules
- **Delete operations**: Handles `db.delete()` with integrity checks
- **Transaction management**: Ensures proper commit/rollback

#### **4. Business Logic Enforcement**
- **Cross-field validation**: Complex business rules
- **Data consistency**: Ensures referential integrity
- **Audit trail**: Automatic logging of changes
- **Security**: Enforces access controls

### ⛔ **What DAL Replaces in UI Scripts**

#### **Direct Database Operations**
```baan
|* ❌ DON'T USE in UI (when DAL exists)
db.insert(), db.update(), db.delete()
```

#### **Field-Level Validations**
```baan
|* ❌ DON'T USE in UI (when DAL exists)
field.tccom100.cust:
check.input:
    if isspace(tccom100.cust) then
        set.input.error("Customer required")
    endif
```

#### **Defaulting Logic**
```baan
|* ❌ DON'T USE in UI (when DAL exists)  
field.tccom100.type:
when.field.changes:
    if tccom100.type = "CORPORATE" then
        tccom100.payment.terms = 30
    endif
```

## 🔐 **Critical DAL Rule (Original)**

> **"If a table has a DAL, then ALL direct database operations must go through that DAL. Do not call `db.write()`, `db.update()`, or `db.delete()` directly in the UI."**

### **DAL is the Single Source of Truth**
- **All validation logic** resides in DAL
- **All business rules** are enforced by DAL
- **All database operations** are controlled by DAL
- **All applications** (UI, BOI, batch jobs) use the same DAL logic

## 🧩 **DAL Benefits (Original)**

### **1. 🔁 Centralized Logic**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   UI Scripts    │    │  Batch Jobs     │    │  Integration    │
│     (4GL)       │    │     (4GL)       │    │     (BOI)       │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 │
                    ┌─────────────▼─────────────┐
                    │        DAL LAYER          │
                    │  All Business Logic       │
                    │  Single Source of Truth   │
                    └───────────────────────────┘
```

### **2. 🧠 Reusable Logic**
- **UI applications** use DAL for user interactions
- **BOI (Business Object Interface)** uses DAL for integrations
- **Batch jobs** use DAL for mass processing
- **Reports** use DAL for data access

### **3. ✅ Automatic Validation Hooks**
- Engine **automatically calls** field-level hooks during data entry
- Engine **automatically calls** object-level hooks during save/delete
- **No way to bypass** validation in any application
- **Consistent behavior** across all access methods

### **4. 🔐 Security & Integrity**
- **Prevents bypassing** validations in integrations
- **Enforces security** rules at data layer
- **Maintains referential integrity** automatically
- **Provides audit trail** for all changes

### **5. 🧩 DAL2 Support**
- **Dependency-based field logic** for complex relationships
- **Richer UI interaction** with automatic field updates
- **Advanced validation patterns** for modern business requirements

## 📋 **UI Script + DAL Integration Patterns (Original)**

### **Standard UI Script with DAL**
```baan
|* UI Script for Customer Maintenance (tccom0101m000)
table ttccom100 tccom100

|* ===== UI-LEVEL VALIDATION ONLY =====
field.tccom100.cust:
when.field.changes:
    |* Simple UI formatting only
    tccom100.cust = upper(tccom100.cust)
    display("tccom100.cust")
    
    |* Load data via DAL
    dal.read.customer.details(tccom100.cust)

|* ===== USER CHOICES - ALWAYS USE DAL =====
choice.save:
    |* UI validation for user experience
    if isspace(tccom100.cust) then
        message("Please enter customer code")
        return
    endif
    
    |* All business logic and validation in DAL
    dal.save.customer(tccom100)
    
    |* Handle DAL result
    if dal.get.error.message() <> "" then
        message("Error: " & dal.get.error.message())
    else
        message("Customer saved successfully")
    endif

choice.delete:
    |* Confirmation only in UI
    if ask.enum("Delete customer?", tcokca.yes, tcokca.no) = tcokca.yes then
        |* All deletion logic in DAL
        dal.delete.customer(tccom100.cust)
        
        |* Handle result
        if dal.get.error.message() <> "" then
            message("Cannot delete: " & dal.get.error.message())
        else
            message("Customer deleted")
        endif
    endif
```

### **DAL Hook Examples (Conceptual)**
```
|* DAL for tccom100 (Customer table)

field.tccom100.cust.is.mandatory():
    return(true)  |* Customer code always required

field.tccom100.crdt.is.valid():
    if tccom100.crdt < 0 then
        dal.set.error.message("Credit limit cannot be negative")
        return(false)
    endif
    return(true)

field.tccom100.name.update():
    |* Auto-format customer name
    tccom100.name = proper(tccom100.name)

before.save.object():
    |* Complex business validation
    if tccom100.type = "CORPORATE" and isspace(tccom100.tax.id) then
        dal.set.error.message("Corporate customers require tax ID")
        return(false)
    endif
    return(true)

after.save.object():
    |* Update related records, send notifications
    update.customer.credit.history()
    notify.account.manager()
```

## 🎯 **Summary: UI Scripts vs DAL (Original)**

### **Responsibility Matrix**

| Task | UI Script (4GL) | DAL | Reason |
|------|----------------|-----|--------|
| **Field formatting** | ✅ | ❌ | User experience |
| **Required field validation** | ❌ | ✅ | Business rule |
| **Cross-field validation** | ❌ | ✅ | Business logic |
| **Default value setting** | ❌ | ✅ | Data integrity |
| **Database insert/update** | ❌ | ✅ | Centralized control |
| **User confirmation** | ✅ | ❌ | User interaction |
| **Form navigation** | ✅ | ❌ | UI behavior |
| **Error message display** | ✅ | ❌ | User interface |
| **Business rule enforcement** | ❌ | ✅ | Data consistency |
| **Audit trail creation** | ❌ | ✅ | Automatic logging |

### **Development Approach**
```
1. Is this a user interface concern?
   ↓ YES                           ↓ NO
2. Handle in UI Script          2. Handle in DAL
   - Form behavior                 - Business rules
   - User interaction              - Data validation
   - Display formatting            - Database operations
   - Navigation                    - Audit logging
```

## 🚨 **Critical Understanding (Original)**

DAL is **not just another layer** - it's the **foundation** of data integrity in Baan ERP. Every database operation, whether from UI, integration, or batch processing, **must** go through DAL to ensure:

- **Consistent validation** across all access methods
- **Centralized business rules** that cannot be bypassed
- **Data integrity** maintained automatically
- **Audit trails** captured for all changes

This architecture ensures that **business logic lives in one place** and **cannot be circumvented**, making the system reliable and maintainable! 🚀

---

## Complete DAL Hook System

DAL provides a comprehensive two-tier hook system for complete data integrity management:

### **Object-Level Hooks**
**See**: [DAL Object Hooks](Baan_DAL_Object_Hooks.md) for complete object lifecycle management including:
- `before.new.object()` / `after.new.object()` - Record creation lifecycle
- `before.save.object()` / `after.save.object()` - Save/update validation and processing
- `before.destroy.object()` / `after.destroy.object()` - Deletion validation and cleanup
- `before.change.object()` / `after.change.object()` - Edit mode management
- `before.get.object()` / `after.get.object()` - Read operations and post-processing
- Transaction hooks: `after.commit.transaction()` / `after.abort.transaction()`
- Permission hooks: `method.is.allowed()`
- Object set management: `before.open.object.set()`

### **Field-Level Hooks**

**Classic DAL Property Hooks**: [DAL Property Hooks](Baan_DAL_Property_Hooks.md) for basic field-level validation:
- `field.set.defaults()` - Default value assignment
- `field.make.valid()` - Data normalization  
- `field.check()` - Field validation
- `dal.set.property()` / `dal.set.field()` usage

**Advanced DAL2 Field Hooks**: [DAL2 Field Hooks](Baan_DAL2_Field_Hooks.md) for sophisticated field behavior control:
- `field.is.applicable()` / `field.is.never.applicable()` - Field visibility/enablement
- `field.is.readonly()` / `field.is.derived()` / `field.is.mandatory()` - Field state control
- `field.is.valid()` / `field.enum.is.applicable()` - Advanced validation
- `field.update()` - Automatic field value calculation
- `dal.field.depends.on()` - Declarative field dependency system
- `dal.require.field()` / `dal.any.parent.changed()` - Dependency management

**DAL2 Engine Integration**: [DAL2 Validation Flows](Baan_DAL2_Validation_Flows.md) for engine execution mechanics:
- Engine hook calling sequence and UI event mapping
- Internal validation flows: `dal.validate.field()` / `dal.update.field()` / `dal.get.field.state()`
- Business method hooks: `business.method.is.allowed()` / `business.method.is.never.allowed()`
- Complete integration patterns and security controls

**Complete Session Integration**: [DAL Session Integration](Baan_DAL_Session_Integration.md) for UI-DAL integration and complete reference:
- How 4GL UI sessions interact with DAL operations
- Complete UI-DAL flow patterns with error handling
- Advanced integration patterns: multi-table transactions, conditional operations
- Complete DAL/DAL2 hooks reference guide and cheat sheet
- Best practices for session-level DAL integration

---

## Related Documentation

### **DAL System Documentation**
- [DAL Object Hooks](Baan_DAL_Object_Hooks.md) - Complete object lifecycle hook management
- [DAL Property Hooks](Baan_DAL_Property_Hooks.md) - Classic DAL field-level hooks and validation
- [DAL2 Field Hooks](Baan_DAL2_Field_Hooks.md) - Advanced DAL2 field behavior and dependencies
- [DAL2 Validation Flows](Baan_DAL2_Validation_Flows.md) - Engine integration and business method control
- [DAL Session Integration](Baan_DAL_Session_Integration.md) - **Complete UI-DAL integration and hooks reference**

### **Core Development Resources**
- [4GL Quick Reference](4GL_Quick_Reference.md) - Core language syntax and limitations
- [Review Standards](review-standards.md) - Development best practices and standards
- [Baan 4GL Advanced Patterns](Baan_4GL_Advanced_Patterns.md) - Complex business logic patterns 