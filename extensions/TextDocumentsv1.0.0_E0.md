Contract-Header Automation Specification v1.0.0_E01  
**Extension to Text Documents and Forms**  
**A compact interface standard for LLM-aware documents**  
**Draft — February 2026**

**Written By: Aphix**
*2026-02-04*

> *Disclaimer:* There are occasional portions discussing a concept of "safety." That concept is irrelevant for textual documents as they are simply meant to convey information. YAML is discouraged.

***

## 1. Motivation for Text Documents

Code files have crisp boundaries—functions take parameters and return values. Text documents? They're a wilderness of intent, where "inputs" might be raw facts and "outputs" might be insights, decisions, or even new questions. Yet the same token‑efficiency logic applies with double force: LLMs choke on long documents unless given semantic footholds.

Enter the **Document Contract-Header**: a standardized block declaring what information the document *expects to receive* (context, data sources, required reading) and what it *produces* (conclusions, action items, derived knowledge). This transforms rambling reports, meeting notes, forms, and specifications into **composable artifacts** that agents can chain, summarize, or execute against without reading every word.

The principle remains: machines—and their human operators—should know a document's purpose from its first ten lines. What changes is that text documents deal in *fuzzy contracts*: probabilities, interpretations, recommendations rather than strict types.

***

## 2. Core Concept

Every text document gains an optional but highly‑recommended **Contract-Header** at the top, formatted as structured comments or YAML frontmatter. The header declares:

**Inputs**: What information or context must exist for the document to be meaningful.  
**Outputs**: What new knowledge, decisions, or artifacts the document produces.  
**Confidence**: How certain the author is of their conclusions (crucial for AI chaining).  
**ActionRequired**: Concrete next steps with assignees when applicable.

### Example: Meeting Notes

```
---
Contract-Header v1.1 - Document
File: 2026-02-04-product-team-notes.md
Type: meeting-notes
Description: Product team sync on Q1 priorities and 3D printer integration roadmap.
Inputs: 
  - Q1 OKRs (spreadsheet link or prior document)
  - Current sprint velocity data
  - Printer hardware specs (safety envelope document)
Outputs:
  - Prioritized feature list (3 items, confidence: high)
  - Assigned owners and deadlines
  - Risk register update (2 new items)
ActionRequired:
  - Alice: finalize printer safety spec by 2026-02-10
Confidence: 90%
Dependencies: [q1-okrs.md, sprint-42-velocity.csv]
LastGenerated: 2026-02-04T21:58:00Z
---
```

An agent processing this document can immediately extract:  
- Skip reading if Q1 OKRs are missing  
- Trust the 3‑item feature list as "high confidence"  
- Queue Alice's task automatically  
- Only dive deeper if risk assessment needed

***

## 3. Document Header Schema

| Field | Type | Description |
|-------|------|---------------------|
| `Contract-Header` | string | "v1.1 - Document" identifier |
| `File` | string | Document filename |
| `Type` | enum | `meeting-notes`, `report`, `form`, `spec`, `memo`, `proposal` |
| `Description` | string | ≤120 chars purpose statement |
| `Inputs` | list | Required context/documents/data |
| `Outputs` | list | Knowledge/decisions produced |
| `Confidence` | string | "high/medium/low" or percentage |
| `ActionRequired` | list | Concrete tasks w/owners |
| `Dependencies` | list | Linked documents/files |
| `LastGenerated` | ISO timestamp | Audit trail |

***

## 4. Document Types and Examples

### 4.1 Meeting Notes

**Raw input notes:**
```
Product team 2/4: Printer integration blocked on safety spec. Alice researching UL standards.
Bob: velocity down 15% due to printer debugging. Q1 OKR #3 at risk.
Priorities: 1) Safety interlock, 2) filament sensor, 3) UI polish.
```

**With Contract-Header:**
```
---
Contract-Header v1.1 - Document
File: product-2026-02-04.md
Type: meeting-notes
Description: Q1 printer integration sync - safety and velocity discussion
Inputs: Q1 OKRs, sprint velocity data
Outputs: 
  - Top 3 printer priorities (safety first)
  - Alice owns safety spec research
Confidence: high
ActionRequired: 
  - Alice: printer safety spec by 2026-02-10
Dependencies: [q1-okrs.md]
---
Product team 2/4: Printer integration blocked...
```

