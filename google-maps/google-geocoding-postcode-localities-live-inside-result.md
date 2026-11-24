# Google Geocoding `postcode_localities` Lives Inside Each Result

A Google Geocoding response has a `results` array. The optional `postcode_localities` field belongs to a result, not to the top-level response. Reading it from the response root silently misses multi-locality postal codes.

```python
response = {
    "results": [
        {
            "formatted_address": "Example postal area",
            "postcode_localities": ["Northville", "Southville"],
        }
    ],
    "status": "OK",
}

result = response["results"][0] if response.get("results") else {}
localities = result.get("postcode_localities") or []
```

Use the result selected by your geocoding logic rather than assuming the first result always matches. This field is optional and appears for postal codes with multiple localities. Put the field inside the result in test fixtures too: a mock with the same wrong nesting as the implementation can make a dead fallback appear covered.
