## Established Workflow

1. **Complete Script Coverage**: Check all directories systematically
2. **Line Number Verification**: Use grep search, no estimates
3. **Categorization**: Performance > Security > Maintainability > Documentation
4. **Output Format**: CSV with Major (True/False) flag
5. **Focus**: Problematic segments only, exclude positive findings unless requested

## 4GL-Specific Workflow Extensions

### Before Code Implementation:
6. **4GL Syntax Validation**: Check all proposed solutions against 4GL limitations in review-standards.md
7. **Memory Management Review**: Ensure proper cleanup functions are planned
8. **Database Operations Check**: Validate all SQL and database calls for 4GL compatibility (note: db.commit() forbidden only in DAL scripts)

### During Code Implementation:
9. **Conservative Approach**: Use proven 4GL patterns, avoid modern language constructs
10. **Incremental Testing**: Implement small sections to validate syntax before full implementation
11. **Documentation Cleanup**: Separate revision history from active code

### After Code Implementation:
12. **Compilation Validation**: Always verify code compiles without errors
13. **Memory Cleanup Verification**: Ensure all allocated memory has corresponding cleanup
14. **Function Parameter Audit**: Confirm functions have reasonable parameter counts (<10)

### 4GL Error Resolution Process:
- If compilation errors occur, reference 4GL syntax limitations
- Fix errors systematically by category (syntax -> database -> functions)
- Validate each fix against 4GL standards before proceeding
- Document any new 4GL limitations discovered
