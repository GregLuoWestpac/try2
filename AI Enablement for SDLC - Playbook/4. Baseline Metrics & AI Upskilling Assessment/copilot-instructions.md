# Copilot Coding Instructions

## 1. Problem Understanding Assessment

Before making any changes, assess your understanding of the request:

<problem_assessment>
Rate your confidence in understanding the user's request on a scale of 0-10:
- **0-5**: Stop immediately. Ask clarifying questions before proceeding.
- **6-8**: Pause and ask targeted clarifying questions to fill knowledge gaps.
- **9-10**: Proceed with implementation.

Consider these factors when assessing:
- Is the scope clearly defined?
- Do I understand the expected behavior/outcome?
- Are there ambiguous requirements that could lead to incorrect implementation?
- Do I have sufficient context about the affected components?
- Are there edge cases or constraints I should confirm?

**CRITICAL**: Do NOT make any edits until this assessment is complete and confidence is above 8, or clarifying questions have been answered.
</problem_assessment>

---

## 2. Defect Root Cause Classification Criteria

When analyzing defects, use the **"Bug Source & Cause"** field as the high-level root cause category. Classify defects based on the following criteria organized by priority:

### Priority Levels (from XML `<priority>` field)

The priority of a defect is determined by the `<priority>` field in the Jira XML export:

| Priority ID | Priority Name | Description |
|-------------|---------------|-------------|
| `id="1"` | **Critical** | Blocks payments, data corruption, complete feature failure - Immediate resolution required |
| `id="2"` | **High** | Major functionality broken, significant user impact |
| `id="3"` | **Medium** | User experience degraded but workaround exists |
| `id="4"` | **Low** | Cosmetic issues, minor inconvenience |

---

### API Contract Mismatches
Schema mismatches between EXP API and downstream services.

| Indicator | Examples | Action Required |
|-----------|----------|-----------------|  
| "Unrecognized json property" errors | `"Unrecognized json property: 'divisionName' in request body"` | Immediate API contract sync required |
| "Invalid json request" errors | `"code": 1002, "message": "Invalid json request"` | Review API schema changes |
| Field mismatch errors | Missing required fields, unexpected properties | Update Pact contracts and coordinate with API team |
| 400 Bad Request from downstream | Payload structure rejected by downstream | Verify request/response contracts |

**Keywords to identify**: `unrecognized json`, `invalid json`, `api contract`, `schema mismatch`, `400 Bad Request`, `field not recognized`

### Downstream Dependency Issues
Issues caused by external APIs or backend services.

| Indicator | Examples | Action Required |
|-----------|----------|-----------------|
| Mentions of downstream APIs | Entitlement API, Payment API, Accounts API, CAP API, INF API | Escalate to downstream team |
| Downstream error responses | `"DownstreamException"`, `"Downstream Error"`, 5xx from downstream | Investigate API health and data mapping |
| Missing downstream data | "Downstream api return nothing", "missing fields" | Data availability or mapping issue |
| Downstream timeout/unavailable | Connection timeout, service unavailable | Check downstream service health |

**Keywords to identify**: `downstream`, `Entitlement API`, `Payment API`, `Accounts API`, `INF API`, `CAP API`, `DownstreamException`

### Experience API (BFF) Issues
Issues in the Backend-for-Frontend layer that transform data between UI and downstream services.

| Indicator | Examples |
|-----------|----------|
| Data mapping errors | Field mapping mismatches, missing transformations |
| Payload construction issues | Incorrect request payloads to downstream |
| Response transformation bugs | Missing fields in UI responses |

**Keywords to identify**: `EXP API`, `Experience API`, `BFF`, `transformation`

### Requirements and Features Issues
Defects arising from requirement gaps or changes.

| Bug Source & Cause | Sub-Category | Description |
|--------------------|--------------|-------------|
| **Requirements and Features** | Requirement Changes | Feature not matching updated requirements |
| **Requirements and Features** | (unspecified) | Gap between implementation and requirements |

### Frontend UI Issues
Visual and interaction issues affecting user experience.

| Category | Examples | Dependency |
|----------|----------|------------|
| **Styling Issues** | Alignment problems, CSS conflicts, spacing issues | Internal fix |
| **Accessibility** | Missing focus states, screen reader issues, keyboard navigation | Compliance requirement |
| **Responsive/Zoom Issues** | Layout breaks at zoom levels, responsive design bugs | CSS fixes |
| **Icon/Image Issues** | Incorrect sizes, missing icons, misaligned images | Asset or CSS fix |
| **Typography** | Typos, incorrect text, font issues | Content/CSS fix |

