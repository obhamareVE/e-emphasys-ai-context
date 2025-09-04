# DAL2 Validation and Update Flows

DAL2 introduces a structured and engine-driven way of validating and updating fields. This ensures that:
- The UI always reflects valid and consistent field states.
- Validations are triggered automatically when a field is changed.
- Dependencies between fields are respected.

This file explains how the 4GL engine interacts with DAL2 during user interaction and integration.

---

## 🔶 When DAL2 Hooks Are Called by the 4GL Engine

The 4GL engine invokes specific DAL2 hooks at strategic points in the session lifecycle. These hook calls enable the engine to:

- Automatically disable/enable fields
- Validate user input before save
- Set default or derived values
- Trigger field dependencies

### ➤ UI Event Sections and DAL2 Hook Mapping

| **UI Section**               | **DAL2 Hooks Called**                                                                 |
|-----------------------------|----------------------------------------------------------------------------------------|
| `after.form.read:`          | `field.is.never.applicable()`, `business.method.is.never.allowed()`                  |
| `before.new.object:`        | `set.object.defaults()`                                                               |
| `before.display.object:`    | `field.is.applicable()`, `field.is.readonly()`, `field.is.derived()`<br>`field.enum.constant.is.applicable()`<br>`method.is.allowed()`<br>`business.method.is.allowed()` |
| `choice.mark.occur:after.choice:` | `method.is.allowed()`, `business.method.is.allowed()`                             |
| `field.<name>:check.input:` | `field.is.never.applicable()`, `field.is.applicable()`, `field.is.readonly()`<br>`field.is.derived()`, `field.is.mandatory()`, `field.is.valid()` or `field.enum.constant.is.applicable()` |
| `field.<name>:when.field.changes:` | `field.update()` and all dependencies (e.g., `is.applicable()`, `is.readonly()`, etc.) |
| `main.table.io:read.view:`  | `method.is.allowed()`, `business.method.is.allowed()`                                |

### ➤ Hook Execution Context

Understanding when hooks execute is critical for proper DAL2 design:

```baan
|* Form load sequence
after.form.read: 
    |* Engine calls: field.is.never.applicable(), business.method.is.never.allowed()
    |* Purpose: Hide fields/methods that should never appear

before.display.object:
    |* Engine calls: field.is.applicable(), field.is.readonly(), field.is.derived()
    |* Purpose: Set field states (enabled/disabled/readonly)

|* Field change sequence  
field.tdsls401.item:when.field.changes:
    |* Engine calls: field.update() for dependent fields
    |* Engine calls: field.is.applicable(), field.is.readonly() for affected fields
    |* Purpose: Cascade changes through field dependency network
```

---

## 🔶 Field Validation Flow: `dal.validate.field(mode)`

The 4GL engine uses this flow to check if a field's value is valid.

```baan
dal.validate.field(long mode)
{
    if field is empty then
        if field.is.applicable() and not field.is.never.applicable() then
            if mode = DAL_UPDATE and field.is.readonly() then
                return(DALHOOKERROR)
            endif
            if field.is.derived() or field.is.mandatory() then
                return(DALHOOKERROR)
            endif
        endif
    else
        if field is enum then
            if value ≠ default then
                if field.is.never.applicable() or not field.is.applicable() then
                    return(DALHOOKERROR)
                endif
            endif
            if mode = DAL_UPDATE and field.is.readonly() then
                return(DALHOOKERROR)
            endif
            if field.is.derived() or not field.enum.is.applicable() then
                return(DALHOOKERROR)
            endif
        else
            if mode = DAL_UPDATE and field.is.readonly() then
                return(DALHOOKERROR)
            endif
            if field.is.derived() or not field.is.valid() then
                return(DALHOOKERROR)
            endif
        endif
    endif
    return(0)  |* Field is valid
}
```

### ➤ Validation Logic Breakdown

1. **Empty Field Validation**:
   - Check if field is applicable and not never applicable
   - Validate readonly constraints in update mode
   - Ensure mandatory fields are not empty
   - Block user input to derived fields

2. **Non-Empty Field Validation**:
   - **Enum fields**: Validate enum value is applicable and not readonly/derived
   - **Regular fields**: Validate business rules via `field.is.valid()`

3. **Mode-Specific Rules**:
   - `DAL_NEW`: More permissive (allows setting initial values)
   - `DAL_UPDATE`: Stricter (enforces readonly constraints)

