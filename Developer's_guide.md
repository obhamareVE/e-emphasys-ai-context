# Developer's Guide (Context Reference)

Author: GitHub Copilot
Last Updated: 2025-09-04
Scope: Internal context file summarizing enforced IDENT tagging standards for code generation & modification.

---
## 1. IDENT Standard (Authoritative Summary)
IDENT Token Format:
- Pattern: `R<yyyy.mm>.<Jira-ID>` optionally followed by a local sequence suffix (`-<index>`) for multi-snippet cases within the same Jira.
- Examples:
  - Single snippet: `R2025.05.EDM-7129`
  - Multiple related snippets (same Jira, same month): `R2025.06.EDM-19104-1` (first block), `R2025.06.EDM-19104-2` (second block)

### 1.1 Atomic Line Annotation ("New Line")
- Rule: Append `|IDENT.n` to the **end** of a single modified/added line.
- Example:
  ```
  not is.other.line.prsnt.for.same.spcd() then   |R2025.05.EDM-18653.n
  ```

### 1.2 Multi-Line Snippet Annotation
- Place a start marker line **before** the changed block: `|IDENT.sn`
- Place an end marker line **after** the block: `|IDENT.en`
- Do NOT add `.n` markers inside a `.sn`/`.en` fenced block.
- Example:
  ```
  |R2025.06.EDM-19104-1.sn
  else
      if temp.record.count = 1 and
         not isspace(tdext851.memo) and
         not isspace(tdext851.clot) then
          tdextdll0032.update.equipment.memo.to.address(
                          tdext851.memo,
                          tdext851.clot,
                          tdext851.stad)
      endif
  |R2025.06.EDM-19104-1.en
  ```

### 1.3 Commenting Existing Code
- Single existing line commented out: append `|IDENT.o` at end of the (now commented) line.
- Multi-line commented region: wrap with `|IDENT.so` and `|IDENT.eo` (start / end original-comment block).
- Inside `.so`/`.eo` fenced region: do **not** add `.o` per line.

### 1.4 Overlap & Uniqueness Constraints
- No nested IDENT regions.
- No duplicated IDENT markers for same lines.
- Inside a multi-line snippet (`.sn` ... `.en`) you **must not** append additional `.n` to interior lines.
- Each snippet identifier (including suffix like `-1`, `-2`) must be unique within the file.

### 1.5 When to Use Suffixes (-1, -2, ...)
Use `-<seq>` only when the **same Jira** requires multiple separated multi-line blocks in the **same file**. Single isolated block: omit suffix unless future blocks are anticipated (prefer explicit `-1` if sequence certainty adds clarity for later tooling).

---
## 2. Decision Flow (Apply Correct Marker)
1. Are you adding/modifying exactly ONE line (and not inside an active snippet)?
   - Yes → append `|IDENT.n`.
2. Are you adding/modifying a contiguous block of >1 lines? → Wrap with `|IDENT.sn` / `|IDENT.en`.
3. Are you commenting OUT exactly one existing line (not already in a snippet)? → Append `|IDENT.o`.
4. Are you commenting OUT a contiguous block of existing lines? → Wrap with `|IDENT.so` / `|IDENT.eo`.
5. Are you changing content INSIDE an existing `.sn`/`.en` block? → No new marker unless adding new lines that logically remain part of same snippet (still no `.n`).
6. Never mix `.n` with `.sn` or `.so`/`.eo` contexts.

---
## 3. Quick Reference Table
| Intent | Scope | Start Marker | Per-Line Marker | End Marker | Example Suffix Use |
|--------|-------|--------------|-----------------|------------|--------------------|
| Add/modify one line | 1 line | — | `.n` | — | `|R2025.05.EDM-18653.n` |
| Add/modify block | >1 lines | `.sn` | none | `.en` | `|R2025.06.EDM-19104-1.sn` |
| Comment out one line (original) | 1 line | — | `.o` | — | `|R2025.05.EDM-20000.o` |
| Comment out block (original) | >1 lines | `.so` | none | `.eo` | `|R2025.07.EDM-21011.so` |

