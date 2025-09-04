## Effective AI Prompts

### Initial Review Request:
"Review [component] for performance, security, and maintainability issues.  
Focus on problematic code segments only. Output CSV format with verified line numbers.  
Check for patterns from previous-findings.md."

### Follow-up Reviews:
"Apply standards from review-standards.md. Verify all line numbers using grep search."

### Optimization Requests:
"Create optimized version with _cursor suffix addressing findings from Code_Review_Report.csv"

## 4GL-Specific AI Prompts

### Initial 4GL Code Review:
"Review [component] for Infor LN 4GL compatibility issues. Check against 4GL syntax limitations in review-standards.md. Focus on compilation-breaking constructs: struct declarations, mod operator, db.commit() calls in DAL scripts, goto statements, index() function, complex string functions. Output CSV format with verified line numbers."

### 4GL Code Implementation:
"Implement fixes for [component] using only 4GL-compatible syntax. Reference 4GL limitations in review-standards.md. Use conservative approaches: individual variables instead of structs, pos() instead of index(), integer division instead of mod operator, no db.commit() in DAL scripts, simple error handling without goto. Ensure all code compiles without errors."

### 4GL Error Resolution:
"Fix compilation errors in [component] by applying 4GL syntax limitations from review-standards.md. Address errors systematically: 1) Remove forbidden constructs 2) Fix database operations 3) Simplify function logic 4) Validate all changes against 4GL standards."

### 4GL Memory Management:
"Implement memory management for [component] using simple 4GL-compatible cleanup functions. Ensure: 1) All allocations have corresponding cleanup 2) Cleanup functions are simple without conditional logic 3) Cleanup calls at all program termination points 4) No complex memory operations."

### 4GL Validation Prompt:
"Before implementing any 4GL code changes, validate against these critical limitations: No struct, no mod operator, no db.commit() in DAL scripts, no goto, no index(), no complex string functions, proper SQL syntax, explicit function returns, simple memory cleanup. Confirm all suggestions are 4GL-compatible."
