# SAIL GridField Usage Instructions

## Overview
GridField displays tabular data with built-in sorting, paging, selection, and search capabilities. It's optimized for read-only data presentation with rich formatting options for each column, including text, links, tags, buttons, and progress bars.

## ⚠️ CRITICAL: Function Variables in Grid Columns

**ONLY `fv!row` is available in grid columns!**

- ✅ `fv!row` - Access current row data
- ❌ `fv!index` - NOT AVAILABLE (causes runtime error)
- ❌ `fv!item` - NOT AVAILABLE (causes runtime error)

### ❌ COMMON MISTAKE: Using fv!index for Selection

**WRONG - This will fail:**
```sail
a!gridColumn(
  value: a!richTextItem(
    text: fv!row.name,
    link: a!dynamicLink(
      value: fv!index,  /* ERROR: fv!index doesn't exist in grid columns! */
      saveInto: local!selectedIndex
    )
  )
)
```

**RIGHT - Use grid's built-in selection:**
```sail
a!gridField(
  data: local!items,
  columns: { /* ... */ },
  selectable: true,
  selectionValue: local!selectedRows,  /* List of selected row data */
  selectionSaveInto: local!selectedRows,
  maxSelections: 1
)

/* Access selected item (selectionValue is always a LIST): */
local!firstSelected: index(local!selectedRows, 1, null)
```

## ⚠️ CRITICAL GRID COLUMN RESTRICTIONS

### ❌ NEVER Use These in GridColumns:
- **sideBySideLayout** - Use rich text with icons/text instead
- **columnsLayout** - Not supported in grid cells
- **cardLayout** - Too complex for grid cells
- **Arrays of components** - Only single components allowed
- **a!textField** - Use plain text or a!richTextDisplayField instead

### ✅ ONLY Use These in GridColumns:
- Plain text values
- `a!richTextDisplayField` (for formatting, icons, multiple lines)
- `a!tagField` (for status indicators)
- `a!buttonArrayLayout` (for actions)
- `a!progressBarField` (for progress indicators)
- `a!imageField` (for small images/avatars)

## When to Use GridField vs Alternatives

### ✅ Use GridField For:
- **Read-only data tables** - Displaying lists of records, transactions, or reports

### ❌ Use GridLayout Instead For:
- **Editable tables** - When users need to edit data inline

### ❌ Use Alternative Components For:
- **Key-value pairs** - Use `a!sideBySideLayout`
- **Cards/tiles** - Use `a!cardGroupLayout` for non-tabular presentation

## Data Sources for Static Mockups

### Using a!map for Static Data
For mockups and prototypes, use `a!map()` to create sample data:

```sail
a!localVariables(
  local!claimsData: {
    a!map(
      id: "CLM-2024-001",
      dateFiled: date(2024, 3, 15),
      policyHolder: "John Smith", 
      vehicleInfo: "2019 Honda Civic",
      incidentType: "Collision",
      claimAmount: 8500,
      status: "Under Review",
      adjuster: "Sarah Johnson"
    ),
    a!map(
      id: "CLM-2024-002", 
      dateFiled: date(2024, 3, 12),
      policyHolder: "Maria Garcia",
      vehicleInfo: "2021 Toyota Camry", 
      incidentType: "Theft",
      claimAmount: 15000,
      status: "Approved",
      adjuster: "Mike Chen"
    )
  },
  
  a!gridField(
    data: local!claimsData,
    /* columns configuration */
  )
)
```

### Data Structure Requirements
- Each data item must be an `a!map()` with consistent field names
- Field names should match the `sortField` values in columns
- Use appropriate data types (dates, numbers, text)

## Core GridField Parameters

### Essential Parameters
- **data**: List of a!map() items for static mockups
- **columns**: Array of `a!gridColumn()` definitions
- **pageSize**: Number of rows per page (default: 10)
- **emptyGridMessage**: Text shown when no data exists

### Selection Parameters
- **selectable**: Boolean to enable row selection
- **selectionValue**: Currently selected row data
- **selectionSaveInto**: Variables updated on selection change
- **maxSelections**: Maximum allowed selections

### Visual Styling Parameters
- **spacing**: `STANDARD` (default) or `DENSE`
- **borderStyle**: `STANDARD` (default) or `LIGHT`
- **shadeAlternateRows**: Boolean for row striping (default: true)
- **height**: ⚠️ **ONLY set if grid needs fixed height with scrolling rows** - otherwise exclude or use `AUTO`

