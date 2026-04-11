# Content Safety Transcript Dependency

Content safety analysis has a hard dependency on transcripts. If meetings aren't transcribed, content safety checks cannot run.

## The Numbers

In one audit:
- 88% of meetings (22,000 out of 26,000) had no transcripts
- This meant zero content safety coverage for those meetings
- Not a model issue - a data pipeline issue

## Why Transcripts Matter

Content safety models analyze text. They can't process:
- Audio recordings directly
- Video files
- Meeting metadata

They need transcripts as input.

## The Pipeline

```
Meeting Recording
       ↓
Transcription Service (Zoom, etc.)
       ↓
Transcript Storage
       ↓
Content Safety Analysis
       ↓
Flagged Content Review
```

If any step fails, everything downstream stops.

## Root Cause

In this case, Zoom Pro's auto-transcription wasn't enabled system-wide. The feature was included in the plan, just not turned on.

## The Lesson

A "content safety system" is only as good as its data inputs. Measure coverage by transcript availability, not by model accuracy. The best safety model in the world can't analyze what it can't read.
