# Baan 4GL Advanced Development Patterns

## Complex Business Logic Patterns

### Multi-Table Validation Pattern
```baan
main.table.io:
before.write:
    |* Complex cross-table validation
    select count(*)
    from tcord100
    where tcord100.cust = tccom100.cust and tcord100.stat = "OPEN"
    selectdo
        if count(*) > 10 then
            select sum(tcord100.amnt)
            from tcord100  
            where tcord100.cust = tccom100.cust
            selectdo
                if sum(tcord100.amnt) > tccom100.crdt then
                    message("Customer credit limit exceeded")
                endif
            selectempty
            endselect
        endif
    selectempty
    endselect
```

### Conditional Field Logic Patterns
```baan
field.tccom100.cust.type:
when.field.changes:
    on case tccom100.cust.type
    case "CORPORATE":
        enable.fields("tccom100.tax.number", "tccom100.credit.rating")
        tccom100.payment.terms = 30
        disable.fields("tccom100.individual.id")
    case "INDIVIDUAL":
        disable.fields("tccom100.tax.number", "tccom100.credit.rating")
        tccom100.payment.terms = 15
        enable.fields("tccom100.individual.id")
    default:
        disable.fields("tccom100.tax.number", "tccom100.individual.id")
        tccom100.payment.terms = 0
    endcase
    
    |* Refresh dependent fields
    display("tccom100.payment.terms")
    display("tccom100.tax.number")
```

## Integration Patterns

### External System Interface Pattern
```baan
main.table.io:
after.write:
    |* Integration with external CRM system
    if tccom100.sync.required = tcyesno.yes then
        export.to.external.system()
    endif

function export.to.external.system()
{
    string json.data(1024)
    long result.code
    
    |* Build JSON payload
    json.data = "{" &
                "\"customer_id\":\"" & tccom100.cust & "\"," &
                "\"name\":\"" & tccom100.name & "\"," &
                "\"credit_limit\":" & str$(tccom100.crdt) &
                "}"
    
    |* Call external service
    result.code = http.post("https://crm.company.com/api/customers", json.data)
    
    if result.code = 200 then
        tccom100.last.sync = date.num()
        tccom100.sync.status = "SUCCESS"
    else
        tccom100.sync.status = "FAILED"
        message("Failed to sync customer data")
    endif
}
```

### Batch Processing Pattern
```baan
|* Background job processing pattern
domain long process.counter
domain long total.records
domain string process.status(20)

process.counter = 0
total.records = 0
process.status = "PROCESSING"

|* Count total records first
select count(*)
from tccom100
where tccom100.sync.required = tcyesno.yes
selectdo
    total.records = count(*)
selectempty
    message("No records to process")
    return
endselect

|* Process in batches
select tccom100.cust, tccom100.name, tccom100.crdt
from tccom100
where tccom100.sync.required = tcyesno.yes
order by tccom100.cust

selectdo
    process.counter = process.counter + 1
    
    |* Process individual record
    process.customer.record()
    
    |* Update progress every 100 records
    if (process.counter % 100) = 0 then
        message("Processed " & str$(process.counter) & " of " & str$(total.records))
    endif
    
    |* Commit every 500 records to avoid long transactions
    if (process.counter % 500) = 0 then
        commit.transaction()
        db.retry.point()
    endif

selectempty
endselect

process.status = "COMPLETED"
message("Processing completed: " & str$(process.counter) & " records")
```

## Performance Optimization Patterns

### Efficient Query Patterns
```baan
|* Use specific field selection instead of full table reads
select tccom100.cust, tccom100.name, tccom100.crdt
from tccom100
where tccom100.city = "NEW YORK" and tccom100.stat = 1
order by tccom100.name

selectdo
    |* Process only needed fields
    process.customer.summary(tccom100.cust, tccom100.name, tccom100.crdt)
selectempty
endselect

|* Avoid full table scans with proper indexing
select tccom100.cust, tccom100.balance
from tccom100
where tccom100.cust inrange "A000" and "A999"
and tccom100.balance > 1000.0
order by tccom100.cust

selectdo
    update.high.value.customer(tccom100.cust)
selectempty
endselect
```