### Height Options (Only When Fixed Height is Needed)
```sail
/* Only use these when you need fixed height with scrolling */
height: "SHORT",           /* ~120px - very compact */
height: "SHORT_PLUS",      /* ~180px - compact */  
height: "MEDIUM",          /* ~240px - standard fixed height */
height: "MEDIUM_PLUS",     /* ~300px - expanded */
height: "TALL",            /* ~360px - large */
height: "TALL_PLUS",       /* ~480px - very large */
height: "EXTRA_TALL",      /* ~600px - maximum */

/* Default behavior - grid grows with content */
height: "AUTO"             /* Or omit height parameter entirely */
```
## GridColumn Configuration

### Essential Column Parameters
```sail
a!gridColumn(
  label: "Column Header",           /* Required: Column title */
  value: fv!row.fieldName,          /* Required: Cell content */
  width: "NARROW",                  /* Column width */
  sortField: "fieldName",           /* Enables sorting */
  align: "START",                   /* Cell alignment */
  backgroundColor: "#FEF2F2".     /* Optional cell background color for highlighting */
)
```

### Column Width Guidelines
#### Strategy #1: grids with only a few columns
- Set all column widths to `AUTO`

#### Strategy #2: more complex spreadsheet-style grids
- Set each column width to a fixed value corresponding to content size:
    - `ICON`: Narrowest (~48px) - for single icons or indicators
    - `ICON_PLUS`: (~60px) - for short abbreviations
    - `NARROW`: (~82px) - very short numbers/text
    - `NARROW_PLUS`: (~172px) - short numbers/text
    - `MEDIUM`: (~260px) - names, short titles
    - `MEDIUM_PLUS`: (~430px) - short descriptions
    - `WIDE`: Large content (~600px) - long descriptions/paragraphs

❌❌❌ `WIDE_PLUS` is NOT a valid column width!!!

### Column Alignment Options
- **START**: Left-aligned (default)
- **CENTER**: Center-aligned
- **END**: Right-aligned (ideal for numbers, amounts)

## Common Column Content Patterns

### 1. Basic Text with Styling
```sail
a!gridColumn(
  label: "Policy Holder",
  value: fv!row.policyHolder,
  width: "NARROW_PLUS",
  sortField: "policyHolder"
)
```

### 2. Linked Text (Clickable Records)
```sail
a!gridColumn(
  label: "Claim ID", 
  value: a!richTextDisplayField(
    value: a!richTextItem(
      text: fv!row.id,
      style: "STRONG",
      link: a!recordLink(),
      linkStyle: "STANDALONE"
    ),
    labelPosition: "COLLAPSED"
  ),
  width: "NARROW_PLUS",
  sortField: "id"
)
```

### 3. Formatted Currency/Numbers
```sail
a!gridColumn(
  label: "Claim Amount",
  value: a!richTextDisplayField(
    value: a!richTextItem(
      text: dollar(fv!row.claimAmount),
      style: "STRONG"
    ),
    labelPosition: "COLLAPSED"
  ),
  width: "NARROW",
  align: "END",
  sortField: "claimAmount"
)
```

### 4. Status Tags
```sail
a!gridColumn(
  label: "Status",
  value: a!tagField(
    tags: a!tagItem(
      text: fv!row.status,
      backgroundColor: if(
        fv!row.status = "Approved",
        "POSITIVE",
        if(
          fv!row.status = "Denied",
          "NEGATIVE", 
          if(
            fv!row.status = "Under Review",
            "ACCENT",
            "SECONDARY"
          )
        )
      )
    ),
    size: "SMALL",
    labelPosition: "COLLAPSED"
  ),
  width: "NARROW_PLUS",
  sortField: "status"
)
```

### 5. Stacked Content (Multiple Lines per Cell)

```sail
a!gridColumn(
  label: "Claim Details",
  value: a!richTextDisplayField(
    value: {
      a!richTextItem(
        text: fv!row.id,
        style: "STRONG"
      ),
      char(10),
      a!richTextItem(
        text: text(fv!row.dateFiled, "MMM d, yyyy"),
        color: "SECONDARY",
        size: "SMALL"
      )
    },
    labelPosition: "COLLAPSED"
  ),
  width: "NARROW_PLUS",
  sortField: "id"
)
```

### 6. Icons with Text
**❌ WRONG - Using sideBySideLayout:**
```sail
a!gridColumn(
  value: a!sideBySideLayout(    /* ❌ NOT ALLOWED IN GRID COLUMNS */
    items: {
      a!sideBySideItem(item: a!richTextIcon(icon: "user")),
      a!sideBySideItem(item: a!richTextDisplayField(...))
    }
  )
)
```

**✅ CORRECT - Using richTextDisplayField with icons:**
```sail
a!gridColumn(
  label: "Contact Info",
  value: a!richTextDisplayField(
    value: {
      a!richTextIcon(icon: "user", size: "SMALL", color: "#6B7280"),
      " ",
      fv!row.policyHolder,
      char(10),
      a!richTextIcon(icon: "phone", size: "SMALL", color: "#6B7280"),
      " ",
      a!richTextItem(
        text: fv!row.phoneNumber,
        color: "SECONDARY",
        size: "SMALL"
      )
    },
    labelPosition: "COLLAPSED"
  ),
  width: "MEDIUM",
  sortField: "policyHolder"
)
```

