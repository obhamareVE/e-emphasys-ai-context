# DAL2 Field-Level Hooks

In DAL2, the field-level hooks replace the classic DAL `field.check()` mechanism. These hooks allow the 4GL engine to interact with fields intelligently by:

- Automatically disabling or enabling fields in the UI
- Validating field content and dependencies
- Calculating or clearing values based on business logic

DAL2 hooks are triggered by the engine at key moments like form load, field change, or object display. They are declared in the DAL script and always operate on the current object.

---

## 🔹 Hook: `field.is.never.applicable()`

### Purpose
Hides the field completely from the UI if it is never applicable.

### When called
- Just before `after.form.read`
- During value checking if the field is non-empty

### Return
- `true`: field must be empty and will be removed from UI
- `false`: field may be visible

### Example
```baan
function extern domain tcbool whinh200.sfrv.is.never.applicable()
{
    if whwmd000.lfcs = tcyesno.no then
        dal.set.error.message("whinhs0395")
        return(true)
    endif
    return(false)
}
```

---

## 🔹 Hook: `field.is.applicable()`

### Purpose
Indicates if the field should be enabled and retain a value. If not, the field is cleared and disabled.

### When called
* Before `before.display.object`
* On field change if dependencies exist

### Return
* `true`: keep field value, field is enabled
* `false`: field is cleared and disabled

### Example
```baan
function extern boolean whinh200.sfit.is.applicable()
{
    if whinh200.ittp <> whinh.ittp.item.transfer then
        dal.set.error.message("whinhs0239")
        return(false)
    endif
    return(true)
}
```

---

## 🔹 Hook: `field.is.readonly()`

### Purpose
Marks a field as read-only (uneditable) if business logic requires it.

### When called
* Before `before.display.object`
* During `when.field.changes` and validation

### Note
This is **not called** in `DAL_NEW` mode.

### Return
* `true`: field is read-only in update mode
* `false`: field is editable

### Example
```baan
function extern boolean whinh200.otyp.is.readonly()
{
    if not MANUAL.WHINH200 then
        dal.set.error.message("whinhs2359")
        return(true)
    endif
    return(false)
}
```

---

## 🔹 Hook: `field.is.derived()`

### Purpose
Indicates the field's value is derived by the DAL, not user input. It will be shown as read-only in the UI.

### Return
* `true`: field is derived and must not be manually entered
* `false`: field is user-editable or not derived

### Example
```baan
function extern boolean temmt020.sdat.is.derived()
{
    if temmt020.prnt = temmt001.euro then
        dal.set.error.message("tecals0493")
        return(true)
    endif
    return(false)
}
```

---

## 🔹 Hook: `field.is.mandatory()`

### Purpose
Marks the field as required — the field must not be empty.

### When called
* When field is empty
* Also used for visual cues (asterisks in UI)

### Return
* `true`: field is required
* `false`: field may be empty

### Example
```baan
function extern boolean whinh200.sfit.is.mandatory()
{
    if whinh200.ittp = whinh.ittp.item.transfer then
        dal.set.error.message("whinhs0181")
        return(true)
    endif
    return(false)
}
```

---

## 🔹 Hook: `field.is.valid()`

### Purpose
Performs business logic checks on non-enum fields.

### When called
* Only for non-enum fields
* Only when the field is **not empty**

### Return
* `true`: field passes validation
* `false`: field is not valid

### Example
```baan
function extern boolean whinh200.otyp.is.valid(long mode)
{
    if mode = 0 then return(true)
    if order.type.ok(whinh200.otyp, whinh200.ittp) <> 0 then
        return(false)
    endif
    return(true)
}
```

---

## 🔹 Hook: `field.enum.is.applicable()`

### Purpose
Specifies which enum values are allowed. Non-applicable values will not appear in dropdowns.

### When called
* Before `before.display.object`
* During enum mask setup

### Return
* `true`: enum constant allowed
* `false`: enum constant hidden

### Example
```baan
function extern boolean whinh200.oorg.sales.is.applicable(long mode)
{
    if not TD.IMPLEMENTED then
        dal.set.error.message("whinhs8532")
        return(false)
    endif
    return(true)
}
```

---

## 🔹 Hook: `field.update()`

### Purpose
Calculates or updates the field's value in response to changes in related fields. Eliminates most `when.field.changes` logic in UI.

### When called
* After field dependencies are triggered
* Before `when.field.changes` is called