### Memory-Efficient Processing
```baan
|* Process large datasets in chunks
domain long batch.size
domain long current.batch
domain string last.processed.key(20)

batch.size = 1000
current.batch = 0
last.processed.key = ""

while true do
    current.batch = 0
    
    select tccom100.cust, tccom100.balance
    from tccom100
    where tccom100.cust > last.processed.key
    order by tccom100.cust
    with 1000 rows
    
    selectdo
        current.batch = current.batch + 1
        process.customer.balance(tccom100.cust, tccom100.balance)
        last.processed.key = tccom100.cust
    selectempty
        |* No more records
        break
    endselect
    
    |* If less than batch size, we're done
    if current.batch < batch.size then
        break
    endif
    
    |* Commit batch and continue
    commit.transaction()
    db.retry.point()
endwhile
```

## Error Handling Patterns

### Comprehensive Error Management
```baan
main.table.io:
before.write:
    |* Centralized validation with detailed error reporting
    string validation.errors(1024)
    long error.count
    
    validation.errors = ""
    error.count = 0
    
    |* Collect all validation errors
    if tccom100.crdt < 0 then
        error.count = error.count + 1
        validation.errors = validation.errors & 
                           "Error " & str$(error.count) & ": Credit limit cannot be negative. "
    endif
    
    if tccom100.balc > tccom100.crdt then
        error.count = error.count + 1
        validation.errors = validation.errors &
                           "Error " & str$(error.count) & ": Balance exceeds credit limit. "
    endif
    
    if isspace(tccom100.name) then
        error.count = error.count + 1
        validation.errors = validation.errors &
                           "Error " & str$(error.count) & ": Customer name is required. "
    endif
    
    |* Display consolidated errors
    if error.count > 0 then
        message("Validation failed: " & validation.errors)
    endif
```

### Retry Logic Pattern
```baan
function safe.database.operation() : long
{
    long retry.count
    long max.retries
    long operation.result
    
    retry.count = 0
    max.retries = 3
    operation.result = 0
    
    while retry.count < max.retries do
        db.retry.point()
        
        select tccom100.cust, tccom100.balance
        from tccom100 for update
        where tccom100.cust = "CUST001"
        
        selectdo
            tccom100.balance = tccom100.balance + 100.0
            db.update(tccom100, DB.RETRY)
            
            if db.retry then
                retry.count = retry.count + 1
                if retry.count < max.retries then
                    message("Retry attempt " & str$(retry.count))
                    continue
                else
                    message("Max retries exceeded")
                    return(0)
                endif
            endif
            
            commit.transaction()
            operation.result = 1
            break
            
        selectempty
            message("Record not found")
            return(0)
        endselect
    endwhile
    
    return(operation.result)
}
```

## Security Patterns

### Role-Based Access Control
```baan
main.table.io:
before.read:
    |* Check user permissions before allowing read access
    if not check.user.permission("CUSTOMER_READ") then
        message("Access denied: Insufficient permissions")
        |* Prevent read operation
    endif
    
    |* Additional security for sensitive data
    if tccom100.confidential = tcyesno.yes then
        if not check.user.permission("CONFIDENTIAL_DATA") then
            message("Access denied: Confidential customer data")
        endif
    endif

before.write:
    |* Check modification permissions
    if not check.user.permission("CUSTOMER_MODIFY") then
        message("Access denied: Cannot modify customer data")
    endif
    
    |* Additional checks for financial data
    if tccom100.crdt <> old.tccom100.crdt then
        if not check.user.permission("CREDIT_LIMIT_MODIFY") then
            message("Access denied: Cannot modify credit limits")
        endif
    endif

function check.user.permission(permission.name) : long
{
    select count(*)
    from tccom150
    where tccom150.user = logname$ 
    and tccom150.permission = permission.name
    and tccom150.active = tcyesno.yes
    selectdo
        if count(*) > 0 then
            return(1)
        endif
    selectempty
    endselect
    
    return(0)
}
```

