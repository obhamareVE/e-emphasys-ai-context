# DAL Property Hooks

In Infor ERP's original DAL (Data Access Layer) model, logic was defined at both the object level and the field (property) level. Property hooks define field-specific behaviors that are executed when a field's value changes, either through a UI session or from external calls (like BOI).

> ⚠️ Note: In modern implementations, DAL2 replaces these property hooks with a more powerful and reusable field-based hook model. However, understanding DAL property hooks is important for working with legacy DALs.

---

## Purpose of Property Hooks

Property hooks centralize the integrity logic for individual fields (properties). They are called automatically by the 4GL engine when insert, update, or create operations are triggered — provided the field is changed using the correct DAL API (`dal.set.property()`).

---

## Types of Property Hooks

### 1. `field.set.defaults()`

- **Purpose**: Assign a default value to a field.
- **Used By**: Business Object Interface (BOI) clients via `DAL_NEW` or `DAL_UPDATE`.
- **Typical Use Case**:
  ```c
  function extern void tdsls401.amnt.set.defaults()
  {
      tdsls401.amnt = 0
  }
  ```

---

### 2. `field.make.valid()`

* **Purpose**: Sanitize or normalize a field value before validation.
* **Does Not**: Perform validation itself.
* **Typical Use Case**:

  ```c
  function extern void tdsls401.amnt.make.valid()
  {
      tdsls401.amnt = round.amount(tdsls401.amnt)
  }
  ```

---

### 3. `field.check()`

* **Purpose**: Validate field logic when field is changed via DAL or BOI.
* **Replaces**: The traditional `check.input()` in UI scripts.
* **Typical Use Case**:

  ```c
  function extern long tdsls401.amnt.check()
  {
      if tdsls401.amnt < 0 then
          dal.set.error.message("tdsls4045")
          return(DALHOOKERROR)
      endif
      return(0)
  }
  ```

---

## Using `dal.set.property()` to Trigger Property Hooks

To ensure the above hooks are triggered correctly, you must **not** assign field values directly. Use:

```c
dal.set.property("tdsls401", object_set, "tdsls401.amnt", value, DAL_UPDATE)
```

> ❗ If you assign a field value directly or use `db.set()`, these hooks will **not be triggered**.

---

### Deprecated: `dal.set.property()` → Use `dal.set.field()` Instead

The function `dal.set.property()` is deprecated and replaced by `dal.set.field()` in DAL2 contexts. However, it's still found in legacy DALs.

---

### Immediate Check with `dal.set.property.with.check()`

If you need to call the check hook immediately, use:

```c
dal.set.property.with.check("table", object_set, "field", value, mode)
```

This ensures the `field.check()` hook is executed immediately after setting the value.

---

### Reading Property Flags: `dal.get.property.flag()`

You can retrieve a field's status using:

```c
flag = dal.get.property.flag("table", object_set, "field", FLAG_CHANGED)
```

This is useful for conditionally checking if a value was changed before validation or update logic.

---

## Property Hook Integration with Object Hooks

Property hooks work in conjunction with object-level DAL hooks:

1. **Field Assignment**: Use `dal.set.property()` or `dal.set.field()` to trigger property hooks
2. **Field Validation**: `field.check()` executes during object validation phase
3. **Object Save**: Property changes are committed during `before.save.object()` / `after.save.object()`

### Example Integration:
```c
// In UI script or business logic
dal.set.property("tccom100", o.set, "tccom100.nama", new.name, DAL_UPDATE)
// This triggers:
// 1. tccom100.nama.make.valid() - normalize the name
// 2. tccom100.nama.check() - validate the name
// Later during save:
// 3. before.save.object() - object-level validation
// 4. after.save.object() - object-level post-processing
```

---

## Best Practices

### ✅ Do:
- Always use `dal.set.property()` or `dal.set.field()` to modify field values
- Implement `field.check()` for business rule validation
- Use `field.make.valid()` for data normalization (rounding, formatting)
- Return `DALHOOKERROR` from `field.check()` on validation failure
- Use `dal.set.error.message()` to provide meaningful error messages

### ❌ Don't:
- Assign field values directly (bypasses property hooks)
- Use `db.set()` when DAL property hooks exist
- Perform complex business logic in `field.make.valid()`
- Forget to return proper error codes from `field.check()`

---

## Migration from Legacy to DAL2

When migrating from legacy DAL property hooks to DAL2:

1. **Replace** `dal.set.property()` with `dal.set.field()`
2. **Convert** property hooks to field-based hooks in DAL2 structure
3. **Consolidate** related field validations into reusable field hook libraries
4. **Test** thoroughly to ensure hook execution order is preserved

---

## Summary

| Hook                            | Purpose                     | When Used                       |
| ------------------------------- | --------------------------- | ------------------------------- |
| `field.set.defaults()`          | Assign default field value | During BOI inserts or `DAL_NEW` |
| `field.make.valid()`            | Round or normalize value    | Before field check              |
| `field.check()`                 | Validate field integrity    | On insert/update                |
| `dal.set.property()`            | Sets field & triggers hooks | From UI/DAL logic               |
| `dal.set.property.with.check()` | Sets field & runs check()   | From UI/DAL logic               |
| `dal.get.property.flag()`       | Check field status/flags    | For conditional logic           |

> ✅ Always use DAL functions to modify field values to ensure hooks are executed correctly.

---

## Related Documentation

- [Baan DAL Architecture](Baan_DAL_Architecture.md) - Complete DAL object-level hooks
- [4GL Quick Reference](4GL_Quick_Reference.md) - Core language syntax
- [Review Standards](review-standards.md) - Development best practices 