# TCPA Compliance Double Layer

For voice campaigns: even if your CRM (like HubSpot) handles Do-Not-Call filtering and quiet-time rules, implement a second compliance layer on your side.

## The Stakes

- Single non-compliant call: **$1,500 fine**
- Class action exposure: multiply by number of violations
- No "I thought HubSpot handled it" defense

## Recommended Double Layer

```python
def can_call(phone_number, timezone):
    # Layer 1: Your own DNC list
    if is_on_internal_dnc(phone_number):
        return False, "Internal DNC"
    
    # Layer 2: Quiet hours check (your logic)
    if is_quiet_hours(timezone):
        return False, "Quiet hours"
    
    # Layer 3: CRM verification (HubSpot/etc)
    if not crm.is_callable(phone_number):
        return False, "CRM DNC"
    
    return True, "OK"
```

## What to Check

1. **Your internal DNC list** - maintained from direct opt-outs
2. **National DNC registry** - if applicable
3. **Quiet hours** - typically 8am-9pm in recipient's timezone
4. **CRM status** - HubSpot/others as secondary check

## The Lesson

Don't trust external systems for compliance verification. HubSpot's filters might have gaps, delays, or configuration issues. Your liability means you need independent verification. Trust but verify - or in this case, verify and verify again.