**Keywords to identify**: `alignment`, `styling`, `CSS`, `accessibility`, `focus state`, `zoom`, `icon`, `font`, `typo`

### Design Issues
User experience and interface design problems.

| Bug Source & Cause | Sub-Category | Description |
|--------------------|--------------|-------------|
| **Design** | Customer Experience/GUI | UI/UX not matching design specifications |
| **Design** | (unspecified) | General design implementation issues |

### Build & Deployment Issues
Issues related to CI/CD and configuration.

| Bug Source & Cause | Sub-Category | Description |
|--------------------|--------------|-------------|
| **Build & Deployment** | Business/Application Configuration | Configuration errors |
| **Build & Deployment** | (unspecified) | Build pipeline or deployment issues |

### Environment Issues
Test environment specific problems.

| Bug Source & Cause | Description |
|--------------------|-------------|
| **Environment** | Issues specific to INT, E2E, or other test environments |

### Test Data Issues
Issues caused by missing, incorrect, or insufficient test data.

| Bug Source & Cause | Description |
|--------------------|-------------|
| **Test Data** | Missing test data, incorrect data setup, data not matching expected scenarios |

**Keywords to identify**: `test data`, `data setup`, `missing data`, `test account`, `test user`

### External Dependencies (Blocked/Deferred)

Defects requiring coordination with external teams. Track separately for release planning.

#### GEL-Next Dependency
UI component issues dependent on GEL library updates.

| Indicator | Examples | Action |
|-----------|----------|--------|
| `[GEL NEXT DEPENDENCY]` in title | Date picker focus state, icon sizing | Link to GEL-Next GitHub issue |
| `GEL-Next` label | Component rendering bugs | Monitor GEL release schedule |
| GEL component bugs at zoom levels | Misaligned indicators, border issues | Defer until GEL fix available |

**Tracking**: Create GitHub issue at `WestpacGEL/GEL-next` and link in Jira ticket.

#### W1 Team Dependency
OpenFin/widget integration issues owned by W1 team.

| Indicator | Examples | Action |
|-----------|----------|--------|
| `W1-Defect` label | CSS conflicts, tab configurations | Assign to W1 team |
| OpenFin/widget CSS override | Tailwind conflicts, missing classes | Request W1 to scope their styles |
| Container/tab configuration | Tab names, window behavior | W1 configuration change |

#### 10x Core Banking Platform Dependency
Issues in the 10x core banking platform requiring fix from 10x/UK team.

| Indicator | Examples | Action |
|-----------|----------|--------|
| `10x` mentioned as root cause | GL posting issues, account events, product switch bugs | Escalate to 10x/UK team |
| 10x platform bugs | Duplicate subscription keys, incorrect event data | Track separately for release planning |
| 10x data/event issues | Missing events, incorrect mappings from 10x | Coordinate with 10x support |
| ATLAS/GCM 10x integration | Data sync issues, feed problems | Verify 10x-side configuration |

**Keywords to identify**: `10x`, `10x platform`, `UK team`, `core banking`, `10x bug`, `10x fix`

**Tracking**: Escalate to 10x support team and track resolution timeline separately.

#### Downstream API Team Dependency
Backend API issues requiring downstream team involvement.

| Indicator | Examples | Action |
|-----------|----------|--------|
| API contract changes | Schema updates, new fields | Coordinate Pact contract update |
| Missing API functionality | Sorting, filtering not working | Raise with downstream team |
| Data quality issues | Incorrect data returned | Escalate to data/API team |

---

### Quick Classification Checklist

When triaging a defect:

1. **Check Bug Source & Cause field** - Primary classification
2. **Scan for keywords** in title and description:
   - `unrecognized json`, `invalid json` → **API Contract Mismatch (Critical)**
   - API names, `DownstreamException` → **Downstream Dependency (Critical)**
   - `10x`, `10x platform`, `UK team` → **10x External Dependency (track separately)**
   - `alignment`, `styling`, `GEL` → Frontend UI (Medium)
   - `mapping`, `payload`, `EXP` → Experience API (High)
3. **Check labels** for additional context:
   - `GEL-Next` → External dependency (track separately)
   - `W1-Defect` → W1 team ownership
   - `Accessibility_defect` → Compliance priority
4. **Review comments** for root cause analysis from developers
5. **Priority** is determined by the XML `<priority>` field:
   - `id="1"` → **Critical**: Blocks payments, data corruption, complete feature failure
   - `id="2"` → **High**: Major functionality broken, significant user impact
   - `id="3"` → **Medium**: User experience degraded but workaround exists
   - `id="4"` → **Low**: Cosmetic issues, minor inconvenience