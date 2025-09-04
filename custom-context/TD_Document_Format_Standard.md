# Technical Design (TD) Document Format Standard

## Overview
This document defines the exact format and structure for Technical Design documents that must be followed precisely to match company standards.

## Global Rules

### Heading Levels
- **H1** → Title
- **H2** → Major sections (1, 2)
- **H3** → Sub-sections (1.1, 1.2, etc.)
- **H4** → Sub-sub-sections (2.1.1, etc.)

### Table Requirements
- All tables use a fixed grid with header row bold
- No merged header cells
- Keep column order exactly as specified
- Grid with visible borders

### Component Section Naming
Select sub-section title based on component Type:
- **Table Definition** → "Table Design"
- **Script/Library source** → "Programming Logic"
- **DLL/DAL** → "Change"
- **Integrated Session** → "Change"

### Content Requirements
- Insert a 3-level Table of Contents under "Contents"
- Use metadata table exactly with fields: Name, Type, New/Existing, VRC for changes, Retrofit VRC's
- Keep repeated phrasing verbatim where applicable
- Preserve duplicates if source lists the same component more than once

## Document Structure Template

```
Technical Design

<Sub-title>

Version <X.Y>
<DD Month YYYY>

Module: <Module>
Title: <Title>
Issue ID: <ID>
Prepared by: <Name>

| Approver Name | Title | Signature | Date |
|---------------|--------|-----------|------|
| <blank row>   |        |           |      |

Contents
(Insert 3-level TOC)

1 Introduction

1.1 Scope
| Component Name | Component Type | Complexity | VRC's |
|----------------|----------------|------------|-------|
| <row per component> |

1.2 Definitions, acronyms, abbreviations
| Term | Description |
|------|-------------|
| <rows> |

1.3 FD-TD Linkage
| FD Section | TD Section |
|------------|------------|
| <rows> |

1.4 References Documents
| Document | Section | Sub-section (if any) |
|----------|---------|---------------------|
| <rows> |

1.5 Technical Realization Plan:
| TD | Module | Owner | TR Sequence |
|----|--------|-------|-------------|
| <rows> |

2 Technical Design

# Repeat the following block for each component in exact order

2.<n> Component details

Name <value>
Type <Table Definition | Script/Library source | DLL | DAL | Integrated Session>
New/Existing <New | Existing>
VRC for changes <value>
Retrofit VRC's <value or ->

2.<n>.1 <Table Design | Programming Logic | Change>

## If Table Design
Table: <code>
<description>
Change: <e.g., New fields>

| Field | Domain | Description | Initial Value | Mandatory | Reference |
|-------|--------|-------------|---------------|-----------|-----------|
| <rows> |

Table: <value>
Reference Mode: <value>
Check by DBMS: <value>
Delete Mode: <value>

Index
<index list>

## If Programming Logic / Change
1. <logic item 1>
2. <logic item 2>
...
```

## Component Metadata Table Format

Use a two-column key/value table (5 rows):

| Name | |
|------|---|
| Type | |
| New/Existing | |
| VRC for changes | |
| Retrofit VRC's | <value or -> |

## Programming Logic Format
- Use numbered lists
- Keep recurring phrasing exactly when provided by source
- Example standard phrasing: "Logic added to handle Consignment sales order/ Consignment Return Order Functionality."

## Table Design Controls
Render the four control lines as separate paragraphs immediately after the fields grid:
- Table: <value>
- Reference Mode: <value>
- Check by DBMS: <value>
- Delete Mode: <value>

Then an Index label with entries or left blank.

## Fonts and Styling (Word Output)
When rendering to Word, apply corporate TD template styles:
- **Title**: Heading 1 (centered)
- **Sub-title**: centered
- **Body**: Normal paragraph
- **Tables**: grid with visible borders; header row bold

## Validation Checklist
Before finalizing any TD document:
- [ ] Exact section names and numbering used (no invented sections)
- [ ] Component subsection titles match Type mapping rules
- [ ] All required metadata fields included
- [ ] Table schemas follow exact column orders
- [ ] Fixed grid formatting applied to all tables
- [ ] 3-level TOC included
- [ ] Corporate styling guidelines followed
- [ ] Recurring phrasing preserved exactly
- [ ] Empty placeholder rows maintained where specified

## Notes
- Keep empty placeholder rows where shown in templates
- Preserve duplicates if source lists same component multiple times
- Do not invent new section names
- Maintain exact column order in all tables
- If template file provided, inherit its styles rather than redefining