### 4.2 Technical Reports

```
---
Contract-Header v1.1 - Document
File: printer-load-test-report.md
Type: report
Description: 72hr stress test of cosmically-dangerous 3D printer under max load
Inputs: 
  - Test gcode files (24 total)
  - Temperature/strain sensor logs
  - Firmware version 2.1.7
Outputs:
  - Max safe bed temp: 245C (was 230C)
  - Motor strain anomaly at 85% duty cycle
  - Recommendation: limit duty cycle to 80%
Confidence: 95%
ActionRequired: Update safety spec with new thermal limits
---
## Executive Summary
72hr test completed successfully. Discovered printer can handle 245C bed temp...
```

### 4.3 Forms and Templates

**Employee Onboarding Form:**
```
---
Contract-Header v1.1 - Document
File: onboarding-alice-smith.md
Type: form
Description: New hire onboarding checklist for Alice Smith, printer safety specialist
Inputs: 
  - Employee contract details
  - Manager approval
  - Safety training completion certificate
Outputs:
  - Hardware access credentials
  - Team assignment confirmation
  - First week goals document
ActionRequired: 
  - HR: issue badge by 2026-02-06
  - Manager: schedule safety training
Status: pending
---
# Alice Smith Onboarding Checklist
[ ] Laptop issued
[ ] Email account created
[X] Safety training scheduled
```

### 4.4 Proposals and Specifications

```
---
Contract-Header v1.1 - Document
File: printer-safety-interlock-proposal.md
Type: proposal
Description: Hardware/software interlock system to prevent cosmically-dangerous printer accidents
Inputs: Recent load test report, UL safety standards
Outputs:
  - Proposed safety envelope (15 parameters)
  - Cost estimate: $2.8k materials + 40hr engineering
  - Implementation timeline: 3 weeks
Confidence: medium (needs thermal modeling)
ActionRequired: Engineering review by 2026-02-12
Dependencies: [printer-load-test-report.md]
---
## Safety Interlock Proposal
Following the recent 72hr load test revealing thermal margin...
```

***

## 5. Automation for Text Headers

### 5.1 Python Generator for Markdown/YAML

```python
import re, datetime, yaml, frontmatter
from typing import Dict, Any, List

def generate_document_header(filepath: str) -> Dict[str, Any]:
    """Auto-generate Contract-Header for text documents."""
    
    with open(filepath, 'r', encoding='utf-8') as f:
        content = f.read()
    
    filename = filepath.split('/')[-1]
    
    # Simple heuristics for document type and content
    doc_type = infer_document_type(content)
    description = infer_description(content[:500])
    
    # Extract action items, people, dates heuristically
    actions = extract_action_items(content)
    
    metadata = {
        'Contract-Header': 'v1.1 - Document',
        'File': filename,
        'Type': doc_type,
        'Description': description,
        'Confidence': 'medium',
        'ActionRequired': actions,
        'LastGenerated': datetime.datetime.utcnow().isoformat() + 'Z'
    }
    
    # Prepend as YAML frontmatter
    header = f"""---
{generate_yaml(metadata)}
---

"""
    
    # Remove existing header if present
    post = re.sub(r'^---\s*\n(.*?)\n---\s*\n?', '', content, flags=re.DOTALL)
    
    new_content = header + post.lstrip()
    with open(filepath, 'w') as f:
        f.write(new_content)
    
    return {'header': header, 'metadata': metadata}

def infer_document_type(content: str) -> str:
    if 'action items' in content.lower() or 'next steps' in content.lower():
        return 'meeting-notes'
    elif any(word in content.lower() for word in ['test', 'results', 'data', 'analysis']):
        return 'report'
    elif content.strip().startswith('#') and 'proposal' in content.lower():
        return 'proposal'
    return 'memo'
```

### 5.2 Example Processing

