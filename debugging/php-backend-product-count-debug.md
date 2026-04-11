# PHP Backend Product Count Debug

When debugging why products don't appear in search or API responses, compare the actual product count returned by the backend against the expected count.

## The Debugging Process

1. **Establish baseline** - Find a working product and note its API response
2. **Compare counts** - Check if the API returns fewer products than expected
3. **Identify missing items** - Diff the lists to find gaps

## Real Example

```bash
# Working product query
curl "/api/products?type=mobile-office&size=20ft" | jq '.products | length'
# Returns: 19

# Problem query
curl "/api/products?type=mobile-office&size=40ft" | jq '.products | length'
# Returns: 18

# Expected: 19 products
# The 40ft mobile office is missing from the response
```

## The Fix

The backend query was excluding products with a specific flag that shouldn't have applied. Count comparison made the bug obvious.

## Debugging Template

```python
def debug_product_search(query, expected_count=None):
    response = api.search(query)
    actual_count = len(response['products'])
    
    print(f"Query: {query}")
    print(f"Returned: {actual_count} products")
    
    if expected_count and actual_count != expected_count:
        print(f"MISMATCH! Expected {expected_count}")
        # List missing product IDs
        returned_ids = {p['id'] for p in response['products']}
        expected_ids = get_expected_ids(query)
        print(f"Missing: {expected_ids - returned_ids}")
```

## The Lesson

Count discrepancies are the fastest way to spot data issues. When something's missing, the count won't match. Start there before diving into complex query debugging.
