# Session ↔ DAL Integration

This section explains **how 4GL UI sessions interact with DAL** in a DAL-based architecture. Since direct database operations (`db.write`, `db.delete`, etc.) are no longer used, all object manipulations must go through the DAL.

---

## 🔶 When the UI Calls DAL

DAL logic is automatically triggered by the **4GL engine** during user interactions in a session. This includes:

- Creating a new record
- Changing or deleting a record
- Saving a record
- Validating a field
- Pressing a form command linked to a business method

---

## 🔶 UI → DAL Flow Overview

| UI Section                   | What to Call                   | DAL Effect                            |
|-----------------------------|--------------------------------|----------------------------------------|
| `input.choice()`            | `dal.insert(tmain)`            | Triggers DAL insert logic              |
| `on.choice.save`            | `dal.update(tmain)`            | Triggers DAL update logic              |
|                             | `dal.delete(tmain)`            | Triggers DAL destroy logic             |
|                             | `commit.transaction()`         | Commits transaction if all DAL hooks pass |
|                             | `abort.transaction()`          | Aborts if DAL returns `DALHOOKERROR`   |
| `before.display.object:`    | — (engine calls `*.is.*()` hooks) | UI is dynamically enabled/disabled     |
| `field.<fld>:check.input:`  | —                              | Engine calls validation hooks          |
| `form.command:`             | Call DAL business method       | e.g., `post.transaction()`             |

### ➤ Complete UI-DAL Integration Flow

```baan
|* Example: Complete order entry session with DAL integration

choice.input:
    |* Create new order
    dal.insert("tdsls400")
    |* Engine automatically calls:
    |* 1. before.new.object()
    |* 2. set.object.defaults() 
    |* 3. after.new.object()

field.tdsls400.cuno:when.field.changes:
    |* Engine automatically calls DAL2 field hooks:
    |* 1. field.update() for dependent fields
    |* 2. field.is.applicable() for related fields
    |* 3. field.is.readonly() validation

on.choice.save:
    db.retry.point()
    ret = dal.update("tdsls400", main.table, error.occurred, true, db.retry)
    if ret = 0 then
        commit.transaction()
        |* Engine calls: after.commit.transaction()
    else
        |* Engine calls: after.abort.transaction()
        if db.retry then
            return  |* Allow user to correct and retry
        endif
    endif

form.command.post.order:
    |* Call business method
    ret = post.order()
    |* Business method internally checks:
    |* - post.order.is.never.allowed()
    |* - post.order.is.allowed()
```

---

## 🔶 Example: Insert a New Record via DAL

```baan
input.choice:
    on case input.selection
    case 1:  |* Add new customer
        dal.insert("tccom100")
        break
    case 2:  |* Add new item
        dal.insert("tcibd001") 
        break
    endcase
```

This will:

* Trigger `before.new.object()`
* Call `set.object.defaults()` to fill initial values
* Trigger `after.new.object()` and `before.save.object()`
* Validate all field hooks
* If all pass, insert into the database

### ➤ Complete Insert Flow with Error Handling

```baan
function insert.customer()
{
    |* Insert with proper error handling
    ret = dal.insert("tccom100", o.set, error.occurred, true, db.retry)
    
    if ret <> 0 then
        if db.retry then
            |* DAL validation failed - allow retry
            return
        else
            |* Fatal error - cannot proceed
            message("Failed to create customer: " & error.message$)
            abort.transaction()
            return
        endif
    endif
    
    |* Success - record created with all DAL validations passed
    message("Customer created successfully")
}
```

---

## 🔶 Example: Update with Retry Logic

```baan
on.choice.save:
    db.retry.point()
    
    |* Update main table through DAL
    ret = dal.update("tdsls400", main.table, error.occurred, true, db.retry)
    if ret <> 0 then
        if db.retry then
            return  |* Allow user to correct validation errors
        else
            abort.transaction()
            return
        endif
    endif
    
    |* Update related lines if main record saved successfully
    selectdo
    select  tdsls401.*
    from    tdsls401
    where   tdsls401.orno = tdsls400.orno
    and     tdsls401.pono = tdsls400.pono
    selectempty
        |* No lines to update
    selectdo
        ret = dal.update("tdsls401", line.set, error.occurred, true, db.retry)
        if ret <> 0 then
            if db.retry then
                return
            else
                abort.transaction()
                return
            endif
        endif
    endselect
    
    commit.transaction()
    message("Order updated successfully")
```