**Input meeting notes:**
```
# Product Sync 2/4/26
Printer safety spec still pending. Alice researching.
Need to update Q1 priorities based on velocity drop.
ACTION: Alice to deliver safety spec by Friday.
```

**After `generate_document_header()`:**
```
---
Contract-Header v1.1 - Document
File: product-sync-2026-02-04.md
Type: meeting-notes
Description: Product sync covering printer safety spec status and Q1 priorities
Inputs: Q1 priorities document
Outputs: Updated action items and deadlines
Confidence: medium
ActionRequired: 
  - Alice: deliver safety spec by 2026-02-06
LastGenerated: 2026-02-04T22:15:00Z
---

# Product Sync 2/4/26
...
```

***

## 6. LLM Agent Behavior for Documents

```
SYSTEM PROMPT TEMPLATE:

"When processing documents with Contract-Header v1.1 - Document:

1. READ THE HEADER FIRST. It tells you what's reliable vs speculative.
2. Trust 'high confidence' outputs as facts for downstream reasoning.
3. Skip documents missing required Inputs.
4. When chaining documents, prefer higher-confidence sources.
5. Extract ActionRequired items into your task queue automatically.
6. Flag 'medium/low confidence' outputs when summarizing to stakeholders.
7. For hardware/job documents, verify SafetyBoundaries exist before execution.
"
```

***

## 7. Chaining Documents (The Real Power)

Consider this workflow:

```
Doc A: Meeting Notes → outputs 3 priorities
    ↓
Doc B: Engineering Assessment → inputs A's priorities → outputs cost/timeline
    ↓  
Doc C: Executive Decision → inputs B's assessment → outputs approval + budget
```

Each document only needs to read its predecessor’s header:

```
Doc B sees: "Doc A outputs: prioritized feature list (confidence: high)"
→ Trusts the list, focuses analysis effort
Doc C sees: "Doc B outputs: cost/timeline (confidence: medium)"
→ Treats as directional, seeks confirmation
```

Token savings compound geometrically.

***

## 8. Edge Cases and Extensions

### 8.1 Email Threads
```
---
Contract-Header v1.1 - Document
File: printer-safety-thread.eml
Type: email-thread
Description: Cross-team discussion on printer safety interlock requirements
Inputs: Load test report, UL standards
Outputs: Consensus requirements document
Participants: Alice(eng), Bob(prod), Carol(safety)
ActionRequired: Alice to draft spec
---
```

### 8.2 Research Notes (Low Confidence)
```
---
... 
Confidence: 25% (early hypothesis stage)
Outputs: Tentative filament degradation model (needs validation)
---
```

### 8.3 Legal/Compliance Docs (High Confidence)
```
---
Confidence: 100% (legally binding)
Type: compliance-form
ActionRequired: Sign by 2026-02-10 or lose hardware access
---
```

***

## 9. Prosumer Hardware Integration

Your cosmically‑dangerous 3D printer job queue becomes self‑documenting:

```
---
Contract-Header v1.1 - Document
File: dragon-keycap-print-job.gcode
Type: hardware-job
Description: Print sculpted keycap using new safety envelope
Inputs: dragon-keycap.stl, redPLA filament loaded
Outputs: Printed keycap in tray A3
SafetyBoundaries: 
  - maxTemp: 245C
  - dutyCycle: 80%
  - area: "Hatch-A3"
Status: approved
---
G1 X10 Y10 Z0.2 F3000 ; start print...
```

The print controller reads 10 lines, confirms safety bounds match physical state, then executes. No deep parsing. No disasters.

***

## 10. Closing Observation

Documents were always contracts—they just never said so explicitly. Now they do, in a form both human and machine can audit instantly. The meeting notes that extract tasks. The report that surfaces one critical finding. The proposal that clarifies exactly what decision needs making.

In an AI‑augmented world, every document becomes a potential program. This header makes them legible to both silicon and carbon processors. The alternative—treating every page as an undifferentiated blob—isn't just inefficient. It's irresponsible.

**The machine age demands literate documents. This is how they learn to speak plainly.**