### Example
```baan
function extern void whinh200.sfty.update()
{
    on case whinh200.ittp
    case whinh.ittp.receipt:
        if whinh200.order.origin.is(whinh200.oorg, ...) then
            whinh200.sfty = tctyps.work.center
        else
            whinh200.sfty = tctyps.partner
        endif
        break
    case whinh.ittp.issue:
    case whinh.ittp.transfer:
        whinh200.sfty = tctyps.warehouse
        break
    endcase
}
```

---

# DAL2 Field Dependencies

In DAL2, field dependencies allow the system to automatically trigger hooks like `update()` or `is.applicable()` for a field when one of its parent fields changes. This enables smarter behavior in the UI and ensures data consistency without needing explicit 4GL event code like `when.field.changes:`.

---

## 🔶 Purpose of Field Dependencies

DAL2 introduces field dependencies to:
- Determine default or derived values based on other fields
- Automatically re-evaluate field state (applicable, readonly, etc.)
- Replace logic from `when.field.changes` in 4GL sessions
- Improve testability and integration (used by 4GL engine, not just UI)

---

## 🔶 Defining Dependencies with `dal.field.depends.on()`

```baan
function extern long before.open.object.set()
{
    dal.field.depends.on("fieldA", HOOK_UPDATE, "fieldB")
    return(0)
}
```

### ➤ Syntax

```baan
dal.field.depends.on(
    const string fieldname,
    long hook.list,
    const string parent.field.name [, ...]
    [, long hook.list, const string parent.field.name, ...]
)
```

### ➤ Hook Constants

| Constant             | Affects Hook                                |
| -------------------- | ------------------------------------------- |
| `HOOK_IS_APPLICABLE` | `field.is.applicable()`                     |
| `HOOK_IS_READONLY`   | `field.is.readonly()`                       |
| `HOOK_IS_DERIVED`    | `field.is.derived()`                        |
| `HOOK_IS_VALID`      | `field.is.valid()` / `enum.is.applicable()` |
| `HOOK_IS_MANDATORY`  | `field.is.mandatory()`                      |
| `HOOK_UPDATE`        | `field.update()`                            |

---

## 🔶 Basic Dependency Example

Suppose we have a table `temmt020` with the following fields:

* `prnt`: Parent Currency
* `sdat`: Start Date
* `edat`: End Date

Start and End dates depend on Parent Currency.

```baan
function extern long before.open.object.set()
{
    dal.field.depends.on("temmt020.sdat",
        HOOK_IS_APPLICABLE + HOOK_UPDATE,   "temmt020.prnt")

    dal.field.depends.on("temmt020.edat",
        HOOK_IS_APPLICABLE + HOOK_UPDATE,   "temmt020.prnt",
        HOOK_UPDATE,                        "temmt020.sdat")

    return(0)
}
```

---

## 🔶 Controlling Update Order: `dal.require.field()`

To avoid cyclic or premature updates between dependent fields, use:

```baan
dal.require.field("other.field")
```

Use this inside `field.update()` hooks to ensure prerequisite fields are updated first.

### Example
```baan
function extern void whinh200.total.update()
{
    |* Ensure quantity and price are calculated first
    dal.require.field("whinh200.qty")
    dal.require.field("whinh200.price")
    
    |* Now calculate total
    whinh200.total = whinh200.qty * whinh200.price
}
```

---

## 🔶 Check Dependency Trigger: `dal.any.parent.changed()`

Within `field.update()` hooks, use this to verify if any parent field that the current field depends on has changed:

```baan
function extern void whinh200.discount.update()
{
    if dal.any.parent.changed() then
        |* Recalculate discount only if dependencies changed
        calculate.customer.discount()
    endif
}
```

---

## 🔶 Check Which Field Triggered Update: `dal.parent.caused.update()`

```baan
function extern void whinh200.status.update()
{
    if dal.parent.caused.update("whinh200.qty") then
        |* Quantity changed - check stock availability
        validate.stock.availability()
    endif
    
    if dal.parent.caused.update("whinh200.item") then
        |* Item changed - reset all related fields
        reset.item.dependent.fields()
    endif
}
```

This helps write smarter update logic depending on the actual trigger source.

---

## 🔶 Advanced Example: Mutually Dependent Fields