---
## 4. Validation Checklist (Pre-Commit)
- [ ] All identifiers match `RYYYY.MM.EDM-<digits>(-<seq>)?`.
- [ ] Month (`MM`) is zero-padded & matches release cycle.
- [ ] No `.n` lines exist inside any `.sn`/`.en` or `.so`/`.eo` fenced region.
- [ ] No nested snippet or original-comment regions.
- [ ] Suffix numbering (`-1`, `-2`) increments without gaps inside the same file for the same Jira (recommended, though gaps are tolerated if intentional—document in commit message).
- [ ] Removed or replaced code remains traceable (commented original vs new snippet separated clearly).

---
## 5. Examples (Good vs Bad)
### 5.1 Good Single-Line Addition
```
new.logic.flag = true    |R2025.08.EDM-22501.n
```
### 5.2 Bad (Improper Single-Line)
```
new.logic.flag = true    |R25.8.EDM22501.n   # Wrong date format & Jira pattern
```
### 5.3 Good Multi-Line
```
|R2025.08.EDM-22502.sn
if not isspace(code) then
    normalize(code)
endif
|R2025.08.EDM-22502.en
```
### 5.4 Bad (Interior `.n` inside snippet)
```
|R2025.08.EDM-22502.sn
if not isspace(code) then   |R2025.08.EDM-22502.n   # INVALID extra marker
    normalize(code)
endif
|R2025.08.EDM-22502.en
```

### 5.5 Good Original Code Block Comment
```
|R2025.08.EDM-22503.so
old.proc.call()
legacy.cleanup()
|R2025.08.EDM-22503.eo
```

---
## 6. Tooling & Automation Hints
Potential future linters / CI hooks can enforce:
- Regex scan ensuring markers only appear in allowed syntactic positions.
- Balance checks: every `.sn` has matching `.en`; every `.so` has matching `.eo`.
- No overlap by tracking line index intervals.
- Chronological audit: ensure `yyyy.mm` not in the future relative to build date (optional policy).

---
## 7. FAQ
Q: Can I mix multiple Jira IDs in the same snippet?  
A: No. One snippet → one Jira ID.

Q: If I add a new line inside an existing snippet later, do I give it `.n`?  
A: No—remain inside the original fenced region without extra markers.

Q: What if I must partially modify lines inside a `.so`/`.eo` region?  
A: You should not—those lines represent preserved original code. Add new active logic outside the original-comment fence using a new snippet or single-line markers.

Q: Do I ever nest `.so` inside `.sn`?  
A: No. If you need to show original vs new: comment original first (`.so`/`.eo`), then add new logic with separate `.sn` or `.n` markers.

---
## 8. Implementation Conventions
- Maintain consistent spacing: marker lines (`|IDENT.sn`, etc.) should be isolated on their own line with no leading/trailing spaces besides indentation if required by language (prefer column 0 unless style dictates otherwise).
- For line-end markers (`.n`, `.o`), ensure at least two spaces before the pipe for readability where style allows.
- Do not alter historical IDENT markers—treat them as immutable audit artifacts.

---
## 9. Commit Message Template
```
EDM-<Jira#>: <Concise Summary>

Changes:
- Added <feature/validation> (R2025.MM.EDM-<Jira#>(-<seq>))
- Commented legacy block (R2025.MM.EDM-<Jira#>)

Validation:
- IDENT markers validated (no overlap, patterns pass)
```

---
## 10. Enforcement Ready Regex Snippets
(Adjust per language tooling)
- IDENT capture: `R20[0-9]{2}\.(0[1-9]|1[0-2])\.EDM-[0-9]+(?:-[0-9]+)?`
- Start snippet: `^\|R.*\.sn$`
- End snippet: `^\|R.*\.en$`
- Start original: `^\|R.*\.so$`
- End original: `^\|R.*\.eo$`
- Single line new: `\|R.*\.n$`
- Single line original: `\|R.*\.o$`

---
## 11. Summary (One-Liner)
Always tag every new or commented code change using the strict IDENT grammar: single lines with `.n` / `.o`, blocks with balanced `.sn`/`.en` or `.so`/`.eo`, never nesting or duplicating markers, and always using canonical `RYYYY.MM.EDM-<Jira>(-<seq>)` identifiers.

---
End of Developer's Guide.
