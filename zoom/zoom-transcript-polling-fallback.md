# Zoom Transcript Polling Fallback

When Zoom auto-transcription isn't enabled system-wide, you can implement a polling fallback that checks the last day's meetings every 15 minutes for missing transcripts.

However, this only works if transcription was generated at meeting time. Zoom retains recordings for approximately one month, but it cannot retroactively transcribe old audio.

## Key Insight

In one system audit, roughly 88% of meetings (22,000 out of 26,000) had no transcripts because Zoom Pro's auto-transcription wasn't turned on. The feature is included in most plans at no extra cost - it just needs to be enabled.

## Implementation Pattern

```python
# Check for missing transcripts from the past day
# Run this every 15 minutes via cron or scheduled job

def poll_for_missing_transcripts():
    yesterday = datetime.now() - timedelta(days=1)
    meetings = get_meetings_since(yesterday)
    
    for meeting in meetings:
        if not meeting.has_transcript:
            transcript = fetch_zoom_transcript(meeting.id)
            if transcript:
                save_transcript(meeting.id, transcript)
```

## The Lesson

Before building complex fallback systems, check if the root cause is simply a disabled feature. Polling can't fix what was never captured in the first place.