---

## 🔶 Field Update Flow: `dal.update.field(mode)`

This is used to automatically update derived fields based on parent changes.

```baan
dal.update.field(long mode)
{
    if field.is.never.applicable() or not field.is.applicable() then
        make field empty
    else
        if mode = DAL_NEW or not field.is.readonly() then
            call field.update()
            call field.make.valid()
        endif
    endif
}
```

### ➤ Update Flow Logic

1. **Applicability Check**: If field is not applicable, clear its value
2. **Editability Check**: Only update if field is editable (not readonly in update mode)
3. **Value Calculation**: Call `field.update()` to calculate new value
4. **Value Normalization**: Call `field.make.valid()` to normalize (round, format, etc.)

### ➤ Example Integration
```baan
function extern void tdsls401.amnt.update()
{
    |* Calculate line total from quantity * price * (1 - discount%)
    if dal.any.parent.changed() then
        dal.require.field("tdsls401.qtyord")
        dal.require.field("tdsls401.pric") 
        dal.require.field("tdsls401.drat")
        
        tdsls401.amnt = (tdsls401.qtyord * tdsls401.pric) * 
                        (1 - tdsls401.drat / 100)
    endif
}
```

---

## 🔶 Get Field UI State: `dal.get.field.state(mode)`

This returns the UI state (DISABLED / READONLY / ENABLED) based on DAL2 hooks.

```baan
dal.get.field.state(long mode)
{
    if not field.is.applicable() then
        return(DISABLED)
    else
        if (mode = DAL_UPDATE and field.is.readonly()) or field.is.derived() then
            return(READONLY)
        else
            return(ENABLED)
        endif
    endif
}
```

### ➤ UI State Logic

| Field State | Condition | UI Behavior |
|------------|-----------|-------------|
| **DISABLED** | `field.is.applicable() = false` | Field is grayed out, not visible |
| **READONLY** | `field.is.readonly() = true` OR `field.is.derived() = true` | Field is visible but not editable |
| **ENABLED** | Field is applicable and editable | Field accepts user input |

### ➤ State Priority Order
```
DISABLED > READONLY > ENABLED
```

---

## 🔶 Advanced Validation Patterns

### ➤ Cross-Field Validation
```baan
function extern boolean tdsls401.edat.is.valid(long mode)
{
    |* End date must be after start date
    if tdsls401.edat <= tdsls401.sdat then
        dal.set.error.message("End date must be after start date")
        return(false)
    endif
    return(true)
}
```

### ➤ Conditional Mandatory Fields
```baan
function extern boolean tdsls401.refno.is.mandatory()
{
    |* Reference number required for special order types
    if tdsls401.otyp = tdsls.otyp.special then
        return(true)
    endif
    return(false)
}
```

### ➤ Dynamic Field Applicability
```baan
function extern boolean tdsls401.disc.is.applicable()
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
```

---

## 🔶 Notes on `field.is.derived()`

This hook is only evaluated when:

* The DAL is not a SUBDAL (`SUBDAL = false`)
* The DAL is executing in the "Data Input" or "Integration" context

### ➤ Derived Field Example
```baan
function extern boolean tdsls401.total.is.derived()
{
    |* Total is always calculated, never user-entered
    return(true)
}

function extern void tdsls401.total.update()
{
    |* Calculate total automatically
    tdsls401.total = tdsls401.qty * tdsls401.price
}
```

---

# DAL2 Business Method Hooks

DAL2 introduces centralized control over *business methods* — operations that are typically invoked by form commands in the UI (such as posting, approval, generation, etc.).

DAL2 provides special hooks that allow you to dynamically:
- **Enable/disable** a business method (e.g., graying out a button)
- **Completely remove** the method from the form (e.g., hide the button)

These hooks are evaluated by the **4GL engine**, allowing consistent UI behavior and secure business logic enforcement.

---

## 🔶 What Is a Business Method?

A business method is a function defined in the DAL that can be called:
- From the UI (via a form command)
- From another script
- From integration

Examples:
```baan
function extern long post.transaction()
function extern long approve.invoice()
function extern long generate.lines()
function extern long calculate.taxes()
function extern long print.document()
```

Business methods represent **high-level business operations** that go beyond simple CRUD operations.

---

## 🔶 Hook 1: `business.method.is.allowed()`

