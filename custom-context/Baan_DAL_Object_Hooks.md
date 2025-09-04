# DAL Object Hooks

Object hooks are the primary way to implement centralized integrity logic for inserts, updates, and deletes in DAL. These hooks are automatically called by the 4GL engine when operations are performed on a DAL-enabled table.

Each object hook is bound to a particular phase in the object lifecycle (insert, update, delete, read, etc.). The DAL engine executes them in a predefined order depending on the operation being performed.

---

## 🔹 Object Lifecycle Hooks

### `before.new.object(mode)`

Executed **before** a new record is created. Use this to override default values, set runtime constraints, or initialize context.

```4gl
function extern long before.new.object(long mode)
{
    if not sales.module.implemented() then
        dal.set.error.message("tdsls0001")
        return(DALHOOKERROR)
    endif
    return(0)
}
```

---

### `after.new.object(mode)`

Executed **after** a new object is initialized. Typically used to adjust fields based on interdependencies or load subdata.

```4gl
function extern long after.new.object(long mode)
{
    calculate.default.totals()
    return(0)
}
```

---

### `before.get.object(mode)`

Executed **before** an object is read from the database. Rarely needed, but can be used to prevent unauthorized reads.

```4gl
function extern long before.get.object(long mode)
{
    if not can.read.record() then
        dal.set.error.message("Access denied.")
        return(DALHOOKERROR)
    endif
    return(0)
}
```

---

### `after.get.object(mode)`

Executed **after** an object is read. Useful for preloading additional data, flags, or applying derived logic.

```4gl
function extern long after.get.object(long mode)
{
    |* Load calculated fields after reading
    calculate.balance.fields()
    load.reference.data()
    return(0)
}
```

---

### `before.save.object(mode)`

Executed **before** the object is written to the database. This is the primary hook for integrity checks.

```4gl
function extern long before.save.object(long mode)
{
    if tdsls401.amnt < 0 then
        dal.set.error.message("Amount must be positive.")
        return(DALHOOKERROR)
    endif
    return(0)
}
```

---

### `after.save.object(mode)`

Executed **after** a successful write to the database. Often used for adjusting other tables or final updates.

```4gl
function extern long after.save.object(long mode)
{
    dal.set.property("tdsls400", ttdsls400, "amnt",
        tdsls400.amnt - old.amnt + tdsls401.amnt, DAL_UPDATE)
    dal.update("tdsls400", ttdsls400, ret, true, db.retry)
    return(ret)
}
```

---

### `before.change.object(mode)`

Executed when an object is switched from view mode to change mode. You can block editing here.

```4gl
function extern long before.change.object(long mode)
{
    if record.is.locked() or not edit.allowed() then
        dal.set.error.message("Record cannot be modified")
        return(DALHOOKERROR)
    endif
    return(0)
}
```

---

### `after.change.object(mode)`

Executed after the object is placed in change mode. Used to preload or update data for edit.

```4gl
function extern long after.change.object(long mode)
{
    |* Initialize editing context
    backup.original.values()
    load.edit.permissions()
    return(0)
}
```

---

### `before.destroy.object(mode)`

Executed before an object is deleted. Use this for referential checks or to prevent deletion.

```4gl
function extern long before.destroy.object(long mode)
{
    if record.has.references() then
        dal.set.error.message("Cannot delete referenced record.")
        return(DALHOOKERROR)
    endif
    return(0)
}
```

---

### `after.destroy.object(mode)`

Executed after the object is deleted. Typically used for cleanup or cascading deletes.

```4gl
function extern long after.destroy.object(long mode)
{
    |* Clean up related records
    delete.audit.trail()
    cascade.delete.children()
    return(0)
}
```

---

## 🔹 Transaction and Set Hooks

### `before.open.object.set()`

Called when an object set is opened. This is the ideal place to declare DAL2 field dependencies using `dal.field.depends.on()`.

```4gl
function extern long before.open.object.set()
{
    dal.field.depends.on("total", HOOK_UPDATE, "qty", "price")
    return(0)
}
```

---

### `after.commit.transaction(mode)`

Executed after a transaction has been successfully committed.

```4gl
function extern long after.commit.transaction(long mode)
{
    |* Post-commit processing
    send.notification.emails()
    update.cache()
    return(0)
}
```

---

### `after.abort.transaction(mode)`

Executed if a transaction is rolled back. Useful for cleanup or messaging.

```4gl
function extern long after.abort.transaction(long mode)
{
    |* Transaction rollback cleanup
    cleanup.temp.files()
    log.rollback.reason()
    return(0)
}
```

---

## 🔹 Other Object Hooks

### `method.is.allowed()`

Used by the engine to determine if a standard method (e.g. update, delete) is allowed on this object.

```4gl
function extern boolean method.is.allowed()
{
    return user.has.permission("edit.sales.orders")
}
```

---

