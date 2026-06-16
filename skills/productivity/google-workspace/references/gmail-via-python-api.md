# Gmail via Google Python API (when gws is unavailable)

If `gws` CLI is not installed, use the Google API Python client directly via `execute_code`.

## Prerequisites

The google-auth and google-api-python-client packages are already installed on the host.

The OAuth token is at `~/.hermes/google_token.json` (created by the google-workspace setup script).

## Pattern

```python
from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build

creds = Credentials.from_authorized_user_file("~/.hermes/google_token.json")

# refresh if expired
if creds.expired and creds.refresh_token:
    creds.refresh(Request())

service = build("gmail", "v1", credentials=creds)

# Search
results = service.users().messages().list(userId="me", q="query", maxResults=10).execute()

# Read metadata (headers only)
msg = service.users().messages().get(
    userId="me", id=msg_id, format="metadata",
    metadataHeaders=["From","Subject","Date"]
).execute()

# Read full body
msg = service.users().messages().get(userId="me", id=msg_id, format="full").execute()
```

## Available APIs (same token)

```python
calendar_service = build("calendar", "v3", credentials=creds)
drive_service = build("drive", "v3", credentials=creds)
sheets_service = build("sheets", "v4", credentials=creds)
docs_service = build("docs", "v1", credentials=creds)
people_service = build("people", "v1", credentials=creds)
```

## Token Location

Token path: `~/.hermes/google_token.json` — auto-refreshes. Save back after refresh.