This ensures:

* If `dal.update()` fails (returns `DALHOOKERROR`), the user can correct and retry
* If successful, changes are committed
* Related records are updated in sequence
* Transaction is rolled back if any update fails

---

## 🔶 Example: Delete via DAL

```baan
on.choice.delete:
    |* Confirm deletion
    if not yesno("Delete this record?") then
        return
    endif
    
    db.retry.point()
    
    |* Delete through DAL
    ret = dal.delete("tccom100", main.table, error.occurred, true, db.retry)
    if ret <> 0 then
        if db.retry then
            return  |* Allow user to see error and cancel
        else
            abort.transaction()
            return
        endif
    endif
    
    commit.transaction()
    message("Record deleted successfully")
```

* Triggers `before.destroy.object()` and `after.destroy.object()`
* Aborts if deletion is not allowed (e.g., referenced records exist)

---

## 🔶 Handling Validation Errors

When a DAL hook (e.g., `field.is.valid()` or `before.save.object()`) fails:

* It returns `DALHOOKERROR`
* The engine prevents saving the record
* Error messages are displayed via `dal.set.error.message("...")`
* Developer must handle UI retry or exit path

### ➤ Advanced Error Handling Pattern

```baan
function save.with.validation()
{
    db.retry.point()
    
    |* Attempt to save through DAL
    ret = dal.update("tfinv100", invoice.set, error.occurred, true, db.retry)
    
    switch(ret)
    case 0:
        |* Success
        commit.transaction()
        message("Invoice saved successfully")
        break
        
    case DALHOOKERROR:
        if db.retry then
            |* Validation error - display message and allow retry
            message("Validation failed: " & dal.get.error.message())
            return  |* User can correct and try again
        else
            |* Fatal validation error
            message("Fatal validation error - cannot save")
            abort.transaction()
        endif
        break
        
    default:
        |* System error
        message("System error occurred: " & str$(ret))
        abort.transaction()
        break
    endswitch
}
```

---

## 🔶 Business Method Call from Session

```baan
form.command.post:
    ret = post.transaction()
    
    switch(ret)
    case 0:
        message("Transaction posted successfully")
        break
    case DALHOOKERROR:
        message("Posting failed: " & dal.get.error.message())
        break
    default:
        message("System error during posting: " & str$(ret))
        break
    endswitch

function extern long post.transaction()
{
    |* Security and applicability checks
    if post.transaction.is.never.allowed() then
        dal.set.error.message("Posting not available for this transaction type")
        return(DALHOOKERROR)
    endif
    
    if not post.transaction.is.allowed() then
        dal.set.error.message("Transaction cannot be posted at this time")
        return(DALHOOKERROR)
    endif
    
    |* Business logic
    db.retry.point()
    
    |* Update transaction status
    tfgld100.posted = tcyesno.yes
    tfgld100.post.date = date.num()
    tfgld100.post.user = logname$
    
    ret = dal.update("tfgld100", o.set, error.occurred, true, db.retry)
    if ret <> 0 then
        return(ret)
    endif
    
    |* Generate posting entries
    ret = generate.posting.entries()
    if ret <> 0 then
        abort.transaction()
        return(ret)
    endif
    
    commit.transaction()
    return(0)
}
```

The DAL method `post.transaction()` should internally call:

* `post.transaction.is.allowed()`
* `post.transaction.is.never.allowed()`

Before proceeding with any logic.

---

## 🔶 Advanced Integration Patterns

### ➤ Multi-Table Transaction Pattern

```baan
function process.order.with.lines()
{
    db.retry.point()
    
    |* Save order header
    ret = dal.update("tdsls400", header.set, error.occurred, true, db.retry)
    if ret <> 0 then
        if db.retry then return endif
        abort.transaction()
        return(ret)
    endif
    
    |* Process each order line
    for i = 1 to lines.count()
        ret = dal.update("tdsls401", line.sets(i), error.occurred, true, db.retry)
        if ret <> 0 then
            if db.retry then return endif
            abort.transaction()
            return(ret)
        endif
    endfor
    
    |* Call business method for order processing
    ret = process.order.logic()
    if ret <> 0 then
        abort.transaction()
        return(ret)
    endif
    
    commit.transaction()
    return(0)
}
```

### ➤ Conditional DAL Operations

