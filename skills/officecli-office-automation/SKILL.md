---
name: officecli-office-automation
description: Use OfficeCLI to create, read, and edit Word, Excel, and PowerPoint files from the command line with AI-friendly commands.
triggers:
  - "create a Word document"
  - "edit an Excel spreadsheet"
  - "make a PowerPoint presentation"
  - "modify Office files"
  - "automate document generation"
  - "convert Office files to HTML"
  - "add slides to presentation"
  - "update spreadsheet formulas"
---

# OfficeCLI Office Automation Skill

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

OfficeCLI is an Office suite purpose-built for AI agents to create, read, and edit Word (.docx), Excel (.xlsx), and PowerPoint (.pptx) files. It's a single binary with no Office installation required, featuring a unified CLI interface and built-in rendering to HTML/PNG.

## Installation

**macOS / Linux:**
```bash
curl -fsSL https://raw.githubusercontent.com/iOfficeAI/OfficeCLI/main/install.sh | bash
```

**Windows (PowerShell):**
```powershell
irm https://raw.githubusercontent.com/iOfficeAI/OfficeCLI/main/install.ps1 | iex
```

**Manual installation:**
1. Download the binary for your platform from [GitHub Releases](https://github.com/iOfficeAI/OfficeCLI/releases)
2. Run `officecli install` to add to PATH

**Verify installation:**
```bash
officecli --version
```

## Core Commands

### File Operations

**Create a new document:**
```bash
# Blank files
officecli create document.docx
officecli create spreadsheet.xlsx
officecli create presentation.pptx

# With initial content
officecli create report.docx --prop title="Q4 Report"
```

**View document content:**
```bash
# View as outline (tree structure)
officecli view document.docx outline

# View as plain text
officecli view document.docx text

# View as HTML (opens in browser)
officecli view document.docx html

# View as JSON structure
officecli view document.docx json
```

**Get specific elements:**
```bash
# Get element details as JSON
officecli get document.docx '/paragraph[1]' --json
officecli get spreadsheet.xlsx '$Sheet1:A1' --json
officecli get presentation.pptx '/slide[1]/shape[2]' --json
```

### Live Preview

**Watch mode (auto-refresh in browser):**
```bash
# Start live preview server on http://localhost:26315
officecli watch presentation.pptx

# Custom port
officecli watch presentation.pptx --port 8080

# Every edit you make will auto-refresh the browser
```

## Word Documents (.docx)

### Path Syntax
- `/paragraph[N]` - Nth paragraph (1-indexed)
- `/paragraph[N]/run[M]` - Mth run in Nth paragraph
- `/table[N]` - Nth table
- `/table[N]/row[R]/cell[C]` - Cell in table

### Add Content

**Add paragraphs:**
```bash
# Simple paragraph
officecli add report.docx / --type paragraph --prop text="Executive Summary"

# Styled paragraph
officecli add report.docx / --type paragraph \
  --prop text="Quarterly Revenue Analysis" \
  --prop style=Heading1 \
  --prop align=center

# With formatting
officecli add report.docx / --type paragraph \
  --prop text="Important notice" \
  --prop bold=true \
  --prop color=FF0000 \
  --prop size=14
```

**Add runs (inline formatting):**
```bash
officecli add report.docx '/paragraph[1]' --type run \
  --prop text="bold text" \
  --prop bold=true
```

**Add tables:**
```bash
# 3x3 table
officecli add report.docx / --type table \
  --prop rows=3 \
  --prop cols=3

# Set cell content
officecli set report.docx '/table[1]/row[1]/cell[1]' \
  --prop text="Header 1"
```

**Add images:**
```bash
officecli add report.docx / --type picture \
  --prop src=chart.png \
  --prop width=15cm \
  --prop height=10cm
```

**Add headers/footers:**
```bash
# Add header to first section
officecli add report.docx '/section[1]' --type header \
  --prop type=default

# Add content to header
officecli add report.docx '/section[1]/header[1]' --type paragraph \
  --prop text="Confidential Report" \
  --prop align=right
```

### Modify Content

**Set paragraph properties:**
```bash
officecli set report.docx '/paragraph[1]' \
  --prop text="Updated title" \
  --prop style=Heading1 \
  --prop align=center \
  --prop spaceBefore=240 \
  --prop spaceAfter=120
```

**Set run properties:**
```bash
officecli set report.docx '/paragraph[2]/run[1]' \
  --prop bold=true \
  --prop italic=true \
  --prop font=Arial \
  --prop size=12 \
  --prop color=0000FF
```

### Remove Content

```bash
# Remove paragraph
officecli remove report.docx '/paragraph[3]'

# Remove table
officecli remove report.docx '/table[1]'
```

### RTL and i18n Support

```bash
# RTL paragraph (Arabic, Hebrew)
officecli add report.docx / --type paragraph \
  --prop text="مرحبا بك" \
  --prop direction=rtl \
  --prop lang.cs=ar-SA

# Per-script fonts
officecli add report.docx / --type paragraph \
  --prop text="Hello 世界 مرحبا" \
  --prop font.latin=Arial \
  --prop font.ea="MS Gothic" \
  --prop font.cs="Arial Unicode MS"
```

## Excel Spreadsheets (.xlsx)

### Cell Addressing
- `$SheetName:A1` - Cell A1 in named sheet
- `$Sheet1:B2:D4` - Range B2:D4
- `/sheet[1]/cell[A1]` - Path-based addressing

### Add Content

**Add sheets:**
```bash
# Add new sheet
officecli add budget.xlsx / --type sheet --prop name="Q1 Budget"

# Add with properties
officecli add budget.xlsx / --type sheet \
  --prop name="Summary" \
  --prop tabColor=FF0000 \
  --prop state=visible
```

**Add cells:**
```bash
# Simple value
officecli add budget.xlsx '$Sheet1:A1' --type cell \
  --prop value="Revenue"

# Number with formatting
officecli add budget.xlsx '$Sheet1:B1' --type cell \
  --prop value=125000 \
  --prop format='$#,##0.00'

# Formula
officecli add budget.xlsx '$Sheet1:C1' --type cell \
  --prop formula='=SUM(B1:B10)'

# Date
officecli add budget.xlsx '$Sheet1:D1' --type cell \
  --prop value=2024-01-15 \
  --prop format='yyyy-mm-dd'
```

**Batch add cells:**
```bash
# Multiple cells
officecli add budget.xlsx '$Sheet1:A1' --type cell --prop value="Name"
officecli add budget.xlsx '$Sheet1:B1' --type cell --prop value="Amount"
officecli add budget.xlsx '$Sheet1:A2' --type cell --prop value="Sales"
officecli add budget.xlsx '$Sheet1:B2' --type cell --prop value=50000
```

**Add tables:**
```bash
# Create table from range
officecli add budget.xlsx '$Sheet1:A1:C10' --type table \
  --prop name="SalesData" \
  --prop style=TableStyleMedium2 \
  --prop showHeader=true
```

**Add charts:**
```bash
# Column chart
officecli add budget.xlsx '$Sheet1' --type chart \
  --prop chartType=column \
  --prop dataRange='$Sheet1:A1:B10' \
  --prop title="Sales by Region" \
  --prop x=5cm \
  --prop y=2cm

# Pie chart
officecli add budget.xlsx '$Sheet1' --type chart \
  --prop chartType=pie \
  --prop dataRange='$Sheet1:A1:B5' \
  --prop title="Market Share"
```

**Add pivot tables:**
```bash
officecli add budget.xlsx '$Sheet2:A1' --type pivottable \
  --prop sourceRange='$Sheet1:A1:D100' \
  --prop rowFields=Region \
  --prop dataFields=Sales \
  --prop name="SalesByRegion"
```

### Modify Content

**Set cell values:**
```bash
officecli set budget.xlsx '$Sheet1:A1' --prop value="Updated"

# With formatting
officecli set budget.xlsx '$Sheet1:B2' \
  --prop value=75000 \
  --prop format='$#,##0.00' \
  --prop bold=true \
  --prop color=FF0000
```

**Set sheet properties:**
```bash
officecli set budget.xlsx '/sheet[1]' \
  --prop name="Renamed Sheet" \
  --prop tabColor=00FF00 \
  --prop direction=rtl
```

### Formulas

**150+ built-in functions with auto-evaluation:**
```bash
# Math
officecli add budget.xlsx '$Sheet1:C1' --type cell \
  --prop formula='=SUM(A1:A10)'

# Conditional
officecli add budget.xlsx '$Sheet1:D1' --type cell \
  --prop formula='=IF(B1>1000,"High","Low")'

# Lookup
officecli add budget.xlsx '$Sheet1:E1' --type cell \
  --prop formula='=VLOOKUP(A1,$Sheet2:A:B,2,FALSE)'

# Date functions
officecli add budget.xlsx '$Sheet1:F1' --type cell \
  --prop formula='=TODAY()'
```

### Sort Data

```bash
# Sort range by column
officecli add budget.xlsx '$Sheet1:A1:C10' --type sort \
  --prop key1=B \
  --prop order1=desc

# Multi-key sort
officecli add budget.xlsx '$Sheet1:A1:D20' --type sort \
  --prop key1=A \
  --prop order1=asc \
  --prop key2=B \
  --prop order2=desc
```

### Conditional Formatting

```bash
officecli add budget.xlsx '$Sheet1:B2:B10' --type conditionalformatting \
  --prop type=colorScale \
  --prop minColor=FF0000 \
  --prop maxColor=00FF00
```

## PowerPoint Presentations (.pptx)

### Path Syntax
- `/slide[N]` - Nth slide
- `/slide[N]/shape[M]` - Mth shape on slide
- `/slide[N]/table[M]` - Mth table on slide

### Add Content

**Add slides:**
```bash
# Blank slide
officecli add deck.pptx / --type slide

# With title
officecli add deck.pptx / --type slide \
  --prop title="Introduction"

# With background
officecli add deck.pptx / --type slide \
  --prop title="Dark Theme" \
  --prop background=1A1A2E
```

**Add shapes (text boxes):**
```bash
# Simple text box
officecli add deck.pptx '/slide[1]' --type shape \
  --prop text="Hello World" \
  --prop x=2cm \
  --prop y=3cm \
  --prop width=10cm \
  --prop height=2cm

# Styled text box
officecli add deck.pptx '/slide[1]' --type shape \
  --prop text="Key Findings" \
  --prop x=1cm \
  --prop y=2cm \
  --prop font=Arial \
  --prop size=32 \
  --prop color=FFFFFF \
  --prop bold=true \
  --prop align=center
```

**Add images:**
```bash
# Basic image
officecli add deck.pptx '/slide[1]' --type picture \
  --prop src=logo.png \
  --prop x=1cm \
  --prop y=1cm \
  --prop width=5cm \
  --prop height=3cm

# With effects
officecli add deck.pptx '/slide[1]' --type picture \
  --prop src=photo.jpg \
  --prop x=5cm \
  --prop y=5cm \
  --prop brightness=20 \
  --prop contrast=10 \
  --prop shadow=true
```

**Add tables:**
```bash
# Create table
officecli add deck.pptx '/slide[1]' --type table \
  --prop rows=3 \
  --prop cols=3 \
  --prop x=2cm \
  --prop y=5cm

# Set cell content
officecli set deck.pptx '/slide[1]/table[1]/row[1]/cell[1]' \
  --prop text="Header 1" \
  --prop bold=true
```

**Add charts:**
```bash
officecli add deck.pptx '/slide[1]' --type chart \
  --prop chartType=column \
  --prop title="Sales Growth" \
  --prop x=2cm \
  --prop y=5cm \
  --prop width=20cm \
  --prop height=10cm
```

**Add 3D models:**
```bash
officecli add deck.pptx '/slide[1]' --type 3dmodel \
  --prop src=model.glb \
  --prop x=5cm \
  --prop y=5cm \
  --prop width=10cm \
  --prop height=10cm
```

### Slide Transitions and Animations

**Add transitions:**
```bash
officecli set deck.pptx '/slide[2]' \
  --prop transition=morph \
  --prop duration=1000
```

**Add animations:**
```bash
officecli add deck.pptx '/slide[1]/shape[1]' --type animation \
  --prop effect=fadeIn \
  --prop duration=500
```

### Modify Content

**Set shape properties:**
```bash
officecli set deck.pptx '/slide[1]/shape[1]' \
  --prop text="Updated text" \
  --prop font=Calibri \
  --prop size=28 \
  --prop color=333333 \
  --prop fill=F0F0F0
```

**Set slide properties:**
```bash
officecli set deck.pptx '/slide[1]' \
  --prop title="New Title" \
  --prop background=FFFFFF \
  --prop hidden=false
```

### Speaker Notes

```bash
# Add notes to slide
officecli add deck.pptx '/slide[1]' --type notes \
  --prop text="Remember to mention the Q3 results"
```

## Common Patterns

### Batch Document Generation

```bash
#!/bin/bash
# Generate monthly reports

for month in Jan Feb Mar Apr; do
  officecli create "report-${month}.docx"
  officecli add "report-${month}.docx" / --type paragraph \
    --prop text="${month} Report" \
    --prop style=Heading1
  officecli add "report-${month}.docx" / --type paragraph \
    --prop text="Generated on $(date)"
done
```

### CSV to Excel

```bash
# Import CSV data
officecli create data.xlsx
officecli add data.xlsx '$Sheet1' --type import \
  --prop src=data.csv \
  --prop format=csv
```

### Multi-slide Presentation from Template

```bash
#!/bin/bash
# Create presentation with consistent styling

officecli create pitch.pptx

# Title slide
officecli add pitch.pptx / --type slide \
  --prop title="Product Launch" \
  --prop background=1A1A2E

# Content slides
for topic in "Problem" "Solution" "Market" "Team"; do
  officecli add pitch.pptx / --type slide \
    --prop title="$topic" \
    --prop background=1A1A2E
  
  officecli add pitch.pptx '/slide[-1]' --type shape \
    --prop text="Content for $topic" \
    --prop x=2cm \
    --prop y=5cm \
    --prop color=FFFFFF
done
```

### Automated Report with Charts

```bash
#!/bin/bash
# Generate Excel report with chart

officecli create sales.xlsx

# Add headers
officecli add sales.xlsx '$Sheet1:A1' --type cell --prop value="Month"
officecli add sales.xlsx '$Sheet1:B1' --type cell --prop value="Revenue"

# Add data
officecli add sales.xlsx '$Sheet1:A2' --type cell --prop value="January"
officecli add sales.xlsx '$Sheet1:B2' --type cell --prop value=50000
officecli add sales.xlsx '$Sheet1:A3' --type cell --prop value="February"
officecli add sales.xlsx '$Sheet1:B3' --type cell --prop value=65000
officecli add sales.xlsx '$Sheet1:A4' --type cell --prop value="March"
officecli add sales.xlsx '$Sheet1:B4' --type cell --prop value=72000

# Add chart
officecli add sales.xlsx '$Sheet1' --type chart \
  --prop chartType=line \
  --prop dataRange='$Sheet1:A1:B4' \
  --prop title="Monthly Revenue" \
  --prop x=5cm \
  --prop y=2cm
```

### Extract and Transform

```bash
#!/bin/bash
# Extract text from Word doc, process, create new doc

# Extract text
text=$(officecli view input.docx text)

# Process (example: convert to uppercase)
processed=$(echo "$text" | tr '[:lower:]' '[:upper:]')

# Create new document
officecli create output.docx
officecli add output.docx / --type paragraph --prop text="$processed"
```

## Integration with Scripts

### Python Integration

```python
import subprocess
import json

def create_report(title, content):
    """Create a Word document using OfficeCLI."""
    filename = f"{title}.docx"
    
    # Create document
    subprocess.run(["officecli", "create", filename])
    
    # Add title
    subprocess.run([
        "officecli", "add", filename, "/",
        "--type", "paragraph",
        "--prop", f"text={title}",
        "--prop", "style=Heading1"
    ])
    
    # Add content
    for paragraph in content:
        subprocess.run([
            "officecli", "add", filename, "/",
            "--type", "paragraph",
            "--prop", f"text={paragraph}"
        ])
    
    return filename

def get_cell_value(filename, cell):
    """Get cell value from Excel file."""
    result = subprocess.run([
        "officecli", "get", filename, cell, "--json"
    ], capture_output=True, text=True)
    
    data = json.loads(result.stdout)
    return data.get("attributes", {}).get("value")

# Usage
create_report("Q1 Summary", [
    "Revenue increased by 25%.",
    "Customer satisfaction improved.",
    "New product launch successful."
])

value = get_cell_value("budget.xlsx", "$Sheet1:B2")
print(f"Budget value: {value}")
```

### Node.js Integration

```javascript
const { execSync } = require('child_process');

function createPresentation(title, slides) {
  const filename = `${title}.pptx`;
  
  // Create presentation
  execSync(`officecli create "${filename}"`);
  
  // Add slides
  slides.forEach(slide => {
    execSync(`officecli add "${filename}" / --type slide --prop title="${slide.title}"`);
    
    if (slide.content) {
      execSync(`officecli add "${filename}" '/slide[-1]' --type shape --prop text="${slide.content}" --prop x=2cm --prop y=5cm`);
    }
  });
  
  return filename;
}

function viewAsHTML(filename) {
  execSync(`officecli view "${filename}" html`);
}

// Usage
const slides = [
  { title: "Introduction", content: "Welcome to our presentation" },
  { title: "Main Points", content: "Key findings and results" },
  { title: "Conclusion", content: "Thank you" }
];

const file = createPresentation("meeting", slides);
viewAsHTML(file);
```

## Troubleshooting

### Binary not found after installation
```bash
# Check if binary is in PATH
which officecli

# If not, run install again
officecli install

# Or manually add to PATH
export PATH="$PATH:$HOME/.officecli/bin"
```

### File not opening in Office
```bash
# Verify file is valid
officecli view document.docx outline

# Check file wasn't corrupted
file document.docx
# Should show: "Microsoft Word 2007+"
```

### Preview not refreshing in watch mode
```bash
# Ensure watch server is running
officecli watch document.docx

# Check if port is blocked
officecli watch document.docx --port 8081

# Clear browser cache or hard refresh (Ctrl+Shift+R / Cmd+Shift+R)
```

### Permission denied errors
```bash
# Ensure file isn't open in another program
# Close Office/LibreOffice/other editors

# Check file permissions
ls -l document.docx
chmod 644 document.docx
```

### Invalid path errors
```bash
# Verify document structure first
officecli view document.docx outline

# Paths are 1-indexed: /paragraph[1], /slide[1]
# Use --json to see exact paths
officecli get document.docx / --json
```

### Formula not calculating in Excel
```bash
# OfficeCLI auto-evaluates 150+ functions
# Unsupported functions show #NAME? error
# Check formula syntax:
officecli get spreadsheet.xlsx '$Sheet1:A1' --json
```

### RTL text not rendering correctly
```bash
# Set direction property
officecli set document.docx '/paragraph[1]' \
  --prop direction=rtl \
  --prop lang.cs=ar-SA

# For complex scripts, set appropriate font
officecli set document.docx '/paragraph[1]' \
  --prop font.cs="Arial Unicode MS"
```

### Image not appearing
```bash
# Check image path is correct and file exists
ls -l image.png

# Ensure image format is supported (PNG/JPG/GIF/SVG)
file image.png

# Use absolute path if relative path fails
officecli add doc.docx / --type picture --prop src=/full/path/to/image.png
```

## Configuration

OfficeCLI is designed to work out-of-the-box with no configuration needed. All settings are passed as command-line flags.

**Common flags:**
- `--json` - Output as JSON
- `--port PORT` - Custom port for watch server (default: 26315)
- `--prop key=value` - Set element properties

## Documentation

- **Wiki:** https://github.com/iOfficeAI/OfficeCLI/wiki
- **Word Reference:** https://github.com/iOfficeAI/OfficeCLI/wiki/word-paragraph
- **Excel Reference:** https://github.com/iOfficeAI/OfficeCLI/wiki/excel-cell
- **PowerPoint Reference:** https://github.com/iOfficeAI/OfficeCLI/wiki/ppt-slide
- **i18n Support:** https://github.com/iOfficeAI/OfficeCLI/wiki/i18n
- **GitHub:** https://github.com/iOfficeAI/OfficeCLI
- **Discord:** https://discord.gg/2QAwJn7Egx

## Key Concepts

1. **XPath-like addressing:** All elements use path notation (`/paragraph[1]`, `$Sheet1:A1`, `/slide[1]/shape[2]`)
2. **Unified interface:** Same `add`, `set`, `get`, `remove`, `view` verbs across all formats
3. **Live preview:** `watch` command provides real-time browser preview with auto-refresh
4. **No dependencies:** Single binary, no Office installation needed
5. **AI-first design:** Commands optimized for LLM generation and automation
6. **Format preservation:** Edit documents without breaking complex formatting
7. **Rich i18n:** Full RTL support, per-script fonts, locale-aware rendering

When helping users with OfficeCLI, focus on the unified command structure and path-based addressing. The same patterns work across Word, Excel, and PowerPoint with format-specific properties.
