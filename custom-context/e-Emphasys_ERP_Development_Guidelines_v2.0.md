# e-Emphasys ERP Development Guidelines v2.0

## 1. Introduction

This document outlines the standards, structure, and practices that must be followed for ERP custom development and maintenance across e-Emphasys Baan/Infor ERP applications. It ensures consistency, maintainability, and code quality across development teams.

---

## 2. Baan Standards

### 2.1 Naming Conventions

- **Package Prefixes**: All custom developments must follow a unique prefix, such as `tsext`, to separate them from standard packages.
- **DLLs**: Named as `<package><module>dll<name>`, e.g., `tsextdllglobalFunctionsDLL`.
- **Functions**:
  - Internal: Use simple names like `calculate.total()`.
  - DLL Exported: Must follow full naming convention: `extern function domain <dll>.<function>()`

### 2.2 Script Types

- **UI Script**: Attached to sessions; handles screen logic.
- **DAL Script**: Table-based logic; includes object and property hooks.
- **General Library (DLL)**: Shared global logic.
- **Report / Job Scripts**: Background batch logic or printed outputs.
- **Menu/Control Scripts**: Drive control over session menus.

---

## 3. Coding Guidelines

### 3.1 Script Header

Each script must include a header block at the top with:
- Author
- Date
- Purpose
- Change history

```baan
|* Author      : Anish
|* Date        : 17-Jul-2025
|* Purpose     : Validate sales order total before posting
|* Change History:
|* Rev  Date        Author      Description
|* 1    17-Jul-2025 Anish       Initial version
|* 2    20-Jul-2025 Anish       Added validation for negative amounts
```

### 3.2 Code Structure Requirements

#### 3.2.1 Declaration Section
- All domain declarations must be properly organized
- External domains should be clearly separated from internal ones
- Table declarations should follow standard naming conventions

#### 3.2.2 Function Organization
- Functions should have single responsibility
- Complex functions should be broken down into smaller units
- Proper error handling must be implemented

#### 3.2.3 Database Operations
- Use DAL patterns where applicable for data integrity
- Implement proper transaction boundaries with db.retry.point()
- Always include commit.transaction() or abort.transaction()

### 3.3 Documentation Standards

#### 3.3.1 Function Documentation
Each function must include:
```baan
|******************************************************************************
|* Function: function.name
|* Purpose: Brief description of function purpose
|* Parameters: 
|*   param1 - Description of parameter 1
|*   param2 - Description of parameter 2
|* Returns: Description of return value
|* Author: Developer Name
|* Date: Creation Date
|******************************************************************************
```

#### 3.3.2 Inline Comments
- Complex business logic must be commented
- Database operations should explain the business purpose
- Any workarounds or special handling should be documented

### 3.4 Error Handling Standards

#### 3.4.1 Consistent Error Patterns
- All functions should return meaningful error codes
- Error messages should be user-friendly and informative
- System errors should be logged for debugging

#### 3.4.2 Transaction Safety
- All database operations must be wrapped in proper transaction boundaries
- Rollback procedures must be implemented for error conditions
- Concurrency handling with retry patterns is mandatory

---

## 4. Performance Guidelines

### 4.1 Database Optimization
- Use appropriate indexes in SELECT statements
- Avoid unnecessary table joins
- Implement efficient WHERE clauses
- Use db.retry.point() for concurrency control

### 4.2 Memory Management
- Minimize variable scope
- Clean up large data structures when no longer needed
- Avoid excessive domain declarations

### 4.3 Query Optimization
- Use selective WHERE clauses
- Implement proper ORDER BY usage
- Avoid SELECT * statements
- Use aggregate functions efficiently

---

## 5. Security Standards

### 5.1 Data Protection
- Sensitive data (like credit card numbers) must be masked
- Implement proper access controls
- Log security-related operations for audit

### 5.2 Input Validation
- All user inputs must be validated
- Implement boundary checks for numeric inputs
- Sanitize string inputs to prevent injection

---

## 6. Version Control Standards

### 6.1 Change Management
- All changes must be documented in script headers
- Version numbers should follow semantic versioning
- Change descriptions must be clear and specific

### 6.2 Code Reviews
- All custom developments require peer review
- Critical changes require senior developer approval
- Performance impact must be assessed for major changes

---

## 7. Testing Requirements

### 7.1 Unit Testing
- All functions should be unit tested
- Edge cases and error conditions must be tested
- Test data should cover all business scenarios

### 7.2 Integration Testing
- Multi-module interactions must be tested
- Database integrity should be verified
- Performance testing for high-volume operations

---

## 8. Deployment Standards

### 8.1 Environment Promotion
- Changes must be tested in development environment first
- Quality assurance sign-off required before production
- Rollback procedures must be documented and tested

### 8.2 Documentation Requirements
- Technical documentation for complex implementations
- User documentation for new features
- Operations documentation for system administrators

---

## 9. Compliance and Quality Assurance

### 9.1 Code Quality Metrics
- Cyclomatic complexity should be minimized
- Code duplication should be avoided
- Maintainability index should be tracked

### 9.2 Standards Compliance
- Regular audits of code against these guidelines
- Automated tools should be used where possible
- Training programs for development teams

---

## 10. Best Practices Summary

### 10.1 Development Best Practices
1. Follow the DRY principle (Don't Repeat Yourself)
2. Implement proper separation of concerns
3. Use meaningful variable and function names
4. Write self-documenting code
5. Implement proper error handling
6. Use consistent coding styles
7. Optimize for performance and maintainability

### 10.2 Team Collaboration
1. Regular code reviews
2. Knowledge sharing sessions
3. Documentation updates
4. Cross-training on modules
5. Continuous improvement processes

---

## 11. Tools and Resources

### 11.1 Development Tools
- BAAN/Infor Development Environment
- Version control systems
- Code review tools
- Testing frameworks

### 11.2 Reference Materials
- BAAN 4GL Language Reference
- DAL Architecture Documentation
- Performance Optimization Guides
- Security Best Practices

---

## Appendix A: Code Examples

### A.1 Standard Function Template
```baan
|******************************************************************************
|* Function: calculate.order.total
|* Purpose: Calculate total amount for sales order including tax
|* Parameters: 
|*   order.number - Sales order number to calculate
|* Returns: tcamnt - Total order amount including tax
|* Author: Development Team
|* Date: Current Date
|******************************************************************************
function domain tcamnt calculate.order.total(domain tcorno order.number)
{
    domain tcamnt total.amount
    domain tcamnt tax.amount
    
    db.retry.point()
    
    select  sum(tdsls401.pric * tdsls401.dqua):total.amount,
            sum(tdsls401.iagt):tax.amount
    from    tdsls401
    where   tdsls401._index1 = {:order.number}
    selectdo
        return(total.amount + tax.amount)
    selectempty
        return(0.0)
    endselect
}
```

### A.2 Error Handling Example
```baan
function domain tcbool process.with.error.handling()
{
    db.retry.point()
    
    select  table.field
    from    table
    where   table.key = {:key.value}
    selectdo
        |* Process the record
        commit.transaction()
        return(true)
    selectempty
        error.message = "Record not found"
        return(false)
    endselect
}
```

---

**Document Version**: 2.0
**Last Updated**: Current Date
**Next Review**: Annual Review Required
**Approved By**: Development Management Team 