# Sync Management APIs
Use these additional endpoints if a developer needs to interact with the lifecycle of a sync pipeline

## Overview
Available endpoints to manage a sync pipeline:
- `LIST SYNCS`
- `GET SYNC STATUS`
- `DISABLE SYNC`
- `RE-ENABLE SYNC`
- `DELETE SYNC`

## Endpoints
- `LIST SYNCS` to list all sync pipelines for an end-user

```bash
curl --request GET \
  --url https://sync.useparagon.com/api/syncs \
  --header 'Authorization: Bearer PARAGON_JWT'
```

Returned data:

```json
{
  "data": [
    {
      "id": "<string>",
      "integration": "<string>",
      "pipeline": "<string>",
      "status": "INITIALIZING",
      "configurationName": "<string>",
      "dateCreated": "2023-11-07T05:31:56Z",
      "dateUpdated": "2023-11-07T05:31:56Z"
    }
  ]
}
```

- `GET SYNC STATUS` to get the status of a specific sync pipeline

```bash
curl --request GET \
  --url https://sync.useparagon.com/api/syncs/{syncId} \
  --header 'Authorization: Bearer PARAGON_JWT'
```

Returned data:
```json
{
  "status": "INITIALIZING",
  "summary": {
    "totalRecords": 123,
    "syncedRecordsCount": 123,
    "lastSyncedAt": "2023-11-07T05:31:56Z",
    "latestCursor": "<string>"
  }
}
```

- `DISABLE SYNC` to disable a specified sync pipeline

```bash
curl --request POST \
  --url https://sync.useparagon.com/api/syncs/{syncId}/disable \
  --header 'Authorization: Bearer PARAGON_JWT'
```

- `RE-ENABLE SYNC` to re-enable a disabled sync pipeline

```bash
curl --request POST \
  --url https://sync.useparagon.com/api/syncs/{syncId}/reenable \
  --header 'Authorization: Bearer PARAGON_JWT'
```

- `DELETE SYNC` to delete a specified sync pipeline

```bash
curl --request DELETE \
  --url https://sync.useparagon.com/api/syncs/{syncId} \
  --header 'Authorization: Bearer PARAGON_JWT'
```