```baan
function update.customer.with.credit.check()
{
    |* Read current state
    ret = dal.read("tccom100", customer.set, error.occurred, tccom100.cuno)
    if ret <> 0 then
        return(ret)
    endif
    
    |* Check if credit limit changed
    if tccom100.limit.changed then
        |* Recalculate credit status through business method
        ret = recalculate.credit.status()
        if ret <> 0 then
            return(ret)
        endif
    endif
    
    |* Update customer
    ret = dal.update("tccom100", customer.set, error.occurred, true, db.retry)
    if ret <> 0 then
        if db.retry then return endif
        abort.transaction()
        return(ret)
    endif
    
    commit.transaction()
    return(0)
}
```

---

## 🧠 Best Practices

### ✅ Do:
* **Always use DAL functions**: `dal.insert()`, `dal.update()`, `dal.delete()` instead of direct `db.*` operations
* **Use `commit.transaction()`** only after DAL operations have succeeded
* **Use `db.retry.point()`** to safely allow retry after validation errors
* **Check return codes** from all DAL operations and handle errors appropriately
* **Business methods must always check** `*.is.allowed()` and `*.is.never.allowed()`
* **Use transaction boundaries** properly with `db.retry.point()` and `commit.transaction()`

### ❌ Don't:
* **Never call `db.write`, `db.delete`, `db.update`** directly on DAL-managed tables
* **Don't bypass DAL validation** by using direct database operations
* **Don't forget error handling** - always check DAL return codes
* **Don't mix DAL and direct database operations** within the same transaction
* **Don't call DAL hooks directly** - let the engine invoke them

---

## ✅ Summary: DAL Invocation in Sessions

| Action        | 4GL UI Code            | DAL Hook Triggered                                                                           |
| ------------- | ---------------------- | -------------------------------------------------------------------------------------------- |
| Add record    | `dal.insert(tmain)`    | `before.new.object()`, `set.object.defaults()`, `after.new.object()`, `before.save.object()` |
| Change record | `dal.update(tmain)`    | `before.change.object()`, `after.change.object()`, `before.save.object()`                    |
| Delete record | `dal.delete(tmain)`    | `before.destroy.object()`, `after.destroy.object()`                                          |
| Save/commit   | `commit.transaction()` | Commits data if DAL returned 0, calls `after.commit.transaction()`                          |
| Cancel        | `abort.transaction()`  | Calls `after.abort.transaction()` if defined                                                 |
| Form command  | `my.method()`          | Calls `my.method.is.allowed()` and executes logic                                            |

---

# DAL / DAL2 Appendix: Hooks Reference

This file provides a compact reference of all **DAL** and **DAL2** hooks supported by the 4GL engine in Infor ERP. It acts as a cheat sheet for developers integrating data access logic with UI scripts.

---

## 🔷 Object Hooks

| Hook Name                  | When Called                            | Purpose                                | DAL2 Only |
|---------------------------|----------------------------------------|----------------------------------------|-----------|
| `before.open.object.set()`| Before object set is opened            | Initialization, parameter loading, field dependencies | No        |
| `before.get.object(dir)`  | Before reading object (GET)            | Block specific reads, security checks  | No        |
| `after.get.object(dir)`   | After reading object (GET)             | Filter out rows, post-read processing  | No        |
| `before.new.object()`     | User initiates insert                  | Signal start of new object, validation | ✅ Yes     |
| `after.new.object()`      | Before save of new object              | End of new phase, final setup         | ✅ Yes     |
| `before.change.object()`  | User begins update                     | Signal start of update, lock validation| ✅ Yes     |
| `after.change.object()`   | Before save of changed object          | End of update phase, change processing | ✅ Yes     |
| `before.save.object(mode)`| Before save (insert/update)            | Validate record integrity, business rules| No        |
| `after.save.object(mode)` | After save (insert/update)             | Cascade changes, trigger related logic| No        |
| `before.destroy.object()` | Before delete                          | Block deletion, referential checks    | No        |
| `after.destroy.object()`  | After delete                           | Cleanup or cascade delete              | No        |
| `after.commit.transaction()` | After transaction commit            | Cross-table updates post-save          | No        |
| `after.abort.transaction()`  | After transaction rollback          | Cleanup on abort                       | No        |
| `set.object.defaults()`   | During insert initialization           | Assign default values                  | No        |
| `method.is.allowed(m)`    | Before UI enables methods              | Enable/disable standard methods        | No        |

