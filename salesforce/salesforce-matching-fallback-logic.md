# Salesforce Matching Fallback Logic

When matching calendar meetings to Salesforce contacts, the primary key is email. When that's absent, name matching can work as a fallback - but expect lower accuracy.

## Root Cause Analysis

In one audit, roughly 75% of meeting-contact matching issues were caused by Salesforce calendar events missing either:
- The "Name" field, or
- The "Who" field

Without these, there's nothing to match against.

## Data Synchronization Gotcha

When contacts are reassigned to new owners (counselors, reps, etc.), both systems may need updating:

1. **Salesforce** - the primary source of truth
2. **HubSpot** - if used as a secondary CRM or enrichment layer

If only one system is updated, the matching logic will break silently for reassigned contacts.

## Fallback Priority

```python
def match_contact(meeting):
    # Primary: email match (most reliable)
    if meeting.email:
        return find_by_email(meeting.email)
    
    # Fallback: name match (less reliable)
    if meeting.name:
        return find_by_name(meeting.name)
    
    # No match possible - flag for manual review
    return None
```

## The Lesson

Don't assume data completeness. Log match rates and investigate the "unmatched" bucket - it often reveals systemic data quality issues.