### 7. Action Buttons
```sail
a!gridColumn(
  label: "Actions",
  value: a!buttonArrayLayout(
    buttons: {
      a!buttonWidget(
        label: "Delete",
        icon: "trash",
        style: "OUTLINE",
        size: "SMALL",
        color: "SECONDARY"
      ),
      a!buttonWidget(
        label: "Share",
        icon: "share",
        style: "OUTLINE",
        size: "SMALL",
        color: "SECONDARY"
      )
    },
    align: "START"
  ),
  width: "NARROW",
  align: "CENTER"
)
```

### 8. Conditional Background Color Highlighting

To conditionally set cell, row, or column background color, use `if` of `a!match` to set backgroundColor property on `a!gridColumn`

```sail
a!gridField(
  data: local!taskData,
  columns: {
    a!gridColumn(
      label: "Task Name",
      value: fv!row.taskName,
      width: "MEDIUM"
    ),
    a!gridColumn(
      label: "Due Date",
      value: text(fv!row.dueDate, "MMM d, yyyy"),
      width: "NARROW",
      backgroundColor: if( /* Set red background color if overdue */
        fv!row.dueDate < today(),
        "ERROR",
        "NONE"
      )
    ),
    a!gridColumn(
      label: "Assigned To",
      value: fv!row.assignedTo,
      width: "NARROW"
    )
  }
)
```

## Advanced Grid Patterns

### Standard Grid
```sail
a!gridField(
  data: local!claimsData,
  columns: {
    a!gridColumn(
      label: "Claim ID",
      value: a!richTextDisplayField(
        value: {
          a!richTextItem(
            text: fv!row.id,
            style: "STRONG",
            link: a!recordLink(),
            linkStyle: "STANDALONE"
          ),
          char(10),
          a!richTextItem(
            text: text(fv!row.dateFiled, "MMM d, yyyy"),
            color: "SECONDARY",
            size: "SMALL"
          )
        },
        labelPosition: "COLLAPSED"
      ),
      width: "NARROW_PLUS",
      sortField: "dateFiled"
    ),
    a!gridColumn(
      label: "Insured",
      value: a!richTextDisplayField(
        value: {
          a!richTextIcon(icon: "user", size: "SMALL", color: "#6B7280"),
          " ",
          fv!row.policyHolder,
          char(10),
          a!richTextIcon(icon: "car", size: "SMALL", color: "#6B7280"),
          " ",
          a!richTextItem(
            text: fv!row.vehicleInfo,
            color: "SECONDARY",
            size: "SMALL"
          )
        },
        labelPosition: "COLLAPSED"
      ),
      width: "MEDIUM",
      sortField: "policyHolder"
    ),
    a!gridColumn(
      label: "Claim Amount",
      value: a!richTextDisplayField(
        value: {
          a!richTextItem(
            text: dollar(fv!row.claimAmount),
            style: "STRONG",
            size: "MEDIUM"
          ),
          char(10),
          a!richTextItem(
            text: fv!row.incidentType,
            color: "SECONDARY",
            size: "SMALL"
          )
        },
        labelPosition: "COLLAPSED"
      ),
      width: "NARROW_PLUS",
      align: "END",
      sortField: "claimAmount"
    ),
    a!gridColumn(
      label: "Status",
      value: a!tagField(
        tags: a!tagItem(
          text: fv!row.status,
          backgroundColor: if(
            fv!row.status = "Approved",
            "POSITIVE",
            if(
              fv!row.status = "Denied",
              "NEGATIVE",
              if(
                fv!row.status = "Under Review",
                "ACCENT",
                "SECONDARY"
              )
            )
          )
        ),
        size: "SMALL",
        labelPosition: "COLLAPSED"
      ),
      width: "NARROW_PLUS",
      sortField: "status"
    ),
    a!gridColumn(
      label: "Actions",
      value: a!buttonArrayLayout(
        buttons: {
          a!buttonWidget(
            label: "Review",
            style: "OUTLINE",
            size: "SMALL",
            color: "ACCENT"
          )
        }
      ),
      width: "NARROW",
      align: "CENTER"
    )
  },
  pageSize: 15,
  initialSorts: {
    a!sortInfo(field: "dateFiled", ascending: false)
  },
  spacing: "STANDARD",
  borderStyle: "LIGHT",
  shadeAlternateRows: true,
  emptyGridMessage: "No claims found matching your criteria.",
  selectable: true,
  selectionValue: local!selectedClaims,
  selectionSaveInto: local!selectedClaims,
  maxSelections: 5
  /* No height parameter - grid grows with content */
)
```

