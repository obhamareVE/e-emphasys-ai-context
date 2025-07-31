# e-Emphasys ERP

# Development Guidelines

```
Document Version 2. 1
```
```
Release Date: February, 2019
```

```
Document Version History
```
No. Ver No. Change Description Page / Section No. ReleaseDate Changed by

```
1 1. 0 Base Version All 2005 Viral Adeshra
```
```
2 2.0 Update to Base Version All 2018 Viral AdeshraRajesh Mirani,
```
#### 3 2.

```
Naming convention for
Variant reports
introduced. Module code
for Planning has been
changed to L.
```
```
Page 26 12 Feb 2019 Viral Adeshra
```

© Copyright e-Emphasys ERP Development. All rights reserved by e-Emphasys Technologies Inc.

The information in this document is subject to change without prior notice. No part of this document may
be reproduced, translated, stored or transmitted in any form or by any means, electronic or mechanical,
for any purpose, without the express written permission from e-Emphasys Technologies Inc.

The material contained in this document constitutes and contains confidential & proprietary information of
e-Emphasys. Access to this document does not pass on rights of any material, title or any other interest
included in this document.

By accessing this document, the reader acknowledges that confidentiality will be maintained, and this
document will be used only for the purpose for which it is intended. Despite due care taken by
e-Emphasys Development center, the document may contain some typographical mistakes or other
errors. The information herein may not be complete, or may not meet the readers’ specific requirements.

e-Emphasys Development center assumes no liability for any damages incurred, directly or indirectly,
from any errors, omissions or discrepancies between the software and the information contained in this
document.

e-Emphasys Technologies Inc.

2501 Weston Parkway, Suite 101, Cary, NC 27513,
USA