### ➤ Object Hook Usage Examples

```baan
|* Complete object lifecycle example
function extern long before.save.object(long mode)
{
    if mode = DAL_NEW then
        |* New record validation
        if empty(tccom100.nama) then
            dal.set.error.message("Customer name is required")
            return(DALHOOKERROR)
        endif
    else
        |* Update validation
        if tccom100.stat = tccom.stat.blocked then
            dal.set.error.message("Cannot modify blocked customer")
            return(DALHOOKERROR)
        endif
    endif
    return(0)
}

function extern long after.save.object(long mode)
{
    |* Update customer statistics
    update.customer.statistics(tccom100.cuno)
    
    |* Send notification for new customers
    if mode = DAL_NEW then
        send.new.customer.notification(tccom100.cuno)
    endif
    
    return(0)
}
```

---

## 🔷 Property (Field) Hooks

| Hook Name                  | Purpose                                  | DAL2 Only |
|---------------------------|------------------------------------------|-----------|
| `<field>.set.defaults()`  | Set default values during initialization | No        |
| `<field>.make.valid()`    | Adjust field to valid format (e.g., round off) | No   |
| `<field>.check()`         | Validate field value before save         | No        |
| `<field>.is.applicable()` | Enable/disable field dynamically         | ✅ Yes     |
| `<field>.is.readonly()`   | Make field readonly conditionally        | ✅ Yes     |
| `<field>.is.derived()`    | Make field derived (calculated)          | ✅ Yes     |
| `<field>.is.valid()`      | Custom validation logic                  | ✅ Yes     |
| `<field>.is.mandatory()`  | Determine if field is mandatory          | ✅ Yes     |
| `<field>.update()`        | Calculate or update field value          | ✅ Yes     |
| `<field>.enum.constant.is.applicable()` | Enable specific enum values | ✅ Yes     |
| `<field>.is.never.applicable()` | Disable field permanently         | ✅ Yes     |

### ➤ Field Hook Usage Examples

```baan
|* Classic DAL property hooks
function extern void tccom100.cdit.set.defaults()
{
    tccom100.cdit = 30  |* Default credit days
}

function extern long tccom100.cdit.check()
{
    if tccom100.cdit < 0 or tccom100.cdit > 365 then
        dal.set.error.message("Credit days must be between 0 and 365")
        return(DALHOOKERROR)
    endif
    return(0)
}

|* DAL2 field hooks
function extern boolean tdsls401.disc.is.applicable()
{
    |* Discount only for premium customers
    return(customer.is.premium(tdsls401.cuno))
}

function extern void tdsls401.amnt.update()
{
    if dal.any.parent.changed() then
        dal.require.field("tdsls401.qtyord")
        dal.require.field("tdsls401.pric")
        dal.require.field("tdsls401.disc")
        
        tdsls401.amnt = (tdsls401.qtyord * tdsls401.pric) * 
                        (1 - tdsls401.disc / 100)
    endif
}
```

---

## 🔷 DAL2 Field Dependency Management

| Function                        | Description                                                         |
|---------------------------------|---------------------------------------------------------------------|
| `dal.field.depends.on()`        | Declare dependency relationships between fields and hooks           |
| `dal.require.field()`           | Enforce evaluation order in `update()` hooks                       |
| `dal.any.parent.changed()`      | Check if a parent field (dependency) has changed                    |
| `dal.parent.caused.update(fld)` | Check if `fld` triggered current update call                        |

🔸 Must be called in `before.open.object.set()` or inside `field.update()` depending on context.

### ➤ Dependency Management Examples

```baan
function extern long before.open.object.set()
{
    |* Complex field dependency network
    dal.field.depends.on("tdsls401.amnt",
        HOOK_UPDATE, "tdsls401.qtyord", "tdsls401.pric", "tdsls401.disc")
    
    dal.field.depends.on("tdsls401.disc", 
        HOOK_IS_APPLICABLE + HOOK_UPDATE, "tdsls401.cuno", "tdsls401.item")
    
    dal.field.depends.on("tdsls401.ddat",
        HOOK_UPDATE, "tdsls401.item", "tdsls401.qtyord")
    
    return(0)
}

function extern void tdsls401.disc.update()
{
    if dal.parent.caused.update("tdsls401.cuno") then
        |* Customer changed - get customer discount
        tdsls401.disc = get.customer.discount(tdsls401.cuno)
    endif
    
    if dal.parent.caused.update("tdsls401.item") then
        |* Item changed - check if discount still valid
        if not item.allows.discount(tdsls401.item) then
            tdsls401.disc = 0
        endif
    endif
}
```

