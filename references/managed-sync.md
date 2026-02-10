# Managed Sync for ingesting & syncing 3rd-party integration data

## Managed Sync Overview
Managed Sync is a set of APIs for developers to spin up data pipelines that pull their 
users' 3rd-party integration data (like Salesforce CRM data, Google Drive file data, and ServiceNow tickets)
and keep that data up-to-date. These data pipelines live in Paragon's infrastructure 
and are completely managed by Paragon. Developer can easily start syncs, pause syncs, 
and pull synced data.

When integration data in synced, that data is also normalized across that integration's category. For example, the data schema from Google Drive, Dropbox, Box, and Sharepoint are all normalized so that code/logic can be used across that category.

For file storage integration data, the Permissions API is an API to check permissions 
of any synced file. Paragon manages a graph database behind the scenes, so developers 
can get and check up-to-date permissions without managing that data.

## Basic Usage of Managed Sync APIs

### Sync API
The Sync API is how developer can interact with managed data pipelines for any supported 3rd-party integration.

- The `Enable Sync` spins up a data pipeline to begin syncing your users' integration data

```bash
curl --request POST \
  --url https://sync.useparagon.com/api/syncs \
  --header 'Authorization: Bearer PARAGON_JWT' \
  --header 'Content-Type: application/json' \
  --data '
{
  "integration": "googledrive",
  "pipeline": "files",
  "configuration": {},
  "configurationName": "default"
}
'
```

Output will be in this format: 

```json
{
  "id": "a11347bd-d510-5489-9068-e5fded55a28c",
  "userId": "your-user-id",
  "integration": "googledrive",
  "pipeline": "files",
  "projectId": "1ea8024c-23a7-4885-b925-c50d0faf9318",
  "status": "INITIALIZING"
}
```

- The `Pull Synced Records` endpoint will return an array of synced objects from a sync pipeline

```bash
curl --request GET \
  --url https://sync.useparagon.com/api/syncs/{syncId}/records \
  --header 'Authorization: Bearer PARAGON_TOKEN'
```

The data will vary based off the integration and category of the synd pipeline

- The `Get Synced Record` endpoint gets the data of a specified synced object

```bash
curl --request GET \
  --url https://sync.useparagon.com/api/syncs/{syncId}/records/{recordId} \
  --header 'Authorization: Bearer PARAGON_TOKEN'
```

The data will vary based off the integration and category of the synd pipeline

- For file storage integrations, the `Download File Content` endpoint will return the binary data of a synced file

```bash
curl --request GET \
  --url https://sync.useparagon.com/api/syncs/{syncId}/records/{recordId}/content \
  --header 'Authorization: Bearer PARAGON_TOKEN'
```

### Permissions API
For file storage integrations (Google Drive, Sharepoint, Box, etc.), Paragon will manage a permissions 
graph database that developers can check against. This ensures that file data is only accessed by 
allowed parties.

- `Batch Check Access` to check multiple user-file relationships

```bash
 curl --request POST \
  --url https://sync.useparagon.com/api/permissions/{syncId}/batch-check \
  --header 'Authorization: Bearer PARAGON_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '
{
  "checks": [
    {
      "object": "a657df3b-17e2-5989-bc5f-13ddb7fdab41",
      "user": "user:email@example.com",
      "role": "can_read"
    }
  ]
}
'
```

This endpoint will return data in the following format:

```json
{
  "result": [
    {
      "allowed": true,
      "request": {
        "object": "a657df3b-17e2-5989-bc5f-13ddb7fdab41",
        "user": "user:email@example.com",
        "role": "can_read"
      },
      "error": "<string>"
    }
  ]
}
```

- `List Users` to list all user IDs that have a certain relationship with a file

```bash
curl --request POST \
  --url https://sync.useparagon.com/api/permissions/{syncId}/list-users \
  --header 'Authorization: Bearer PARAGON_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '
{
  "object": "a657df3b-17e2-5989-bc5f-13ddb7fdab41",
  "role": "can_read"
}
'
```

This will return data in the following format:

```json
{
  "users": [
    "email@example.com"
  ]
}
```

- `List Objects` to list all object IDs that has user has a certain relationship with

```bash
curl --request POST \
  --url https://sync.useparagon.com/api/permissions/{syncId}/list-objects \
  --header 'Authorization: Bearer PARAGON_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '
{
  "objectType": "file",
  "user": "email@example.com",
  "role": "can_read"
}
'
```

This will return data in the following format:

```json
{
  "objects": [
    "a657df3b-17e2-5989-bc5f-13ddb7fdab41"
  ]
}
```

## Additional Sync Management APIs
For additional API endpoints to list, disable, and delete sync pipelines, see the guidance on [sync-management](sync-management.md)