Phone: + 1 919 657 6565
Fax: + 1 919 657 0773
e-mail: info@e-emphasys.com
Website: [http://www.e-emphasys.com](http://www.e-emphasys.com)


## Table of Contents


- 1. About this Document
   - 1.1 Document Conventions
- 2. Software Programming Standards
   - 2.1 Introduction
   - 2.2 Importance of Technical Documentation.....................................................................................
   - 2.3 Standards for Scripts..................................................................................................................
   - 2.4 Script Layout
   - 2.5 Naming Conventions of Variables, Functions
   - 2.6 Marking Changes in Scripts
- 3. Software Coding Standards...........................................................................................................
   - 3.1 Packages
   - 3.2 Modules
   - 3.3 Session....................................................................................................................................
   - 3.4 Correction Program Sessions...................................................................................................
   - 3.5 Table
   - 3.6 Table Fields
   - 3.7 Domains
   - 3.8 Messages
   - 3.9 Labels
   - 3.10 Questions
   - 3.11 Scripts
   - 3.12 Libraries...................................................................................................................................
   - 3.13 Data Access Layer
   - 3.14 SSRS Naming Conventions and Standards
   - 3.15 Naming Conventions for Variant Reports..................................................................................
- 4. General Guidelines
   - 4.1 Guidelines for Creating PMC Solution
   - 4.2 Using the Revision Text Template
   - 4.3 Using PMC Details Template
   - 4.4 Standard Post-Installation Write-up
   - 4.5 Customization Administration
   - 4.6 Guidelines for Correction Programs..........................................................................................
   - 4.7 Rules Concerning Modification of Large Tables
   - 4.8 Reading Parameter Tables
   - 4.9 Usage of DAL2


## 1. About this Document

```
This document describes the Software Engineering Guidelines for e-Emphasys ERP
Development. Within these guidelines, we have tried to collect the principles of our profession.
These principles represent the state-of-the art of what we believe is ‘right’ when building
software.
```
```
Principles are the rules to live by; they represent the collected wisdom of many people who have
learned through experience.
```
### 1.1 Document Conventions

```
This document uses following conventions:
```
```
Convention Description
```
```
This icon indicates additional notes / information about a field, feature or
functionality.
```
```
This icon indicates critical information about a field, feature or functionality.
```
```
<bold text> All button names, field labels and critical information is formatted as bold.
```
```
Images All the images in the document are numbered and have a caption below
them.
```
```
Links and
References Appropriate links and references are provided to related sections.^
```

## 2. Software Programming Standards

### 2.1 Introduction

```
Software Programming Standards help the programmer to:
```
- Share good programming practices
- Provide standard programming solutions and style
- Manage complexity
- Write code that is easier to understand (especially for other developers)
- Provide a reference for reviews and inspections
- Improve documentation (in Microsoft source code, for example, 40% is documentation)

### 2.2 Importance of Technical Documentation.....................................................................................

```
As technical documentation is read by several software/support engineers throughout the
software lifecycle, it is important that the readability of our code greatly improves. A lot of effort is
wasted due to poor documentation.
```
2.2.1 General Standards of Technical Documentation

- Use of US English - Technical documentation should be in correct US English. Although
    most of the existing source code and mnemonics still reflect the former British English terms,
    the descriptive texts should reflect the current US terminology standards. For example, the
    mnemonic 'upd.item.stock' should be documented as ' | updates the inventory data for this
    item'.
- Correctness - Documentation should be without errors. It should be accurate and use
    correct grammar, style, and spelling.
- Completeness **–** Documentation should describe the whole complexity, not just a part of it. If
    there is incomplete documentation, then usually it is distrusted.
- Placement - Documentation is only helpful if it is placed where it is expected. The
    documentation must describe the program flow at the main level. For example, above the
    function nestings. Complex statements must be described at the lowest level, that is, right
    below the statement. For example, Function explanations should not be in the middle of the
    function, but in the header.
- Compliant with standards – Documentation should be compliant with standards for easier,
    faster and better understanding.


### 2.3 Standards for Scripts..................................................................................................................

```
This section explains the standards for various elements of scripts.
```
```
2.3.1 Script Header
```
```
Following are the standards for Script Header:
```
```
Standard Details Mandatory?
General The history of the script:
```
- The script’s name
- The script’s author
- The date of creation
- The associated version

```
Mandatory
```
```
Description Brief description of the program.
```
```
Information that covers the entire source and is of
great importance for understanding. For example, a
design decision, deviations from standard solutions.
```
```
Mandatory
```
```
Optional
```
Modifications (^) • The functional changes added by a new version

- Bug-fixes specified by relevant data

```
Mandatory
```
```
Program Flow A summary of the program flow called from other
session:
```
- Sequence of updating tables
- Calling other sessions, DLLs
- Transaction management

```
Mandatory for
complex
update
programs
```
```
Example:
```
|************************************************************************
|* xiext0160 0 VRC E40M 1 ex
|* Field Campaign Details
|* Vinayak Shetty
|* 01- 05 - 15 [10:53]
|************************************************************************
|* Description
|* This Program displays data of field campaign headers and allows to
|* allows to process records in e-Emphasys ERP.
|************************************************************************
|* ID: <the unique ID number>
|* Name: <author’s name>
|* Date: <the date on which the change was made>
|* Desc: <a description of what has changed / fixed>
|************************************************************************
|* ID: <the unique ID number>
|* Name: <author’s name>
|* Date: <the date on which the change was made>
|* Desc: <a description of what has changed / fixed>
|************************************************************************

```
HEADER
```
```
DESC
```
```
MODIFICATIONS / BUG
```
-

```
FIXES
```

2.3.2 Script

```
Following are the standards for Script:
```
```
Standard Details Mandatory?
Declarations Documentation of:^
```
- Variables
- Defines

```
Mandatory, if no
expressive names
```
```
Data Dictionary
Components:
```
```
Description of
```
- Tables
- Includes
- Messages
- Questions
- User options

```
Mandatory, can be done
using “add comment to
scripts”
```
```
Update Sections A brief description of the updates or checks Mandatory for complex
maintenance programs
```
```
Queries Explanation of the expected output Recommended for
complex queries:
```
- With many where-
    lines
- Order/group by
- Where exist
- Dynamic SQL

Declarations:

declaration:

table twhwmd200 |* Warehouses
table twhwmd215 |* Item Inventory by Warehouse

long nr.items |* number of items/warehouse

domain tcccur currency |* local/home currency

#include “itccom0000” |* Multicurrency handling
#include “itcmcs2000” |* DAL support


Data Dictionary Components:

field.tfgld410.amnt:
check.input:
if tfgld410.amnt > budget.amount then
set.input.error(“tfgld0010”, budget.amount)
|* Amount not allowed, exceeds budget amount %f
endif

if not item.exist.in.itm001(item) then
mess(“tiitms0001”, 1,item)
|* Item %s does not exist
endif
if ask.enum(“tssma0010”, empty) = tcyesno.yes then
|* Copy service order text?
copy.service.order.text()
endif

Queries:

select tibom010.*
from tibom
where tibom010._index1 = {:bom.main.item}
and (tibom010.indt <= :effective.date)
and (tibom010.exdt > :effective.date or
tibom010.exdt = 0)
and not exists (
select a_tibom010._index
from tibom010 a_tibom
where a_tibom010._index1 = {tibom010.mitm, tibom010.pono}
and ((a_tibom010.indt <= :effective.date) and
(a_tibom010.exdt > :effective.date or
a_tibom010.exdt = 0))
and a_tibom010.seqn < tibom010.seqn)
order by tibom010._index
selectdo
|**********************************************************************
|* This query selects from the bom.main.item in the Bill Of
|* Material (BOM)the components (tibom010.sitm) that are
|* valid (= meeting the effective date).
|* However for a component one or more BOM lines may exist
|* that are all valid, so exist within the introduction
|* date (tibom010.indt) and expiry date (tibom010.exdt,
|* 0 means "never expires). Therefore the first
|* valid line will be selected.
|* The query is optimized for performance.
|*********************************************************************
...
endselect


2.3.3 Functions

```
Following are the standards for Functions:
```
```
Standard Details Mandatory?
```
```
External function Structure:
DllUsage
Expl. : summary of function
Pre : pre.conditions
Post : post.conditions
Input : input variables
Output : output variables
Return : return values
EndDllUsage
```
```
Mandatory in all external
functions like DLL, DAL
Business Method, form
command functions.
```
```
Documentation must provide
enough information to
understand the function without
investigating the function logic
itself.
```
```
Internal function Structure:
FunctionUsage
Expl.: summary of function
Pre : pre.conditions
Post : post.conditions
Input : input variables
Output : output variables
Return : return values
EndFunctionUsage
```
```
Mandatory, although this does
not apply to simple, expressive
functions like
read.item.description
```
```
Example:
```
function extern long whext.dll0001.process.planned.stock.transaction(
domain tckoor i.transaction.origin,
domain tcpdno i.order.number)
{
DllUsage
Expl: The purpose of this function is to:
Create/Update/Remove “planned stock transaction” in Warehousing

Pre: The sum of all the quantities up till now by this function
should be >= 0.

Post: Planned stock transactions are created/updated/removed
And the corresponding financial transactions are
generated.
If sum of all quantities are:
0: transactions are removed
If i.quantity=0, and i.date=0, no action is performed.
Input :
i.transaction.origin - Order Type, generally tckoor.act.sfc
i.order.number - (Production) Order Number
Output: -
Return: 0/DALHOOKERROR, when precondition not met: DALHOOKERROR
EndDllUsage
}


### 2.4 Script Layout

```
This section explains the standards for script layouts.
```
2.4.1 Standards for Script Layout

```
Tabs have a standard length of eight positions, the line size is maximum 80 positions because:
```
- The maximum capacity of most editors is 80, which is derived from the 80 columns card
- To be able to edit/display 2 sources simultaneously on screen
- To avoid deep nesting
Do not append extra spaces at the end of a line to fill up the 80 positions.
All labels (section names, subsection names, and function names) start in the first column. All
other statements are at least one tab from the first column.

```
The statements, which are conditionally executed, are always one tab indented compared with
the condition. The begin/end statements are aligned vertically.
```
```
Example,
```
function function.name()
FunctionUsage
Expl:
<rest FunctionUsage left out>
EndFunctionUsage
{
if something then
on case something.else
case 1:
do.something()
break
case 2:
do.something.else()
break
default:
do.nothing()
break
endcase

while not anything
look.for.anything()
endwhile
endif
}

```
Please note that the “then” is always at the end of the “if” statement, and not on a new line.
In case of assignments longer than one line, the second line will start below the first expression
after the equal sign (=):
```
```
Example,
```
invoice.amnt = (payment.amount + payment.difference) *
invoice.rate / invoice.factor


```
For declarations, the following sequence is mandatory:
```
table tkkmmmsvv
long
extern long
double
extern double
string
extern string
domain
extern domain

```
Operators are always preceded and succeeded by spaces.
```
### Example,

amount = amount + 1

```
Only lowercase letters are used in the script. This does not apply to #DEFINES for which capitals
are used.
```
```
Do not put multiple statements on one line.
```
### Example,

long nr.warehouses, nr.departments

```
The following logical program block section must be separated by blank lines:
```
- Functions and sections
- A declaration section between type table, extern, arrays, defines
- Iteration and selection statements like if then else-, case-, for constructions (empty line for
    nesting not needed)
- SQL queries
Curly brackets { or } are on a new line and the rest of the line is empty.
Each statement should begin on a new line.

```
A function header should look like this:
```
function count.and.fill.arrays.with.inv.quantities(
const domain tcitem i.item,
long i.number,
ref domain tdabd.norm o.actual,
ref domain tcmcs.str12 o.format)
{
static domain tcitem temp.item
static domain tccprj temp.project
}


```
At the function call, the arguments will be placed under the function name at the second tab.
Each argument must be on a separate line. If the function call is included in an if-then-else
statement, an extra indent is required to prevent the arguments to have the same indent as the
code in the if-then-else statement itself.
```
fill.stock.from.array(
form.cuni,
actual.occ)

if default.price.read(
g.bpid,
g.item) then
do.something()
endif

```
When using indices in queries, each part in the index must be coded below each other on a
separate row:
```
select erext401.*
from erext
where erext401._index1 = {:i.orno,
:i.pono,
:i.seqn}
selectdo
do.something()
endselect

### 2.5 Naming Conventions of Variables, Functions

```
This section explains the naming conventions of Variable and Functions.
```
2.5.1 Rules Related to Variable Names

```
Separate parts of the name by dots.
In the old version, often the table field mnemonics are used instead of expressive names (for
example, current.cwar derived from whwmd200.cwar). In e-Emphasys ERP, expressive names
are mandatory. Logical names consist of one or more nouns.
It is not allowed to use temporary names (such as temp, tmp5, and so on).
It is not allowed to use variable types (domain) in names (for example, string.item)
To distinguish the different usages of variables, the following coding standards are
recommended:
```
```
Coding Variable is
```
```
g.xxxxxxx global (as opposite to local)
<scriptcode>.xxxxxx extern (and implicitly global)
b.xxxxxx Based
frm.<variable name> All form fields to be prefixed
with a “.frm”
rep.<variable name> All Report Input
fields/variables to be prefixed
with a “.rep”.
```

```
<scriptcode> as prefix for external variables means that the complete DAL/DLL or
Include code must be placed before the actual variable name, so it is always clear
what the origin of the variable is. This is, however, not always needed, for
instance in case of report/form variables.
```
```
Function arguments should be preceded by:
```
```
Coding Argument is Variable passing
```
```
i.xxxxxxx input only local validity in the function
o.xxxxxx output call by reference
io.xxxxxx input and output call by reference
```
```
It is forbidden to assign values to input variables within a function like i.item = o.cust.item. As
alternative one can introduce a new local variable to which the input values is assigned. The
reason for this rule is that an input should stay with the same value as filled upon function call,
otherwise confusion is introduced if it is also meant as an output. In this case, the statement was
the first statement in the function so even doubts were introduced if the o.cust.item was also
input!
```
```
The only exceptions are the standard functions in the DAL for the following variables:
has_changed, mode and method.
```
2.5.2 Rules Related to Functions

```
Following are ruled related to Functions:
```
1. Only use expressive names. The name should tell as much as possible about the meaning,
    and context.
2. Use dots to separate parts of the name. Do not use underscores for this purpose.
3. If functions are defined in a library, the code of that library name must be incorporated in the
    external function name.
4. Return values or output arguments must relate to the logical name. For example, if a logical
    name suggests a question, it must return an answer (for example, true/false).
5. If a commit takes place in a function, the function name must end with the word
    “with.commit”.
6. A good general rule is to start every external function with the script code it is programmed
    in, so the origin of the function is clear.
7. The function name must only do what the name suggests: if a function is named check.xxxxx
    it must not update. However, if a function is named update.xxxx a check in the function is
    allowed to make an update decision.


```
Example,
```
```
The function name below suggests that it is only checking for the presence of a warehouse.
However, it is also performing an update. This can be highly misleading and hence this function
name is not allowed.
```
```
function domain tcbool check.if.warehouse.exists(domain tccwar i.warehouse)
{
db.retry.point()
select tcmcs003.cwar
from tcmcs003 for update
where tcmcs003._index1 = {:i.warehouse}
as set with 1 rows
selectdo
<some statements>
db.update(ttcmcs003, db.retry)
commit.transaction()
return(true)
endselect
return(false)
}
```
### 2.6 Marking Changes in Scripts

```
This section explains how changes should be marked in Scripts.
```
2.6.1 Standards for Marking Changes in Scripts

```
Any change in a script must always be marked with an appropriate eCare ID.
```
```
A brief description of the change is required as part of the Revision Text (Check-in text) as
follows:
```
```
|**********************************************************************
|* ID : EXDEV17.91231_
|* Name : Jaydeep Gajjar
|* Date : 13 September 2017
|* Desc : Changes done for stop Inventory allocation for Parts Sales
|* line from Rental.
|**********************************************************************
```
```
Adjusted lines must be marked. This implies that the previous (erroneous) program lines have to
remain in the program script.
```
```
Erroneous lines on position 1 are preceded by a pipe line “|” (except for DllUsage and
FunctionUsage which is comment).
```
```
At the end of such a line, the “|” character is appended by the Customization Project + eCare ID,
followed by .o (means old) or .n (means new.). Refer to the Customization Administration chapter
for more details on the same.
```
```
If more than one line has been changed, lines must be marked successively. The following
construction can be applied where the start is identified by s (= “start of marked lines”) or the end
by e (= “end of marked lines”). In the code snippet below, “.sn” and “.en” refer to a group of
statement which have been newly introduced. Similarly, “.so” and “.eo” must be used to comment
a block of statements.
```

| get.model.name() |M2016.73509.o
tcextdll2001.get.model.name(
erext401.equp, |M2016.73509.sn

erext401.clot,
equp.f) |M2016.73509.en
| |M2016.73509.so

| if not isspace(erext401.clot) then
| hold.clot = erext401.clot
| endif
| |M2016.73509.eo

When multiple new functions are written in the script under one IssueID, ensure that each
function is marked with a Start-New (“sn”) and End-New (“en”) comment.

function read.warehouse() |M2016.74505.sn
{
get.model.name()
<some statements>
}
|M2016.74505.en

function get.part.description() |M2016.74505.sn
{
<some statements>
} |M2016.74505.en

function get.tax.details() |M2016.74505.sn
{
<some statements>
} |M2016.74505.en


## 3. Software Coding Standards...........................................................................................................

```
This chapter provides the coding standard for each software component. All software
components must be created in e-Emphasys ERP-specific modules only, for example, “ext”.
```
### 3.1 Packages

```
The highest-level software component in e-Emphasys ERP is the Package.
```
```
Coding standard
```
```
Code: pp
```
```
New package codes are defined by Development.
```
```
Examples:
```
```
Package Description
```
```
er Rental
```
```
em e-Emphasys ERP Migration
```
```
ei e-Emphasys ERP Business Intelligence
```
```
xi Integration
```
### 3.2 Modules

```
A package consists of various modules. A module is primarily a technical grouping of sessions
and other software components
```
```
Coding standard
Code: mmm
```
```
The module code always has three characters (lower case).
```
```
Examples:
```
```
Module Description
```
```
ext e-Emphasys ERP module. This is generally the module used for any
new component.
```
```
ecp e-Emphasys ERP Correction Programs. This module should only be
used for Correction programs. Note that this module is not considered
for Translation of software components.
```
```
utl Internal utilities. Any session or software component meant for internal
utilities only must be created in this module. Note that this module is
not considered for Translation of software components.
```

### 3.3 Session....................................................................................................................................

```
Code: pp mmm s f xx o y cc
```
```
Characters Description
```
```
pp package code
```
```
mmm module code
```
```
s any free number
```
```
f function code
```
```
xx sequence number
```
```
o type of process
```
```
yyy
```
```
session sequence number; if more than one session is present for the
same main table, this sequence number can be used to define the
different sessions, starting with 0
```
```
Function
Code Meaning^
1 Single-occurrence maintain session
```
```
2 Process session
```
```
4 Print session
```
```
5 Multi-occurrence display session
```
```
6 MMT/Composite session
```
```
Examples:
```
```
pp mmm s f xx o yyy Explanation
```
```
wh ext 2 5 10 m 000 Multi occurrence display session for table whext
```
```
wh ext 2 1 10 s 000 Single occurrence maintain session for table whext
```
```
wh ext 2 5 10 m 100 Second multi occurrence display session for table whext
```
```
wh ext 2 4 10 m 000 Print session for table whext
wh ext 2 2 10 m 000 Process session on table whext
```
```
wh ext 2 6 10 m 000 Multi Main Table (MMT) session with main table whext
```

### 3.4 Correction Program Sessions...................................................................................................

```
A session pertaining to a correction program has the following naming convention:
```
```
<package><ecp><eCare ID>
```
```
For example: tdecp
```
### Where “ecp” stands for “e-Emphasys ERP Correction Program”.

### 3.5 Table

```
Coding standard
```
```
Code: pp mmm xxx
```
```
Characters Description
pp package code
mmm module code
xxx sequence number
```
```
Examples
```
```
pp mmm xxx
er ext 000
td ext 100
```
### 3.6 Table Fields

```
The standards for table fields consist of such elements as the field name, field sequence, the
standard label, and defaults.
```
3.6.1 Coding Standard

```
Code: pp mmm dddd (for e-Emphasys ERP Tables)
```
```
Code: pp mmm dddd<”.c”> (for standard Infor tables)
```
3.6.2 Field Name

```
Table fields are to be of 4 characters only. The exception is in case of standard Infor tables.
While adding a field to a standard Infor table, append a “.c” to the field name.
```
```
The first three characters of a combined table fields must be cmb. The fourth character is
```
### alphabetically incremented starting from “a”.

3.6.3 Field Sequence

```
It is essential that the field sequencing in the table is correct. The following must be taken into
account:
```
- Primary key fields must be in the proper sequence at the top of the table.
- Fields that logically belong together must also follow each other on the table. For example:
    o Order Quantity and Order Unit
    o Amount and Currency
- Text fields and combined fields must be at the bottom of the table.


3.6.4 Table Field Label

```
The table field label serves as the basic label for form prompts. It must therefore be
representative for a label in a single-occurrence session.
```
3.6.5 Defaults

```
When creating a record, fields must be filled with standard constants, if possible. A default
constant must be defined for enum fields. Where applicable, the same applies to other types of
fields.
```
3.6.6 Indices

```
Index descriptions are displayed in forms after selecting the Sort By option from the View Menu.
The following rules apply:
```
- The index description consists of the table field descriptions in the correct index sequence,
    divided by commas (,).
- Avoid duplicate keys.
- Be careful while making a multibyte field as part of an index. This is because there are
    constraints on the index size. Refer the “Known Limits” chapter in the Programmer’s manual
    for the same.

### 3.7 Domains

```
Coding standard
```
```
Code: pp mmm.dddd
```
```
Characters Description
pp package code
```
```
mmm
```
```
module code
where the
module code
should be “ext”
dddd mnemonic
For Enumerated domains, the following standards apply:
```
1. The domain code follows the same standard as shown above.
2. It is recommended that the Constant numbers are in the multiple 10. This makes it easier to
    add new constants in-between, if required in the future.


### 3.8 Messages

```
Coding standard
```
```
Code: pp t.mmm.xxxxx
```
```
Characters Description
pp package code
```
```
t
```
```
type
```
- “r” - if message used on Report layout, Do not
    use these messages in Scripts for UI events
    even if a reuse is possible. Create a new
    message instead.
- “g” – for all other messages other than those
    meant for report layout.
mmm module code
xxxxx sequence number

```
Example:
whg.ext.00001
```
```
tcr.ext.00020
```
```
pp t (Dot) mmm (Dot) xxxx Comment
```
```
wh g. ext. 00001
```
```
This is a message meant for General
Use
```
```
tc r. ext. 00020
```
```
This is a message which is used on a
Report Layout
```
### 3.9 Labels

```
The system automatically creates Labels for certain software components like Sessions, Tables
and so on. The standard outlined below applies only to all those Labels which need to be created
manually.
```
```
Coding standard for Labels meant for Report Layouts:
```
```
Code: pp r.mmm.xxxxx
```
```
Characters Description
pp package code
```
```
t
```
```
type
```
```
“r” – to be used literally. The Report labels are the
ones that are impacted the most during translation.
Hence this standard has been introduced to easily
identify those labels which have been used on the
Report layouts. Do not reuse these labels on the UI.
Create a new one instead using the standard
provided below for non-Report labels.
mmm module code
xxxxx sequence number
```

Example: ci r.ext. 00001

```
pp t (Dot) mmm (Dot) xxxx Comment
```
```
ci r. ext. 00001
```
```
This is a Label meant for use only on
the Report.
```
Coding standards for Labels other than those meant for Report Layouts:

Code: pp mmm.t.xxxxx

```
Characters Description
pp package code
```
```
t type^
```
- “g” – to be used literally
mmm module code
xxxxx sequence number

Example: ci.g.ext.00005

```
pp t (Dot) mmm (Dot) xxxx Comment
```
```
ci g. ext. 00001
```
```
This is a Label meant for use only on
the UI.
```
### • The fact that labels must be translated makes certain demands on the way they are defined

- Text - for instance prompts - consisting of multiple words (two or three words or even
    complete sentences) must ALWAYS be defined as one single label. Do not split up such word
    group labels. Also do not separate these labels, when they are placed one above the other,
    even if a word group looks like different words. In such cases, use one 2-line label – that is, a
    label with the height set to 2. If you break this rule, translation is impossible, since other
    languages will place the words in a different order, can use different word forms depending on
    a word's grammatical function or the gender of the associated word.

Example:

Do not use a combination of two labels: Print + Text. Use one single label for " Print Text "
(GERMAN = Text drucken)

Correct scenario using a single Label:

```
Label Code Label Description (English) Label Description (German)
```
```
ci.g.ext.00001 Print Text Text drucken
```
Incorrect scenario – splitting a single word into multiple labels can result in an incorrect
translation.

```
Label Code Label Description (English) Label Description (German)
```
```
ci.g.ext.00001 Print Drucken
```
```
ci.g.ext.00002 Text Text
```

- The length of a translation can be very different from that of the original label text. Label fields
    must therefore be as long as the available space in the layout allows, even if the text entered
    into it is only short.
Examples:
- BOM in German = Strukturstückliste
- Where-Used in German = Teileverwendungsnachweis

### 3.10 Questions

```
Coding standard
Code: pp t.mmm.xxxxx
```
```
Characters Description
pp package code
```
```
t type^
```
- “g” – this represent a General Use question.
mmm module code
xxxxx sequence number

```
Example: whg.ext.00001
```
```
pp t (Dot) mmm (Dot) xxxx Comment
```
```
wh g. ext. 00001
```
```
This is a Question meant for use only
on the UI.
```
### 3.11 Scripts

```
Coding standard – the script code is generally in-line with that of the Session code.
```
```
Code: pp mmm s f xx [o yyy]
```
```
Characters Description
pp package code
mmm module code
s any free number
f function code
xx sequence number
o type of process
```
```
yyy
```
```
session sequence number; if more than one session is
present for the same main table, this sequence number can
be used to define the different sessions, starting with 0
```

```
Functional Code Meaning
1 Single-occurrence maintain session
2 Process session
4 Print session
5 Multi-occurrence display session
6 MMT/Composite session
```
```
Examples:
```
```
pp mmm s f xx o yyy
wh ext 2 5 10
wh ext 2 5 10 m 100
```
### 3.12 Libraries...................................................................................................................................

```
Code: pp mmm dll xxxx
```
```
Characters Description
pp package code
mmm module code
dll dll; take this literally
xxxx sequence number
```
```
Example:
```
```
pp mmm dll xxxx
tc ext dll 2001
er ext dll 1000
```
### 3.13 Data Access Layer

```
Libraries used as Data Acces Layer (DAL) must have the same coding as their related table:
```
```
Code: pp mmm xxx
```
```
Characters Description
pp package code
mmm module code
xxx sequence number
```
```
Examples:
```
```
pp mmm xxx
```
### er ext 000

### td ext 100


### 3.14 SSRS Naming Conventions and Standards

```
This section explains the SSRS naming conventions and standards.
```
3.14.1 Standards for SSRS Naming Conventions and Standards

1. Name of the physical zip file containing the RDL:
    a. b<module code><report code>_<yyyy-mm-dd>_<issueid>.zip (For example:
       bslicisli22001100b_ 2017 - 07 - 22 _ 84095 .zip
          i. Note: report code is package + module
ii. Take note of the position of the underscores
       iii. Note that the date is separated by hyphens
2. Include the native report in the solution as a component even if there is no change in the
    native report. This is to ensure that dependencies are built up and by doing so it would be
    easy to find out the solution containing the latest version of the SSRS report.
3. Suffix the word “(SSRS)” as part of the report description. For example: “Parts Invoice
    (SSRS)”.

### 3.15 Naming Conventions for Variant Reports..................................................................................

```
This section explains the naming convention for Variant reports. Note that this naming
convention is for the part highlighted below i.e. only for the Report code part excluding the
Package and the Module.
```
```
Naming convention:
<1 letter module code in lower case><5 letters to denote the report ><3 letter free number>
For example: sinv 001. Thus, the complete report code becomes cislisinv001 (assuming “ci” and
“sli” to be the package and module respectively).
Additional Notes:
```
1. Refer Section 4.5 for the module code that should be used. Note that the module code
    needs to be written in lowercase.
2. Even though 5 letters have been reserved for the report, only 3 have been used in the above
    example. Hence 5 letters to denote the report is the maximum that is allowed.


## 4. General Guidelines

### 4.1 Guidelines for Creating PMC Solution

```
This section explains the guidelines for creating PMC Solution.
```
4.1.1 PMC Solution Guidelines

```
Following are the PMC Solution guidelines:
```
1. The number of the PMC solution must be the same as the eCare ID. Any further solutions on
    the same eCare ID must be suffixed as <pmc number>_1, <pmc number>_2 and so on.
2. Integration solutions must have a suffix of _INT. These solutions however may have
    components from both the “xi” as well as non-“xi” packages.
3. BI solutions must have a suffix of _EI. They should only contain components from the “ei”
    package.
4. Following Guidelines apply for any Development or a Maintenance spanning across different
    modules:
    a. The module codes need to be suffixed as part of the PMC number as follows:

```
Module Suffix to be used Subsequent solutions on the same eCare
```
```
Equipment _EQP _EQP_1, _EQP_2 and so on
Finance _FIN _FIN_1, _FIN_2 and so on
Rental _REN _REN_1, _REN_2 and so on
Parts _PRT _PRT_1, _PRT_2 and so on
Service _SER _SER_1, _SER_2 and so on
Integration _INT _INT_1, _INT_2 and so on
```
```
b. Common components must be in the Base PMC solution and other module-specific
solutions must ideally have co-requisite dependencies with the Base PMC. In absence of
a co-requisite dependency, it is possible that a customer may install only the Base PMC
but not the module-specific PMC’s. Such a scenario may lead to failures in the
application.
For example, if eCare ID is 93027, a single PMC containing all common components
must be present in PMC # 93027. Module-specific PMC’s for the same eCare like
93027_SER, 93027_PRT etc. must have a co-requisite dependency with 93027.
```
```
Ensure that any subsequent solution has a dependency (either manual or automatically
generated) on the Base solution if required.
```
5. Rules concerning the description of PMC solutions:
    1. The Name of the customer should never be a part of the PMC description.
    2. The Name of the VRC should never be a part of the PMC description.
    3. The description must not contain any special characters.
    4. Ensure that the description does not contain any grammatical and/or spelling errors.


### 4.2 Using the Revision Text Template

```
This section explains the revision text template to be used during the Check-in process.
```
4.2.1 Revision Text Template

```
Use the following template during the Check-in process. The ID in the template must match with
the Ident used in the scripts.
```
```
Template:
```
```
ID: <Customization Project Name/ID><eCare ID>
Name: <Your Name>
Date: <Check-in Date>
Desc: <A brief description of the change done>
```
```
For example,
```
```
ID: M2017.91427
Name: Viral Adeshra
Date: 27 Nov 2017
Desc: Tabbing out from the Service Order, Parts Sales Order field used to take 10-15 secs when
the Order series used to have a First Free number with Cache size as non-zero.
Consider a case of a Development or Maintenance issue spanning across more than one
module. In such a scenario, there may be a single script where multiple modules have made their
changes. Each module will have their own eCare ID. However, there would be a single PMC in
this case.
```
```
In such cases, please use the template shown below.
Template:
```
```
ID: <PMC number> ”(Ident Used: <Customization Project Name><eCare ID>)
```
```
Name: <Your Name>
```
```
Date: <Check-in Date>
```
```
Desc: <A brief description of the change done>
Example:
```
```
ID: EXDEV18.100689 (Ident Used: EXDEV18.100689_PRT)
```
```
In the above example, the PMC number is 100689 whereas the Ident used in the script is
EXDEV18.100689_PRT.
```

### 4.3 Using PMC Details Template

```
This section explains the PMC Details template to be used.
```
4.3.1 PMC Details Template

```
The template below must be followed for specifying the PMC details as part of the eCare
Discussion thread. If there are multiple solutions for a given eCare ID, each solution must be
specified as per the template shown below.
```
```
Template
```
```
Solution: <PMC Number> (BASE VRC = <specify the PMC BASE VRC>)
Description:
Technical Checklist (External):
A) Pre-Installation:
B) Post-Installation:
C) General:
Technical Checklist (Internal):
Example:
```
```
Solution : 103215_INT (BASE VRC = E50C 1 E501)
Description : Update External Business Partner (Logility Collaborate)
Technical Checklist (External):
A) Pre-Installation: NA
B) Post-Installation:
1) Execute CRDD for all relevant package combinations.
2) Run session Compile Labels (ttadv1243m000) for "xi" package.
C) General:
1) New fields 'Business Partner Masking', 'Masking Length' and 'Masking Character' will be
available on the interface 'Logility Collaborate Parameters (xikei0101m000)'
Technical Checklist (Internal):
1) Open table xikei001 and check if fields “mask” “mlen” and “mchr” are present.
```

### 4.4 Standard Post-Installation Write-up

```
The table below shows the post-installation write-up for the most common scenarios. The same
must be used as part of the Post-Installation notes while specifying the PMC details in the eCare
Discussion thread.
```
```
Sr
No
```
```
Scenario Write-up to be used
```
```
1 CRDD Execute CRDD for all relevant package combinations.
```
```
2 New table (Replace package names as appropriate in the write-up below).
1) Execute CRDD for all relevant package combinations.
2) Run session Create Table (ttaad4230m000) for all the relevant
companies and package as "<package name>"
```
#### 3

```
Label
compilation
```
```
(Replace package names as appropriate in the write-up below).
Run session Compile Labels (ttadv1243m000) for "<package name>"
package.
```
```
4 RDL Files Example below: (change the package, the RDL name and the zip file
name as appropriate. The word “Product VRC” is to be used instead
of the actual Product VRC name. This is because the solution can
either go to an E40 customer or an E50 customer. Hence, the
generic word “Product VRC” should be used. In case of a
customized RDL, please use the appropriate customized VRC
name).
Deploy erext440111009.rdl on the Report Server after solution
installation. RDL will be present in the path
"$BSE\application\er<Product VRC> \berext\bexterext440111009_2018-
02 - 22_93898.zip".
```
### 4.5 Customization Administration

```
All components that are changed or created need to be tracked in the system. This is done via
the Specific Option named Customization Parts which is available for all the software
components.
```
```
For every component that you create or modify, you need to link the Customization Part which is
comprised of 3 elements:
```
1. Customization Project
2. Project Part
3. Customization Part


The table below provides the Guidelines on the same.

```
VRC eCare Issue
Type
```
```
Customization Project Project Part Customizati
on Part
```
Product Development EXDEV<YY>

```
Where “YY” represents the
year.
```
```
<Module>
<Month
number>
```
```
<Last 3
digits of
eCare ID>
```
Product Maintenance M<YYYY>

```
Where “YYYY” represents
the year.
```
```
<Module>
<Month
number>
```
```
<Last 3
digits of
eCare ID>
```
Customization Development
or
Maintenance

```
<Customized Project
Code><YY>
The Customized Project
code is typically the same
as the custom VRC name
(For example: if the VRC is
E40M_1_rish, then the
Customized Project would
be “RISH” followed by the
current year (2 digits).
```
```
<Module>
<Month
number>
```
```
<Last 3
digits of
eCare ID>
```
```
Module Description
```
```
E Equipment
R Rental
P Parts
F Finance
S Service
I Integration
J Project
T Technical
L Planning
```
Idents in scripts

The idents in the script have the following convention:

<Customization Project>.<eCare ID>

For example:

EXDEV18.10290

M2018. 10290

SMSE 18 .11050


### 4.6 Guidelines for Correction Programs..........................................................................................

```
As mentioned above, correction programs must always be created in the “ecp” module.
Correction programs which are meant for end-customers must always be created as a 4GL
session instead of a 3GL.
Provide a Simulation mode along with a report which would enable the end-user to verify the
outcome before actually performing the correction.
```
### 4.7 Rules Concerning Modification of Large Tables

```
This section explains the rules concerning modification of large tables.
```
4.7.1 Standards for Modification of Large Tables

```
The list below shows the tables along with their number of records as of Nov 2017. We need to
avoid modification to these tables as the Reconfigure Table process (which is part of the CRDD
process) can take several hours for these cases. An alternate approach can be to introduce new
fields as part of a new extension table.
```
```
Table Description Records
```
```
tcext707 Inventory Position History 136,220,308.00
tfgld106 Finalized Transactions 124,965,218.00
tfgld418 Integration References 97,309,522.00
```
```
tcibd012 Effective Cost-Component Structure by Item 49,840,270.00
ticpr340 Effective Valuation Prices by Item and Warehouse 49,839,909.00
ticpr300 Standard Cost Prices by Item 49,808,290.00
```
```
tfgld410 Integration Lines Debit 48,654,761.00
tfgld417 Integration Lines Credit 48,654,761.00
tfert015 Service WIP Transactions 40,399,254.00
tdpcg031 Price Books 33,513,938.00
```
```
tfert016 GDNI Transactions in GL and Integrations 32,515,179.00
tcfin300 Service Financial Reconciliation 32,261,264.00
tffst300 Financial Statement Values 31,272,769.00
```
```
tdsls451 Sales Order Line History 31,227,555.00
tfext958 Temp. Table for Fin. Transactions by Order - D 27,977,286.00
whinr100 Inventory Transactions by Stock Point 27,574,189.00
```
```
whinh 270 Warehousing Order Lines History (Issues) 22,920,817.00
whinr115 Inventory Receipt Transaction Consumption Cost 18,019,533.00
```

### 4.8 Reading Parameter Tables

```
Use DLL “tcext.dll2001.read.parm(..)” to read any parameter table.
```
### 4.9 Usage of DAL2

```
Always use DAL2 for programming all applicable events instead of programming such events in
the UI scripts. Hence, the usage of DAL2 for such scenarios is mandatory.
```