### ✅ Purpose

Used to **enable/disable** the form command related to the business method. If this hook returns `false`, the button is shown but disabled in the UI.

### 🔧 Syntax

```baan
function boolean <methodname>.is.allowed()
```

### ✅ When It's Called

* Before `before.display.object`
* When field values change: `when.field.changes`
* In `choice.mark.occur:after.choice`
* During UI refresh operations

### ✅ Return Values

* `true` – method is allowed (button enabled)
* `false` – method is *not* allowed (button disabled/grayed out)

### 📝 Recommendation

Set an error/warning message to inform the user why the action is disallowed.

### 🧪 Example

```baan
function extern boolean post.transaction.is.allowed()
{
    if tfgld100.posted = tcyesno.yes then
        dal.set.error.message("tfglds0023")  |* Transaction is already posted.
        return(false)
    endif
    
    if tfgld100.amnt = 0 then
        dal.set.error.message("tfglds0024")  |* Cannot post zero amount.
        return(false)
    endif
    
    if not user.has.permission("POST_TRANSACTIONS") then
        dal.set.error.message("tfglds0025")  |* Access denied.
        return(false)
    endif
    
    return(true)
}
```

---

## 🔶 Hook 2: `business.method.is.never.allowed()`

### ❌ Purpose

Used to **completely remove** the form command (button) from the UI. Use this when the method is never relevant for the current record or system configuration.

### 🔧 Syntax

```baan
function boolean <methodname>.is.never.allowed()
```

### ✅ When It's Called

* After `after.form.read`
* Before the screen is drawn
* During initial form setup

### ✅ Return Values

* `true` – method must be removed from the UI (button hidden)
* `false` – method is potentially allowed (evaluated further via `is.allowed()`)

### 📝 Recommendation

Use this to hide methods that are never applicable (e.g., feature not implemented, wrong record type).

### 🧪 Example

```baan
function extern boolean post.transaction.is.never.allowed()
{
    if not tfgld100.finance_module_implemented then
        dal.set.error.message("tfglds0022")  |* Finance module not implemented.
        return(true)
    endif
    
    if tfgld100.ttype <> tfgld.ttype.journal then
        dal.set.error.message("tfglds0026")  |* Posting only for journal entries.
        return(true)
    endif
    
    return(false)
}
```

---

## 🔶 Complete Business Method Implementation

### ➤ Full Method Integration Example

```baan
function extern long post.transaction()
{
    |* Security and applicability check
    if post.transaction.is.never.allowed() then
        dal.set.error.message("tfglds0027")  |* Post transaction not available.
        return(DALHOOKERROR)
    endif
    
    if not post.transaction.is.allowed() then
        dal.set.error.message("tfglds0021")  |* Post transaction is not allowed.
        return(DALHOOKERROR)
    endif

    |* Business logic validation
    if not validate.posting.rules() then
        return(DALHOOKERROR)
    endif

    |* Actual posting logic
    db.retry.point()
    
    |* Update transaction status
    tfgld100.posted = tcyesno.yes
    tfgld100.post.date = date.num()
    tfgld100.post.user = logname$
    
    |* Save changes
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

function extern boolean validate.posting.rules()
{
    |* Check posting period
    if not period.is.open(tfgld100.peri) then
        dal.set.error.message("tfglds0028")  |* Period is closed.
        return(false)
    endif
    
    |* Check balance
    if abs(tfgld100.total.debit - tfgld100.total.credit) > 0.01 then
        dal.set.error.message("tfglds0029")  |* Transaction not balanced.
        return(false)
    endif
    
    return(true)
}
```

---

## 🔶 Business Method Security Pattern

```baan
function extern boolean approve.invoice.is.allowed()
{
    |* Status check
    if tfinv100.stat <> tfinv.stat.pending then
        dal.set.error.message("Invoice not in pending status")
        return(false)
    endif
    
    |* Amount threshold check
    if tfinv100.amnt > user.approval.limit() then
        dal.set.error.message("Amount exceeds approval limit")
        return(false)
    endif
    
    |* Role-based security
    if not user.has.role("INVOICE_APPROVER") then
        dal.set.error.message("Access denied: Insufficient privileges")
        return(false)
    endif
    
    |* Business rule: cannot approve own invoices
    if tfinv100.created.by = logname$ then
        dal.set.error.message("Cannot approve your own invoices")
        return(false)
    endif
    
    return(true)
}
```