---

## 🔷 DAL2 Validation and Update Flows

These functions are called internally by the 4GL engine when field-level validation or update is triggered:

| Function Name          | Description                                         |
|------------------------|-----------------------------------------------------|
| `dal.validate.field(mode)` | Runs field validation in correct hook order     |
| `dal.update.field(mode)`   | Calls update logic and adjusts field if needed  |
| `dal.get.field.state(mode)`| Returns field status: ENABLED / READONLY / DISABLED |

### ➤ Internal Engine Flow (Reference Only)

```baan
|* These are called by the engine - DO NOT call directly
dal.validate.field(DAL_UPDATE)  |* Engine validates field using hooks
dal.update.field(DAL_NEW)       |* Engine updates field using dependencies  
dal.get.field.state(DAL_UPDATE) |* Engine determines UI field state
```

---

## 🔷 DAL2 Business Method Hooks

Used to control execution and UI visibility of form commands tied to business methods.

| Hook Name                          | Purpose                                               |
|-----------------------------------|-------------------------------------------------------|
| `business.method.is.allowed()`    | Return `true`/`false` to enable or disable UI button |
| `business.method.is.never.allowed()` | Remove form command entirely from UI              |

📝 Tip: Use `dal.set.error.message()` inside these hooks for user-friendly feedback.

### ➤ Business Method Examples

```baan
function extern boolean post.order.is.never.allowed()
{
    |* Hide posting for draft orders
    if tdsls400.stat = tdsls.stat.draft then
        return(true)
    endif
    
    |* Hide if posting module not implemented
    if not posting.module.available() then
        return(true)
    endif
    
    return(false)
}

function extern boolean post.order.is.allowed()
{
    |* Check order status
    if tdsls400.stat <> tdsls.stat.confirmed then
        dal.set.error.message("Order must be confirmed before posting")
        return(false)
    endif
    
    |* Check user permissions
    if not user.has.permission("POST_ORDERS") then
        dal.set.error.message("Insufficient privileges to post orders")
        return(false)
    endif
    
    |* Check order completeness
    if not order.is.complete(tdsls400.orno) then
        dal.set.error.message("Order is incomplete - cannot post")
        return(false)
    endif
    
    return(true)
}

function extern long post.order()
{
    |* Security checks
    if post.order.is.never.allowed() or not post.order.is.allowed() then
        dal.set.error.message("Order posting not allowed")
        return(DALHOOKERROR)
    endif
    
    |* Business logic
    db.retry.point()
    
    |* Update order status
    tdsls400.stat = tdsls.stat.posted
    tdsls400.post.date = date.num()
    tdsls400.post.user = logname$
    
    ret = dal.update("tdsls400", o.set, error.occurred, true, db.retry)
    if ret <> 0 then
        return(ret)
    endif
    
    |* Generate accounting entries
    ret = generate.accounting.entries()
    if ret <> 0 then
        abort.transaction()
        return(ret)
    endif
    
    commit.transaction()
    return(0)
}
```

---

## 🔷 Utility Functions

| Function Name                | Purpose                                            |
|-----------------------------|----------------------------------------------------|
| `dal.set.error.message()`   | Push an error message from DAL to UI               |
| `dal.set.warning.message()` | Push a warning message to UI                      |
| `dal.set.info.message()`    | Push an informational message to UI              |
| `dal.get.error.message()`   | Retrieve current error message                    |

### ➤ Message Handling Examples

```baan
function extern long validate.credit.limit()
{
    if tccom100.cmlm > company.max.credit.limit then
        dal.set.error.message("Credit limit exceeds company maximum: " & 
                             str$(company.max.credit.limit))
        return(DALHOOKERROR)
    endif
    
    if tccom100.cmlm > tccom100.turnover * 2 then
        dal.set.warning.message("Credit limit is high relative to turnover")
        |* Warning doesn't block save, just informs user
    endif
    
    return(0)
}
```

---

## 🔷 DAL Function Reference