### `set.object.defaults(mode)`

This is the default-value initializer. It is invoked after the object is created but before input.

```4gl
function extern long set.object.defaults(long mode)
{
    |* Set system defaults
    tccom100.stat = "ACTIVE"
    tccom100.created.date = date.num()
    tccom100.created.by = logname$
    return(0)
}
```

---

## 🔹 DAL Mode Constants

Object hooks receive a `mode` parameter indicating the operation context:

| Mode Constant | Description | Usage |
|--------------|-------------|-------|
| `DAL_NEW` | New record creation | Insert operations |
| `DAL_UPDATE` | Existing record modification | Update operations |
| `DAL_DELETE` | Record deletion | Delete operations |
| `DAL_READ` | Record reading | Read operations |

Example usage:
```4gl
function extern long before.save.object(long mode)
{
    if mode = DAL_NEW then
        |* New record logic
        set.creation.audit()
    else
        |* Update record logic  
        set.modification.audit()
    endif
    return(0)
}
```

---

## 🔹 Hook Integration with Property Hooks

Object hooks and property hooks work together in a coordinated lifecycle:

### **Create Operation Flow:**
1. `before.new.object()` - Object initialization checks
2. `set.object.defaults()` - Set default values
3. `field.set.defaults()` - Set field-specific defaults (property hook)
4. `after.new.object()` - Post-initialization logic
5. User field changes trigger `field.make.valid()` and `field.check()` (property hooks)
6. `before.save.object()` - Final validation before save
7. Database write operation
8. `after.save.object()` - Post-save processing

### **Update Operation Flow:**
1. `before.change.object()` - Edit mode validation
2. `after.change.object()` - Edit mode initialization
3. Field changes trigger property hooks (`field.make.valid()`, `field.check()`)
4. `before.save.object()` - Pre-save validation
5. Database update operation
6. `after.save.object()` - Post-save processing

---

## 🔹 Summary Table: Execution Flow

| Operation  | Hook Sequence                                                                                                 |
| ---------- | ------------------------------------------------------------------------------------------------------------- |
| Create     | `before.new.object` → `set.object.defaults` → `after.new.object`                                              |
| Read       | `before.get.object` → read → `after.get.object`                                                               |
| Update     | `before.change.object` → change → `after.change.object`<br>`before.save.object` → write → `after.save.object` |
| Delete     | `before.destroy.object` → delete → `after.destroy.object`                                                     |
| Object Set | `before.open.object.set` → `after.commit.transaction` / `after.abort.transaction`                             |

---

## 🔹 Best Practices

### ✅ Do:
- Return `0` for success, `DALHOOKERROR` for failure
- Use `dal.set.error.message()` for meaningful error messages
- Implement business logic validation in `before.save.object()`
- Use `after.save.object()` for cascading updates to related tables
- Check the `mode` parameter to differentiate insert vs. update logic
- Use `before.open.object.set()` for DAL2 field dependencies

### ❌ Don't:
- Call object hooks directly - they're automatically invoked by the engine
- Perform database commits within hooks (handled by DAL engine)
- Use direct field assignment when property hooks exist
- Ignore return values from DAL operations in hooks
- Implement UI-specific logic in object hooks

---

## 🔹 Error Handling

```4gl
function extern long before.save.object(long mode)
{
    |* Multiple validation checks
    if tdsls401.amnt <= 0 then
        dal.set.error.message("Amount must be greater than zero")
        return(DALHOOKERROR)
    endif
    
    if not validate.customer() then
        dal.set.error.message("Invalid customer specified")
        return(DALHOOKERROR)
    endif
    
    |* All validations passed
    return(0)
}
```

---

## 🔹 Advanced Patterns

### **Conditional Hook Logic:**
```4gl
function extern long before.save.object(long mode)
{
    switch(mode)
    case DAL_NEW:
        validate.new.record()
        break
    case DAL_UPDATE:
        validate.changes()
        break
    default:
        break
    endswitch
    return(0)
}
```

### **Cross-Table Updates:**
```4gl
function extern long after.save.object(long mode)
{
    |* Update parent table totals
    dal.set.property("tdsls400", parent.set, "total.amount",
        parent.total + tdsls401.amnt, DAL_UPDATE)
    ret = dal.update("tdsls400", parent.set, error.occurred, true, db.retry)
    return(ret)
}
```

---

✅ You must **never call these hooks directly**. The 4GL engine calls them automatically during DAL operations like `dal.insert()`, `dal.update()`, or `dal.delete()`.

---

## Related Documentation

- [DAL Property Hooks](Baan_DAL_Property_Hooks.md) - Field-level validation and defaults
- [Baan DAL Architecture](Baan_DAL_Architecture.md) - Complete DAL system overview
- [4GL Quick Reference](4GL_Quick_Reference.md) - Core language syntax
- [Review Standards](review-standards.md) - Development best practices 