## Sorting and Paging Configuration

### Initial Sorting
```sail
a!gridField(
  initialSorts: {
    a!sortInfo(field: "dateFiled", ascending: false),
    a!sortInfo(field: "claimAmount", ascending: false)
  },
  secondarySorts: {
    a!sortInfo(field: "policyHolder", ascending: true)
  }
)
```

### Paging Configuration
```sail
a!gridField(
  pageSize: 25,                    /* Rows per page */
  pagingSaveInto: local!gridPaging /* Track paging state */
)
```

## Visual Styling Options

### Grid Appearance (Auto Height)
```sail
a!gridField(
  spacing: "DENSE",              /* Compact row height */
  borderStyle: "LIGHT",          /* Subtle borders */
  shadeAlternateRows: false,     /* No row striping */
  /* No height parameter - grid grows with content */
)
```

### Grid Appearance (Fixed Height for Scrolling)
```sail
a!gridField(
  spacing: "DENSE",              /* Compact row height */
  borderStyle: "LIGHT",          /* Subtle borders */
  shadeAlternateRows: false,     /* No row striping */
  height: "MEDIUM"               /* Fixed height with scrollable rows */
)
```

### Selection Styling
```sail
a!gridField(
  selectable: true,
  selectionStyle: "CHECKBOX",     /* or "ROW_HIGHLIGHT" */
  showSelectionCount: true,       /* Display selection counter */
  maxSelections: 10
)
```

## Best Practices

### ✅ DO:
- Use appropriate column widths based on content type
- Provide meaningful `emptyGridMessage` text
- Use consistent text styling across similar columns
- Set `sortField` on columns that should be sortable
- Use `align: "END"` for numeric columns
- Choose appropriate `pageSize` for your use case (10-25 typical)
- Use `a!richTextDisplayField` for formatted content instead of sideBySideLayout
- **Exclude height parameter unless you need fixed height with scrolling**

### ❌ DON'T:
- **NEVER use `a!sideBySideLayout` inside grid columns** - Use multiple rich text items instead
- Put `a!columnsLayout` or `a!cardLayout` inside grid columns
- **Set height parameter unless you specifically need fixed height with scrolling**
- Use very wide grids with too many columns (consider stacking content)
- Forget to set column widths (can cause layout issues)

### Column Content Guidelines:
- **IDs/References**: Use linked rich text with `linkStyle: "STANDALONE"`
- **Names/Text**: Plain text or rich text with emphasis
- **Numbers/Currency**: Right-aligned rich text with formatting
- **Dates**: Formatted text with consistent date format
- **Status**: Tag fields with semantic colors
- **Actions**: Button arrays with consistent styling
- **Icons + Text**: Use rich text with `a!richTextIcon()` and `a!richTextItem()`

## Common Validation Issues

### ❌ WRONG - Using sideBySideLayout in grid column:
```sail
a!gridColumn(
  value: a!sideBySideLayout(     /* ❌ NOT ALLOWED IN GRID COLUMNS */
    items: {
      a!sideBySideItem(item: a!richTextIcon(...)),
      a!sideBySideItem(item: a!richTextDisplayField(...))
    }
  )
)
```

### ✅ CORRECT - Using richTextDisplayField:
```sail
a!gridColumn(
  value: a!richTextDisplayField(
    value: {
      a!richTextIcon(icon: "user", size: "SMALL"),
      " ",
      a!richTextItem(text: fv!row.name, style: "STRONG")
    },
    labelPosition: "COLLAPSED"
  )
)
```

## ⚠️ CRITICAL VALIDATION CHECKLIST

### Grid Column Content Validation:
- [ ] ❌ **NO sideBySideLayouts** in any grid column
- [ ] ❌ **NO columnsLayouts** in any grid column  
- [ ] ❌ **NO cardLayouts** in any grid column
- [ ] ❌ **NO arrays of components** in any grid column
- [ ] ✅ **ONLY** single components: richTextDisplayField, tagField, buttonArrayLayout, etc.

### Alternative Solutions:
- **Instead of sideBySideLayout**: Use `a!richTextDisplayField` with `a!richTextIcon()` and `a!richTextItem()`
- **Instead of complex layouts**: Break content into multiple columns or use stacked rich text
- **For multiple actions**: Use `a!buttonArrayLayout` with multiple buttons
- **Instead of fixed height**: Let grid grow naturally unless specifically constrained

### Grid Column Widths Validation:
- [ ] ✅ All columns have `AUTO` width - OR - all columns have fixed widths (e.g., `MEDIUM`)
- [ ] ❌ NO invalid width values like `WIDE_PLUS`