| DAL Function               | Purpose                                    | Usage Context          |
|---------------------------|--------------------------------------------|------------------------|
| `dal.insert(table)`       | Insert new record through DAL             | UI sessions, business logic |
| `dal.update(table, set)`  | Update existing record through DAL        | UI sessions, business logic |
| `dal.delete(table, set)`  | Delete record through DAL                 | UI sessions, business logic |
| `dal.read(table, set)`    | Read record through DAL                   | Business logic, validation |
| `dal.new(table, set)`     | Create new object set                     | Object initialization      |

### ➤ DAL Function Usage

```baan
|* Standard DAL operation pattern
function save.customer()
{
    db.retry.point()
    
    ret = dal.update("tccom100", customer.set, error.occurred, true, db.retry)
    if ret = 0 then
        commit.transaction()
        message("Customer saved successfully")
    else
        if db.retry then
            |* Validation error - allow retry
            return
        else
            |* Fatal error
            abort.transaction()
            message("Failed to save customer")
        endif
    endif
}
```

---

## 🧠 Final Notes

- Only **table fields** are handled in DAL. Non-table fields must be validated in 4GL scripts.
- **DAL2** is always preferred over classic DAL for new development.
- **UI ↔ DAL** is a one-way interaction. UI scripts can call DAL, but DAL **cannot call UI**.
- Use `#include <bic_dal>` in every DAL script.
- **Always check return codes** from DAL operations and handle errors appropriately.
- **Use proper transaction management** with `db.retry.point()`, `commit.transaction()`, and `abort.transaction()`.

---

## ✅ DAL2 Hook Integration Summary

| Category     | Classic DAL Hooks             | DAL2 Extensions                             |
|--------------|-------------------------------|---------------------------------------------|
| Object       | `before.get.object`, etc.     | `before.new.object`, `after.change.object`  |
| Property     | `check()`, `make.valid()`     | `is.applicable()`, `is.readonly()`, `update()` |
| Field Logic  | —                             | `dal.require.field()`, `dal.any.parent.changed()` |
| Business     | —                             | `business.method.is.allowed()`              |
| Validation   | Manual field checks           | `dal.validate.field()`, automatic engine calls |
| UI Integration| Direct database calls        | Complete DAL abstraction layer              |

---

## 🎯 Complete DAL Development Workflow

### 1. **Define Object Structure**
```baan
#include <bic_dal>

function extern long before.open.object.set()
{
    |* Define field dependencies
    dal.field.depends.on("total", HOOK_UPDATE, "qty", "price", "discount")
    return(0)
}
```

### 2. **Implement Object Lifecycle**
```baan
function extern long before.save.object(long mode)
{
    |* Business validation
    return(validate.business.rules())
}

function extern long after.save.object(long mode)
{
    |* Cascade updates
    return(update.related.records())
}
```

### 3. **Add Field Behavior**
```baan
function extern boolean discount.is.applicable()
{
    return(customer.allows.discount(order.customer))
}

function extern void total.update()
{
    if dal.any.parent.changed() then
        total = qty * price * (1 - discount / 100)
    endif
}
```

### 4. **Implement Business Methods**
```baan
function extern boolean post.order.is.allowed()
{
    return(order.status = "CONFIRMED" and user.has.permission("POST"))
}

function extern long post.order()
{
    |* Security + business logic
    return(execute.posting.logic())
}
```

### 5. **UI Integration**
```baan
|* In 4GL session
on.choice.save:
    db.retry.point()
    ret = dal.update("order.table", main.set, error, true, db.retry)
    if ret = 0 then
        commit.transaction()
    endif

form.command.post:
    ret = post.order()
```

This concludes the **complete reference** of DAL and DAL2 hooks and integration patterns. For detailed examples and advanced patterns, refer to the comprehensive DAL documentation series.

---

## Related Documentation

- [Baan DAL Architecture](Baan_DAL_Architecture.md) - Complete DAL system overview and architecture
- [DAL Object Hooks](Baan_DAL_Object_Hooks.md) - Complete object lifecycle hook management
- [DAL Property Hooks](Baan_DAL_Property_Hooks.md) - Classic DAL field-level hooks and validation
- [DAL2 Field Hooks](Baan_DAL2_Field_Hooks.md) - Advanced DAL2 field behavior and dependencies
- [DAL2 Validation Flows](Baan_DAL2_Validation_Flows.md) - Engine integration and business method control
- [4GL Quick Reference](4GL_Quick_Reference.md) - Core language syntax and limitations
- [Review Standards](review-standards.md) - Development best practices and standards 