### Data Sanitization Pattern
```baan
field.tccom100.name:
check.input:
    |* Sanitize input to prevent injection
    string clean.name(100)
    
    clean.name = sanitize.string.input(tccom100.name)
    
    if clean.name <> tccom100.name then
        tccom100.name = clean.name
        message("Input sanitized for security")
        display("tccom100.name")
    endif

function sanitize.string.input(input.string) : string
{
    string clean.string(100)
    long i
    string char(1)
    
    clean.string = ""
    
    for i = 1 to len(input.string) do
        char = input.string(i;1)
        
        |* Allow only alphanumeric and basic punctuation
        if (char >= "A" and char <= "Z") or
           (char >= "a" and char <= "z") or
           (char >= "0" and char <= "9") or
           char = " " or char = "." or char = "-" then
            clean.string = clean.string & char
        endif
    endfor
    
    return(clean.string)
}
```

## Form Design Patterns

### Dynamic Form Behavior
```baan
field.customer.type:
when.field.changes:
    |* Dynamic form layout based on customer type
    on case customer.type
    case "RETAIL":
        show.group(1)  |* Basic customer info
        hide.group(2)  |* Corporate info
        hide.group(3)  |* Wholesale info
        set.field.label("payment.terms", "Payment Days")
    case "CORPORATE":
        show.group(1)  |* Basic customer info
        show.group(2)  |* Corporate info
        hide.group(3)  |* Wholesale info
        set.field.label("payment.terms", "Net Payment Terms")
    case "WHOLESALE":
        show.group(1)  |* Basic customer info
        hide.group(2)  |* Corporate info
        show.group(3)  |* Wholesale info
        set.field.label("payment.terms", "Trade Terms")
    endcase
    
    |* Update field properties
    refresh.form.layout()
```

### Field Dependency Management
```baan
field.country.code:
when.field.changes:
    |* Clear dependent fields when country changes
    state.code = ""
    city.code = ""
    postal.format = ""
    
    |* Set country-specific defaults
    on case country.code
    case "US":
        postal.format = "NNNNN-NNNN"
        enable.fields("state.code")
    case "CA":
        postal.format = "ANA NAN"
        enable.fields("province.code")
        disable.fields("state.code")
    case "GB":
        postal.format = "AAAA NNN"
        disable.fields("state.code", "province.code")
    default:
        postal.format = ""
        disable.fields("state.code", "province.code")
    endcase
    
    |* Refresh dependent displays
    display("state.code")
    display("postal.format")
```

## Debugging and Troubleshooting Patterns

### Debug Logging Pattern
```baan
domain tcyesno debug.enabled
domain string debug.file(100)

|* Initialize debug settings
debug.enabled = tcyesno.no
debug.file = "/tmp/baan_debug_" & logname$ & ".log"

|* Check for debug environment variable
if getenv("BAAN_DEBUG") = "1" then
    debug.enabled = tcyesno.yes
endif

function debug.log(message.text)
{
    string timestamp(20)
    
    if debug.enabled = tcyesno.yes then
        timestamp = date.to.string(date.num()) & " " & time.to.string(time.num())
        
        |* Write to debug file
        write.to.file(debug.file, 
                      timestamp & " [" & logname$ & "] " & message.text)
    endif
}

|* Usage in field sections
field.tccom100.crdt:
when.field.changes:
    debug.log("Credit limit changed from " & str$(old.tccom100.crdt) & 
              " to " & str$(tccom100.crdt) & " for customer " & tccom100.cust)
    
    tccom100.margin = tccom100.price - tccom100.cost
    display("tccom100.margin")
```

## Best Practices Summary

### Code Organization
1. **Group related fields** - Organize field sections by functional area
2. **Consistent naming** - Use descriptive, consistent variable names
3. **Comment complex logic** - Document business rules and calculations
4. **Separate concerns** - Keep validation, calculation, and UI logic separate

### Performance Guidelines
1. **Minimize database calls** - Batch operations where possible
2. **Use specific field selection** - Avoid SELECT * patterns
3. **Implement proper indexing** - Use appropriate WHERE clauses
4. **Commit strategically** - Balance transaction size vs. lock duration

### Security Considerations
1. **Validate all inputs** - Sanitize user input data
2. **Check permissions** - Implement role-based access control
3. **Audit sensitive operations** - Log security-relevant actions
4. **Handle errors gracefully** - Don't expose system details

### Maintainability Principles
1. **Keep functions focused** - Single responsibility principle
2. **Use meaningful names** - Self-documenting code
3. **Implement proper error handling** - Comprehensive error management
4. **Document business rules** - Clear comments for complex logic