```baan
function extern long before.open.object.set()
{
    dal.field.depends.on("b", HOOK_UPDATE, "a", "c")
    dal.field.depends.on("c", HOOK_UPDATE, "a", "b")
    return(0)
}

function extern void b.update()
{
    dal.require.field("a")
    if a <= 10 then
        dal.require.field("c")
        if dal.any.parent.changed() then
            b = c
        endif
    else
        b = 0
    endif
}

function extern void c.update()
{
    dal.require.field("a")
    if a <= 10 then
        c = 6
    else
        dal.require.field("b")
        if dal.any.parent.changed() then
            c = b
        endif
    endif
}
```

---

## 🔶 Complex Business Logic Example

```baan
function extern long before.open.object.set()
{
    |* Order line total depends on quantity, price, and discount
    dal.field.depends.on("tdsls401.amnt",
        HOOK_UPDATE, "tdsls401.qtyord", "tdsls401.pric", "tdsls401.drat")
    
    |* Discount applicability depends on customer and item
    dal.field.depends.on("tdsls401.drat",
        HOOK_IS_APPLICABLE + HOOK_UPDATE, "tdsls401.cuno", "tdsls401.item")
    
    |* Delivery date depends on item and quantity
    dal.field.depends.on("tdsls401.ddat",
        HOOK_UPDATE, "tdsls401.item", "tdsls401.qtyord")
    
    return(0)
}

function extern void tdsls401.amnt.update()
{
    |* Calculate line amount
    dal.require.field("tdsls401.qtyord")
    dal.require.field("tdsls401.pric")
    dal.require.field("tdsls401.drat")
    
    if dal.any.parent.changed() then
        tdsls401.amnt = (tdsls401.qtyord * tdsls401.pric) * 
                       (1 - tdsls401.drat / 100)
    endif
}

function extern boolean tdsls401.drat.is.applicable()
{
    |* Discount only applicable for premium customers
    if not customer.is.premium(tdsls401.cuno) then
        return(false)
    endif
    
    |* And only for discountable items
    if not item.allows.discount(tdsls401.item) then
        return(false)
    endif
    
    return(true)
}

function extern void tdsls401.drat.update()
{
    if dal.parent.caused.update("tdsls401.cuno") then
        |* Customer changed - get customer discount
        tdsls401.drat = get.customer.discount(tdsls401.cuno)
    endif
    
    if dal.parent.caused.update("tdsls401.item") then
        |* Item changed - check if discount still applies
        if not item.allows.discount(tdsls401.item) then
            tdsls401.drat = 0
        endif
    endif
}
```

---

## ✅ DAL2 Field Hooks Summary

DAL2 field hooks make it possible to:

* Drive field behavior from the DAL instead of UI scripts
* Enforce business rules across all contexts (UI, integration, background)
* Define field logic like applicability, read-only state, defaults, and derived values
* Create sophisticated field dependency networks
* Replace complex `when.field.changes` logic with declarative dependencies

---

## ✅ Field Dependencies Summary

| Concept                      | Purpose                                                    |
| ---------------------------- | ---------------------------------------------------------- |
| `dal.field.depends.on()`     | Declare which fields a field depends on for specific hooks |
| `dal.require.field()`        | Enforce correct evaluation order between dependent fields  |
| `dal.any.parent.changed()`   | Check if a dependent field was changed                     |
| `dal.parent.caused.update()` | Identify specific field that caused update trigger         |

---

## 🔹 Best Practices

### ✅ Do:
- Use `dal.field.depends.on()` in `before.open.object.set()` to declare all dependencies
- Use `dal.require.field()` to ensure proper update order
- Check `dal.any.parent.changed()` before expensive calculations
- Use `dal.parent.caused.update()` for field-specific logic
- Return `true`/`false` consistently from field validation hooks
- Use `dal.set.error.message()` for user-friendly error messages

### ❌ Don't:
- Create circular dependencies without proper `dal.require.field()` controls
- Perform heavy calculations in field hooks without checking if needed
- Mix DAL2 field hooks with traditional `when.field.changes` logic
- Forget to handle all possible field states in hook logic
- Use direct field assignment when field dependencies exist

---

## Related Documentation

- [DAL Object Hooks](Baan_DAL_Object_Hooks.md) - Complete object lifecycle hook management
- [DAL Property Hooks](Baan_DAL_Property_Hooks.md) - Classic DAL field-level hooks
- [Baan DAL Architecture](Baan_DAL_Architecture.md) - Complete DAL system overview
- [4GL Quick Reference](4GL_Quick_Reference.md) - Core language syntax
- [Review Standards](review-standards.md) - Development best practices 