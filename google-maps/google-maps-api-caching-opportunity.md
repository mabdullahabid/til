# Google Maps API Caching Opportunity

ZIP code areas are small and don't change. Caching distance lookups by ZIP code can reduce Google Maps API costs by 60-70%.

## The Problem

One system was making fresh Google Maps API calls on every request, even when calculating distances between the same ZIP codes repeatedly. With Google's pricing model, this added up quickly.

## The Solution

Cache lookups by origin/destination ZIP pairs:

```python
def get_distance(from_zip, to_zip):
    cache_key = f"{from_zip}:{to_zip}"
    
    # Check cache first
    cached = redis.get(cache_key)
    if cached:
        return json.loads(cached)
    
    # Fetch from Google Maps API
    result = google_maps.distance_matrix(from_zip, to_zip)
    
    # Cache indefinitely - ZIP boundaries rarely change
    redis.setex(cache_key, timedelta(days=365), json.dumps(result))
    
    return result
```

## Additional Fix

A geocoding bug was discovered: ZIP code 75500 (Pakistan) was appearing instead of US ZIP codes due to incorrect geocoding parameters. Always validate geocoding results match the expected country.

```python
def validate_us_zip(result):
    country = result['address_components'][-1]['short_name']
    return country == 'US'
```

## The Lesson

ZIP codes are effectively static data. Treating them as dynamic API calls is a waste of money. Cache aggressively - the data won't change from under you.