---

## DLL (General Library) Patterns

### DLL Architecture Overview
DLLs in Infor ERP are **General Library scripts** that provide reusable components with globally accessible functions. They promote modularity and code reuse across UI scripts, DAL, and other components.

### DLL Naming Convention
**Script naming pattern**: `<package><module>dll<dllName>`
```baan
|* Examples:
tsextdllglobalFunctionsDLL
tccomdllvalidationUtils
tinvdllcalculationEngine
```
**⚠️ CRITICAL**: NO dots in the actual script name

### DLL Function Definition Patterns

#### Global Function Syntax
```baan
|* Global function accessible from other scripts
extern function <domain> <package>.<module>.dll<dllName>.<function.name>()
{
    |* Function implementation
    return(value)
}
```

#### Local Function Syntax
```baan
|* Local function - only usable within same DLL script
function <domain> internal.helper.function()
{
    |* Local logic
    return(value)
}
```

### Complete DLL Implementation Example

#### DLL Script: `tsextdllvalidationUtils`
```baan
|******************************************************************************
|* DLL: tsextdllvalidationUtils
|* Purpose: Common validation functions for TSX modules
|* Package: ts (TSX)
|* Module: ext (Extensions)
|******************************************************************************

|* Global validation function for customer credit limits
extern function tcyesno tsext.dllvalidationUtils.validate.customer.credit(tccom.cust customer.id, tcamnt credit.amount)
{
    tcamnt current.balance
    tcamnt credit.limit
    
    |* Get customer balance and credit limit
    select tccom100.balc, tccom100.crdt
    from tccom100
    where tccom100.cust = customer.id
    selectdo
        current.balance = tccom100.balc
        credit.limit = tccom100.crdt
    selectempty
        |* Customer not found
        return(tcyesno.no)
    endselect
    
    |* Check if new amount would exceed limit
    if (current.balance + credit.amount) > credit.limit then
        return(tcyesno.no)
    else
        return(tcyesno.yes)
    endif
}

|* Global function for email validation
extern function tcyesno tsext.dllvalidationUtils.validate.email.format(string email.address())
{
    long at.position
    long dot.position
    
    |* Basic email format validation
    at.position = pos(email.address, "@")
    if at.position <= 1 then
        return(tcyesno.no)
    endif
    
    dot.position = pos(email.address, ".", at.position)
    if dot.position <= (at.position + 1) then
        return(tcyesno.no)
    endif
    
    if len(email.address) <= (dot.position + 1) then
        return(tcyesno.no)
    endif
    
    return(tcyesno.yes)
}

|* Global audit logging function
extern function long tsext.dllvalidationUtils.log.audit.event(string event.type(), string event.description(), string user.id())
{
    |* Log audit event to system audit table
    ttaud001.event.type = event.type
    ttaud001.description = event.description
    ttaud001.user.id = user.id
    ttaud001.timestamp = utc.num()
    ttaud001.session.id = session.id$
    
    insert ttaud001
    
    return(0)  |* Success
}

|* Local helper function - not accessible outside this DLL
function string format.timestamp(long timestamp.value) : string
{
    string formatted.date(20)
    
    formatted.date = date.to.string(timestamp.value, "dd/mm/yyyy hh:mm:ss")
    return(formatted.date)
}

|* Global utility function using local helper
extern function string tsext.dllvalidationUtils.get.formatted.audit.timestamp()
{
    return(format.timestamp(utc.num()))
}
```

### DLL Usage Patterns

#### Including DLL in UI Script
```baan
|******************************************************************************
|* UI Script: tccom1100m000 (Customer Maintenance)
|******************************************************************************

|* Include validation utilities DLL
#pragma used dll tsextdllvalidationUtils

field.tccom100.crdt:
when.field.changes:
    |* Validate credit limit using DLL function
    if not tsext.dllvalidationUtils.validate.customer.credit(tccom100.cust, tccom100.crdt) then
        message("Credit limit validation failed")
        tccom100.crdt = 0
        display("tccom100.crdt")
    else
        |* Log successful credit limit change
        tsext.dllvalidationUtils.log.audit.event("CREDIT_CHANGE", 
                                                 "Credit limit updated to " & str$(tccom100.crdt),
                                                 logname$)
    endif

field.tccom100.email:
check.input:
    |* Validate email format using DLL function
    if len(tccom100.email) > 0 then
        if not tsext.dllvalidationUtils.validate.email.format(tccom100.email) then
            message("Invalid email format")
        endif
    endif
```

