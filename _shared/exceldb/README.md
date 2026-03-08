# Excel Knowledge Database

**Purpose**: Source data and guides for building the terminology and programming-syntax workbooks in Excel.

---

## Structure

```
_shared/exceldb/
├── README.md           # This file
├── implementation.md   # Step-by-step build guide (terminology workbook)
├── reference.md       # Column definitions, L1/L2, rules
├── excel_functions.md  # Counting and search-bar formulas
├── data/              # Terminology workbook source
│   ├── terminology_reference.csv   # Main term data (~957 terms)
│   ├── definitions.csv             # L1/L2/Priority definitions
│   └── schema.csv                  # Column and L1/Type rules
└── syntax/            # Programming syntax reference
    ├── programming_schema.csv      # Column and L1/L2 rules
    └── programming_syntax.csv     # Syntax items by language
```

---

## Quick start

1. **Terminology workbook** - [implementation.md](implementation.md) (start with Iteration 1). Import [data/terminology_reference.csv](data/terminology_reference.csv).
2. **Reference** - [reference.md](reference.md) for columns, categories, and rules.
3. **Formulas** - [excel_functions.md](excel_functions.md) for counting and search.

---

## Files

| File | Purpose |
|------|---------|
| [implementation.md](implementation.md) | Build guide: iterations 1–4 for the terminology workbook |
| [reference.md](reference.md) | Schema, L1/L2, priority, file roles |
| [excel_functions.md](excel_functions.md) | Excel formulas for the workbook |
| data/terminology_reference.csv | Source data for the terminology table |
| data/schema.csv | Column definitions, L1/Type/Priority rules |
| data/definitions.csv | L1/L2/Priority definitions |
| syntax/programming_schema.csv | Schema for programming syntax table |
| syntax/programming_syntax.csv | Programming syntax reference data |