---

## 🔶 Dynamic Method Visibility

```baan
function extern boolean generate.lines.is.never.allowed()
{
    |* Hide for manually entered orders
    if tdsls400.otyp = tdsls.otyp.manual then
        return(true)
    endif
    
    |* Hide if lines already exist
    if lines.exist.for.order(tdsls400.orno) then
        return(true)
    endif
    
    |* Hide if feature not enabled
    if not system.feature.enabled("AUTO_LINE_GENERATION") then
        return(true)
    endif
    
    return(false)
}

function extern boolean generate.lines.is.allowed()
{
    |* Check prerequisites
    if empty(tdsls400.cuno) then
        dal.set.error.message("Customer required for line generation")
        return(false)
    endif
    
    if tdsls400.stat <> tdsls.stat.open then
        dal.set.error.message("Order must be in open status")
        return(false)
    endif
    
    return(true)
}
```

---

## 🔶 Summary Table

| Hook                                 | Purpose                                | UI Behavior                | Called By Engine When...     |
| ------------------------------------ | -------------------------------------- | -------------------------- | ---------------------------- |
| `business.method.is.never.allowed()` | Check if method is *ever* applicable   | Removes button from UI     | After form read, initial setup |
| `business.method.is.allowed()`       | Check if method is *currently* allowed | Enables or disables button | Field change, object display |

### ➤ Hook Execution Order
```
1. business.method.is.never.allowed() → determines button visibility
2. business.method.is.allowed() → determines button state (enabled/disabled)
```

---

## 🔶 Integration with DAL2 Field Dependencies

Business method availability can depend on field values using DAL2 dependencies:

```baan
function extern long before.open.object.set()
{
    |* Method availability depends on status and amount fields
    dal.field.depends.on("post.transaction.button",
        HOOK_IS_APPLICABLE, "tfgld100.stat", "tfgld100.amnt")
    
    return(0)
}
```

---

## ✅ Summary

| DAL2 Hook                    | Called When...                                   | Purpose |
| ---------------------------- | ------------------------------------------------ | ------- |
| `field.is.applicable()`      | Field availability needs to be determined        | Field visibility control |
| `field.is.readonly()`        | Field editability is needed                      | Field state control |
| `field.is.derived()`         | Field is system-controlled, not user-entered     | System field marking |
| `field.is.mandatory()`       | Engine checks for required fields                | Required field validation |
| `field.is.valid()`           | Engine validates a user-entered field value      | Business rule validation |
| `field.enum.is.applicable()` | Engine filters enum list values                  | Enum value filtering |
| `field.update()`             | Triggered by changes in dependent parent fields  | Automatic calculation |
| `field.make.valid()`         | Adjusts field value post-update (e.g., rounding) | Value normalization |
| `business.method.is.never.allowed()` | Engine sets up UI form | Method visibility control |
| `business.method.is.allowed()` | Engine validates method availability | Method enablement control |

---

## 🧠 Design Principles

### ✅ Best Practices

1. **Separation of Concerns**:
   - Use `is.never.allowed()` for static/configuration-based visibility
   - Use `is.allowed()` for dynamic/data-dependent enablement

2. **Error Messages**:
   - Always set meaningful error messages in validation hooks
   - Use consistent message codes for similar conditions

3. **Performance**:
   - Keep hook logic lightweight (called frequently by engine)
   - Use `dal.require.field()` to optimize dependency evaluation

4. **Security**:
   - Implement both UI and method-level security checks
   - Never rely solely on UI state for security

### ❌ Common Pitfalls

- Don't perform expensive operations in frequently-called hooks
- Don't bypass method security checks in business method implementations
- Don't mix UI logic with business validation logic
- Don't create circular dependencies between field updates

---

## Related Documentation

- [DAL2 Field Hooks](Baan_DAL2_Field_Hooks.md) - Advanced field behavior and dependencies
- [DAL Object Hooks](Baan_DAL_Object_Hooks.md) - Complete object lifecycle hook management
- [DAL Property Hooks](Baan_DAL_Property_Hooks.md) - Classic DAL field-level hooks
- [Baan DAL Architecture](Baan_DAL_Architecture.md) - Complete DAL system overview
- [4GL Quick Reference](4GL_Quick_Reference.md) - Core language syntax
- [Review Standards](review-standards.md) - Development best practices 