#### Including DLL in DAL Script
```baan
|******************************************************************************
|* DAL Script: tccom100dal
|******************************************************************************

|* Include validation utilities DLL
#pragma used dll tsextdllvalidationUtils

function extern long before.save.object(long mode)
{
    |* Validate credit limit before saving
    if mode = DAL_NEW or mode = DAL_UPDATE then
        if not tsext.dllvalidationUtils.validate.customer.credit(tccom100.cust, tccom100.crdt) then
            dal.set.error.message("Credit limit validation failed")
            return(DALHOOKERROR)
        endif
        
        |* Log audit event
        tsext.dllvalidationUtils.log.audit.event("CUSTOMER_SAVE",
                                                 "Customer " & tccom100.cust & " saved",
                                                 logname$)
    endif
    
    return(0)
}
```

### Calculation Engine DLL Pattern

#### DLL Script: `tccomdllcalculationEngine`
```baan
|******************************************************************************
|* DLL: tccomdllcalculationEngine  
|* Purpose: Common calculation functions for financial operations
|******************************************************************************

|* Global function for tax calculations
extern function tcamnt tccom.dllcalculationEngine.calculate.sales.tax(tcamnt base.amount, string tax.code())
{
    tcperc tax.rate
    tcamnt tax.amount
    
    |* Get tax rate for code
    select ttcmg010.rate
    from ttcmg010
    where ttcmg010.code = tax.code
    selectdo
        tax.rate = ttcmg010.rate
    selectempty
        tax.rate = 0.0
    endselect
    
    tax.amount = base.amount * (tax.rate / 100.0)
    return(tax.amount)
}

|* Global function for discount calculations
extern function tcamnt tccom.dllcalculationEngine.calculate.discount(tcamnt base.amount, tcperc discount.percentage, tcamnt max.discount)
{
    tcamnt discount.amount
    
    discount.amount = base.amount * (discount.percentage / 100.0)
    
    |* Apply maximum discount limit
    if discount.amount > max.discount then
        discount.amount = max.discount
    endif
    
    return(discount.amount)
}

|* Global function for currency conversion
extern function tcamnt tccom.dllcalculationEngine.convert.currency(tcamnt amount, string from.currency(), string to.currency())
{
    tcrate exchange.rate
    tcamnt converted.amount
    
    if from.currency = to.currency then
        return(amount)
    endif
    
    |* Get exchange rate
    select ttcur101.rate
    from ttcur101
    where ttcur101.fcur = from.currency
    and ttcur101.tcur = to.currency
    and ttcur101.date <= date.num()
    order by ttcur101.date desc
    selectdo
        exchange.rate = ttcur101.rate
        break  |* Take most recent rate
    selectempty
        |* Default to 1:1 if rate not found
        exchange.rate = 1.0
    endselect
    
    converted.amount = amount * exchange.rate
    return(converted.amount)
}
```

### Integration Utility DLL Pattern

#### DLL Script: `tsextdllintegrationUtils`
```baan
|******************************************************************************
|* DLL: tsextdllintegrationUtils
|* Purpose: Common integration functions for external systems
|******************************************************************************

|* Global function for HTTP POST operations
extern function long tsext.dllintegrationUtils.http.post(string url(), string json.data(), ref string response())
{
    long result.code
    string headers(1024)
    
    |* Set standard headers
    headers = "Content-Type: application/json" & chr$(10) &
              "Accept: application/json" & chr$(10) &
              "User-Agent: BaanERP/Integration"
    
    |* Make HTTP call (pseudo-code - actual implementation depends on available HTTP libraries)
    result.code = http.send.request("POST", url, headers, json.data, response)
    
    return(result.code)
}

|* Global function for JSON formatting
extern function string tsext.dllintegrationUtils.format.json.customer(tccom.cust customer.id)
{
    string json.output(2048)
    
    |* Get customer data
    select tccom100.cust, tccom100.name, tccom100.city, tccom100.crdt
    from tccom100
    where tccom100.cust = customer.id
    selectdo
        json.output = "{" &
                      "\"customer_id\":\"" & tccom100.cust & "\"," &
                      "\"name\":\"" & tccom100.name & "\"," &
                      "\"city\":\"" & tccom100.city & "\"," &
                      "\"credit_limit\":" & str$(tccom100.crdt) &
                      "}"
    selectempty
        json.output = "{\"error\":\"Customer not found\"}"
    endselect
    
    return(json.output)
}

|* Global function for external system synchronization
extern function tcyesno tsext.dllintegrationUtils.sync.customer.to.crm(tccom.cust customer.id)
{
    string json.data(2048)
    string response(4096)
    long http.result
    
    |* Format customer data as JSON
    json.data = tsext.dllintegrationUtils.format.json.customer(customer.id)
    
    |* Send to external CRM system
    http.result = tsext.dllintegrationUtils.http.post("https://crm.company.com/api/customers", json.data, response)
    
    if http.result = 200 then
        |* Update sync status
        select tccom100.cust
        from tccom100 for update
        where tccom100.cust = customer.id
        selectdo
            tccom100.last.sync = utc.num()
            tccom100.sync.status = "SUCCESS"
            db.update(tccom100, DB.RETRY)
            if not db.retry then
                commit.transaction()
            endif
        selectempty
        endselect
        
        return(tcyesno.yes)
    else
        return(tcyesno.no)
    endif
}
```

### DLL Best Practices

#### Design Principles
1. **Single Responsibility**: Each DLL should focus on one functional area
2. **Global vs Local**: Use global functions for external access, local for internal helpers
3. **Naming Consistency**: Follow strict naming conventions for predictability
4. **Error Handling**: Implement comprehensive error handling in DLL functions
5. **Documentation**: Thoroughly document DLL purpose and function interfaces

#### Performance Considerations
1. **Function Granularity**: Balance between reusability and performance
2. **Parameter Efficiency**: Use appropriate parameter types (REF/CONST)
3. **Memory Management**: Clean up resources in complex functions
4. **Caching**: Consider caching expensive calculations within DLL

#### Maintenance Guidelines
1. **Version Control**: Maintain DLL versions carefully due to global impact
2. **Testing**: Thoroughly test DLL functions before deployment
3. **Dependencies**: Document DLL dependencies clearly
4. **Backward Compatibility**: Maintain compatibility when updating DLLs

### DLL Usage Patterns Summary

| DLL Type | Purpose | Example Usage |
|----------|---------|---------------|
| **Validation DLL** | Common validation rules | Field validation, business rule checking |
| **Calculation DLL** | Mathematical operations | Tax calculations, currency conversion |
| **Integration DLL** | External system interfaces | API calls, data synchronization |
| **Utility DLL** | General purpose functions | String manipulation, date formatting |
| **Security DLL** | Authentication/authorization | User permission checking, audit logging |

### DLL Integration Architecture

```baan
|* Example: Complete DLL integration in business process
#pragma used dll tsextdllvalidationUtils
#pragma used dll tccomdllcalculationEngine
#pragma used dll tsextdllintegrationUtils

field.tcord100.amount:
when.field.changes:
    |* Step 1: Validate using validation DLL
    if not tsext.dllvalidationUtils.validate.customer.credit(tcord100.cust, tcord100.amount) then
        message("Credit validation failed")
        return
    endif
    
    |* Step 2: Calculate tax using calculation DLL
    tcord100.tax.amount = tccom.dllcalculationEngine.calculate.sales.tax(tcord100.amount, tcord100.tax.code)
    
    |* Step 3: Calculate total
    tcord100.total = tcord100.amount + tcord100.tax.amount
    
    |* Step 4: Sync to external system using integration DLL
    if tcord100.sync.required = tcyesno.yes then
        tsext.dllintegrationUtils.sync.customer.to.crm(tcord100.cust)
    endif
    
    |* Update displays
    display("tcord100.tax.amount")
    display("tcord100.total")
```

This modular approach using DLLs enables:
- **Code Reuse**: Functions available across multiple scripts
- **Maintainability**: Centralized business logic
- **Testing**: Individual DLL testing
- **Performance**: Optimized shared functions
- **Consistency**: Standardized operations across the system 