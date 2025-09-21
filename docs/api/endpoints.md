# Pocket Casts Android API endpoints

This document summarises every network endpoint the Android client integrates with, grouped by the base URL that the Retrofit or OkHttp clients use. The production host names come from the Gradle build configuration and `Settings` constants, while the same paths are used on the debug `.net` hosts during development.【F:build.gradle.kts†L215-L353】【F:modules/services/preferences/src/main/java/au/com/shiftyjelly/pocketcasts/preferences/Settings.kt†L37-L49】

## Base URLs

| Purpose | Base URL |
| --- | --- |
| Sync, account, subscriptions, ratings, files | `https://api.pocketcasts.com` |
| Legacy form posts, OPML import, update polling, search/export helpers | `https://refresh.pocketcasts.com` |
| Podcast metadata cache (podcast details, episode search, show notes) | `https://cache.pocketcasts.com` |
| Static CDN assets (discover metadata, promotions) | `https://static.pocketcasts.com` |
| Curated list downloads | `https://lists.pocketcasts.com` |
| Curated list uploads/sharing | `https://sharing.pocketcasts.com` |
| Anonymous bump statistics | `https://public-api.wordpress.com` |
| Transcript fetches | Dynamic `https://…` transcript URLs |

The following sections document each endpoint, its intent, and a concise example of the request and expected response.

## Sync & account API (`https://api.pocketcasts.com`)

### Authentication & profile management

#### `POST /user/login_pocket_casts`
**Purpose:** Exchange email/password credentials for sync tokens.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L53-L58】

```http
POST https://api.pocketcasts.com/user/login_pocket_casts
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "secret",
  "scope": "mobile"
}
```
```json
{
  "email": "user@example.com",
  "uuid": "user-123",
  "isNew": false,
  "accessToken": "ACCESS_TOKEN",
  "refreshToken": "REFRESH_TOKEN",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

#### `POST /user/login_google`
**Purpose:** Authenticate with a Google ID token and receive sync tokens.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L57-L58】

```http
POST https://api.pocketcasts.com/user/login_google
Content-Type: application/json

{
  "id_token": "google-oauth-token",
  "scope": "mobile"
}
```
```json
{
  "email": "user@example.com",
  "uuid": "user-123",
  "isNew": false,
  "accessToken": "ACCESS_TOKEN",
  "refreshToken": "REFRESH_TOKEN",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

#### `POST /user/token`
**Purpose:** Refresh an access token using a stored refresh token.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L60-L61】

```http
POST https://api.pocketcasts.com/user/token
Content-Type: application/json

{
  "grant_type": "refresh_token",
  "refresh_token": "REFRESH_TOKEN",
  "scope": "mobile"
}
```
```json
{
  "email": "user@example.com",
  "uuid": "user-123",
  "isNew": false,
  "accessToken": "NEW_ACCESS_TOKEN",
  "refreshToken": "REFRESH_TOKEN",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

#### `POST /user/register_pocket_casts`
**Purpose:** Create a new Pocket Casts account with email/password credentials.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L63-L64】

```http
POST https://api.pocketcasts.com/user/register_pocket_casts
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "secret",
  "scope": "mobile"
}
```
```json
{
  "email": "user@example.com",
  "uuid": "user-123",
  "isNew": true,
  "accessToken": "ACCESS_TOKEN",
  "refreshToken": "REFRESH_TOKEN",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

#### `POST /user/forgot_password`
**Purpose:** Trigger a password reset email.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L66-L67】

```http
POST https://api.pocketcasts.com/user/forgot_password
Content-Type: application/json

{
  "email": "user@example.com"
}
```
```json
{
  "success": true,
  "message": "Password reset email sent"
}
```

#### `POST /user/exchange_sonos`
**Purpose:** Exchange a Sonos auth code for Pocket Casts tokens after Sonos login.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L69-L70】

```http
POST https://api.pocketcasts.com/user/exchange_sonos
Authorization: Bearer ACCESS_TOKEN
```
```json
{
  "token": "ACCESS_TOKEN",
  "uuid": "user-123",
  "email": "user@example.com"
}
```

#### `POST /user/change_email`
**Purpose:** Update the account email for the authenticated user.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L72-L73】

```http
POST https://api.pocketcasts.com/user/change_email
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "email": "new@example.com",
  "password": "current-password",
  "scope": "mobile"
}
```
```json
{
  "success": true,
  "message": "Email updated"
}
```

#### `POST /user/delete_account`
**Purpose:** Delete the signed-in user account.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L75-L76】

```http
POST https://api.pocketcasts.com/user/delete_account
Authorization: Bearer ACCESS_TOKEN
```
```json
{
  "success": true,
  "message": "Account deleted"
}
```

#### `POST /user/update_password`
**Purpose:** Change the account password while authenticated.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L78-L79】

```http
POST https://api.pocketcasts.com/user/update_password
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "new_password": "newSecret",
  "old_password": "current-password",
  "scope": "mobile"
}
```
```json
{
  "email": "user@example.com",
  "uuid": "user-123",
  "isNew": false,
  "accessToken": "UPDATED_ACCESS_TOKEN",
  "refreshToken": "REFRESH_TOKEN",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

#### `POST /user/named_settings/update`
**Purpose:** Persist per-user named settings like skip duration or marketing opt-in flags.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L81-L82】

```http
POST https://api.pocketcasts.com/user/named_settings/update
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "m": "Android",
  "v": 1,
  "settings": {
    "skipForward": 45,
    "skipBack": 15,
    "marketingOptIn": true
  }
}
```
```json
{
  "skipForward": { "value": 45, "changed": true },
  "skipBack": { "value": 15, "changed": false },
  "marketingOptIn": { "value": true, "changed": true }
}
```

### Library & sync operations

#### `POST /sync/update`
**Purpose:** Legacy form-encoded sync update for older clients.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L84-L86】

```http
POST https://api.pocketcasts.com/sync/update
Content-Type: application/x-www-form-urlencoded

device=android&lastSync=1714800000&changes=%5B...%5D
```
```json
{
  "success": true,
  "lastModified": 1714800500,
  "records": []
}
```

#### `POST /user/sync/update`
**Purpose:** Binary Protocol Buffers sync endpoint for podcasts, episodes, filters, folders, and bookmarks.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L88-L90】

```http
POST https://api.pocketcasts.com/user/sync/update
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/octet-stream

<Protocol Buffers SyncUpdateRequest bytes>
```
```json
{
  "last_modified": 1714800500,
  "records": [
    { "podcast": { "uuid": "pod-123", "subscribed": true } }
  ]
}
```

#### `POST /up_next/sync`
**Purpose:** Synchronise the Up Next queue between devices.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L92-L93】

```http
POST https://api.pocketcasts.com/up_next/sync
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "deviceTime": 1714800000,
  "version": "1",
  "upNext": {
    "serverModified": 1714799900,
    "changes": [
      {
        "action": 1,
        "modified": 1714800000,
        "uuid": "ep-123",
        "title": "Latest episode",
        "podcast": "pod-123"
      }
    ]
  }
}
```
```json
{
  "serverModified": 1714800100,
  "changes": []
}
```

#### `POST /user/last_sync_at`
**Purpose:** Record or fetch the last sync timestamp for the account (available as RxJava or suspend version).【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L95-L100】

```http
POST https://api.pocketcasts.com/user/last_sync_at
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "m": "mobile",
  "v": 2
}
```
```json
{
  "lastSyncAt": "2024-05-04T10:00:00Z"
}
```

#### `POST /user/podcast/episodes`
**Purpose:** Fetch all episodes for a given podcast UUID from the sync store.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L101-L103】

```http
POST https://api.pocketcasts.com/user/podcast/episodes
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "uuid": "pod-123"
}
```
```json
{
  "episodes": [
    {
      "uuid": "ep-123",
      "url": "https://cdn.example.com/ep-123.mp3",
      "duration": 1800,
      "title": "Episode title",
      "podcast_uuid": "pod-123"
    }
  ]
}
```

#### `POST /user/podcast/list`
**Purpose:** Return the user’s subscribed podcasts and folders as a Protocol Buffers payload.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L104-L107】

```http
POST https://api.pocketcasts.com/user/podcast/list
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/octet-stream

{
  "v": "2",
  "m": "mobile"
}
```
```json
{
  "podcasts": [
    {
      "uuid": "pod-123",
      "episodes_sort_order": 1,
      "folder_uuid": "folder-1"
    }
  ],
  "folders": [
    {
      "folder_uuid": "folder-1",
      "name": "News",
      "color": 5
    }
  ]
}
```

#### `POST /user/playlist/list`
**Purpose:** Retrieve saved filters/playlists either as a minimal JSON list (`BasicRequest`) or the full Protocol Buffers payload (`UserPlaylistListRequest`).【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L108-L114】

```http
POST https://api.pocketcasts.com/user/playlist/list
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "m": "mobile",
  "v": 2
}
```
```json
{
  "playlists": [
    {
      "uuid": "filter-123",
      "title": "Unplayed",
      "unplayed": true,
      "manual": false
    }
  ]
}
```

#### `POST /user/episodes`
**Purpose:** Fetch multiple episodes by UUID across podcasts, using a Protocol Buffers request body.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L115-L117】

```http
POST https://api.pocketcasts.com/user/episodes
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/octet-stream

{
  "podcast_uuids": ["pod-123"],
  "episode_uuids": ["ep-123", "ep-456"]
}
```
```json
{
  "total": 2,
  "episodes": [
    { "uuid": "ep-123", "podcast_uuid": "pod-123", "title": "Episode 123" },
    { "uuid": "ep-456", "podcast_uuid": "pod-123", "title": "Episode 456" }
  ]
}
```

### Listening history

#### `POST /history/sync`
**Purpose:** Bidirectional sync for listening history events.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L119-L120】

```http
POST https://api.pocketcasts.com/history/sync
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "deviceTime": 1714800000,
  "serverModified": 1714799900,
  "version": 1,
  "changes": [
    {
      "action": 1,
      "episode": "ep-123",
      "podcast": "pod-123",
      "modifiedAt": "2024-05-04T09:58:00Z"
    }
  ]
}
```
```json
{
  "serverModified": 1714800100,
  "lastCleared": 0,
  "changes": []
}
```

#### `POST /history/year`
**Purpose:** Fetch aggregated listening history for a given year, optionally including totals.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L122-L123】

```http
POST https://api.pocketcasts.com/history/year
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "version": "1",
  "count": true,
  "year": 2023
}
```
```json
{
  "count": 120,
  "history": {
    "serverModified": 1714800100,
    "lastCleared": 0,
    "changes": []
  }
}
```

#### `POST /sync/update_episode`
**Purpose:** Push a single episode progress update (position, status).【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L125-L126】

```http
POST https://api.pocketcasts.com/sync/update_episode
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "uuid": "ep-123",
  "podcast": "pod-123",
  "position": 600,
  "duration": 1800,
  "status": 2
}
```
```json
{
  "success": true
}
```

### Subscription management

#### `GET /subscription/status`
**Purpose:** Retrieve the current subscription state for the account.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L128-L129】

```http
GET https://api.pocketcasts.com/subscription/status
Authorization: Bearer ACCESS_TOKEN
```
```json
{
  "autoRenewing": true,
  "expiryDate": "2024-12-01T00:00:00Z",
  "paid": 1,
  "platform": 2,
  "frequency": 2,
  "tier": "plus",
  "subscriptions": [
    {
      "type": 1,
      "tier": "plus",
      "platform": 2,
      "frequency": 2,
      "expiryDate": "2024-12-01T00:00:00Z",
      "autoRenewing": true,
      "giftDays": 0
    }
  ],
  "index": 0
}
```

#### `POST /subscription/purchase/android`
**Purpose:** Confirm an Android in-app purchase token and refresh subscription status.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L131-L132】

```http
POST https://api.pocketcasts.com/subscription/purchase/android
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "purchaseToken": "PLAY_STORE_TOKEN",
  "sku": "plus_yearly"
}
```
```json
{
  "autoRenewing": true,
  "expiryDate": "2025-05-04T00:00:00Z",
  "paid": 1,
  "platform": 2,
  "frequency": 2,
  "tier": "plus",
  "subscriptions": [...],
  "index": 0
}
```

### Cloud files & personal uploads

#### `GET /files`
**Purpose:** List all user-uploaded files and storage usage.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L134-L135】

```http
GET https://api.pocketcasts.com/files
Authorization: Bearer ACCESS_TOKEN
```
```json
{
  "files": [
    {
      "uuid": "file-123",
      "title": "My upload",
      "contentType": "audio/mpeg",
      "duration": 900,
      "size": 10485760,
      "imageUrl": "https://cdn.pocketcasts.com/files/file-123.jpg"
    }
  ],
  "account": {
    "totalFiles": 500,
    "totalSize": 32212254720,
    "usedSize": 10485760
  }
}
```

#### `POST /files`
**Purpose:** Update metadata for uploaded files (title, colour, playback position, etc.).【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L137-L138】

```http
POST https://api.pocketcasts.com/files
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "files": [
    {
      "uuid": "file-123",
      "title": "My upload",
      "colour": 3,
      "playedUpTo": 120,
      "playingStatus": 2,
      "duration": 900,
      "hasCustomImage": true
    }
  ]
}
```
```json
{}
```

#### `POST /files/upload/request`
**Purpose:** Request a pre-signed upload URL for a new audio file.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L140-L141】

```http
POST https://api.pocketcasts.com/files/upload/request
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "uuid": "file-123",
  "title": "My upload",
  "colour": 3,
  "contentType": "audio/mpeg",
  "duration": 900
}
```
```json
{
  "uuid": "file-123",
  "url": "https://uploads.pocketcasts.com/file-123?signature=..."
}
```

#### `POST /files/upload/image`
**Purpose:** Request a pre-signed URL for uploading custom artwork for a file.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L143-L144】

```http
POST https://api.pocketcasts.com/files/upload/image
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "uuid": "file-123",
  "size": 204800,
  "contentType": "image/jpeg"
}
```
```json
{
  "url": "https://uploads.pocketcasts.com/file-123.jpg?signature=..."
}
```

#### `PUT {upload URL}`
**Purpose:** Upload binary audio or artwork data to the pre-signed URL provided by the previous request.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L146-L150】

```http
PUT https://uploads.pocketcasts.com/file-123?signature=...
Content-Type: audio/mpeg

<binary audio bytes>
```
```http
HTTP/1.1 200 OK
```

#### `GET /files/upload/status/{uuid}`
**Purpose:** Poll the status of an in-progress upload.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L152-L153】

```http
GET https://api.pocketcasts.com/files/upload/status/file-123
Authorization: Bearer ACCESS_TOKEN
```
```json
{
  "success": true
}
```

#### `DELETE /files/{uuid}`
**Purpose:** Delete a user-uploaded audio file.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L155-L156】

```http
DELETE https://api.pocketcasts.com/files/file-123
Authorization: Bearer ACCESS_TOKEN
```
```http
HTTP/1.1 204 No Content
```

#### `DELETE /files/image/{uuid}`
**Purpose:** Remove custom artwork for an uploaded file.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L158-L159】

```http
DELETE https://api.pocketcasts.com/files/image/file-123
Authorization: Bearer ACCESS_TOKEN
```
```http
HTTP/1.1 204 No Content
```

#### `GET /files/{uuid}`
**Purpose:** Fetch metadata for a single uploaded file.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L161-L162】

```http
GET https://api.pocketcasts.com/files/file-123
Authorization: Bearer ACCESS_TOKEN
```
```json
{
  "uuid": "file-123",
  "title": "My upload",
  "contentType": "audio/mpeg",
  "duration": 900,
  "imageUrl": "https://cdn.pocketcasts.com/files/file-123.jpg"
}
```

#### `POST /user/stats/summary`
**Purpose:** Retrieve listening statistics for the authenticated device (time listened, silence removed, etc.).【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L164-L165】

```http
POST https://api.pocketcasts.com/user/stats/summary
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "deviceId": "device-123",
  "deviceType": 2
}
```
```json
{
  "time_listened": 1234567,
  "time_silence_removal": 23456,
  "time_variable_speed": 12345
}
```

#### `GET /files/usage`
**Purpose:** Show file storage usage quotas for the account.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L167-L168】

```http
GET https://api.pocketcasts.com/files/usage
Authorization: Bearer ACCESS_TOKEN
```
```json
{
  "totalFiles": 500,
  "totalSize": 32212254720,
  "usedSize": 10485760
}
```

### Promotions, ratings, bookmarks & referrals

#### `POST /subscription/promo/redeem`
**Purpose:** Apply a Pocket Casts Plus promo code to the authenticated user.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L170-L171】

```http
POST https://api.pocketcasts.com/subscription/promo/redeem
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json

{
  "code": "FREEPLUS"
}
```
```json
{
  "code": "FREEPLUS",
  "description": "3 months of Pocket Casts Plus",
  "starts_at": "2024-05-04T00:00:00Z",
  "ends_at": "2024-08-04T00:00:00Z"
}
```

#### `POST /subscription/promo/validate`
**Purpose:** Validate a promo code without redeeming it (no auth required).【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L173-L174】

```http
POST https://api.pocketcasts.com/subscription/promo/validate
Content-Type: application/json

{
  "code": "FREEPLUS"
}
```
```json
{
  "code": "FREEPLUS",
  "description": "3 months of Pocket Casts Plus",
  "starts_at": "2024-05-04T00:00:00Z",
  "ends_at": "2024-08-04T00:00:00Z"
}
```

#### `POST /user/bookmark/list`
**Purpose:** Fetch the user’s bookmarks using the binary sync format.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L176-L178】

```http
POST https://api.pocketcasts.com/user/bookmark/list
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/octet-stream

{
  "podcast_uuid": "pod-123",
  "episode_uuid": "ep-123",
  "time": 600
}
```
```json
{
  "bookmarks": [
    {
      "bookmark_uuid": "bookmark-1",
      "podcast_uuid": "pod-123",
      "episode_uuid": "ep-123",
      "time": 600,
      "title": "Great quote",
      "createdAt": "2024-05-04T10:00:00Z"
    }
  ]
}
```

#### `POST /user/podcast_rating/add`
**Purpose:** Submit or update the user’s rating for a podcast.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L180-L182】

```http
POST https://api.pocketcasts.com/user/podcast_rating/add
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/octet-stream

{
  "podcast_uuid": "pod-123",
  "podcast_rating": 5
}
```
```json
{
  "podcast_uuid": "pod-123",
  "podcast_rating": 5,
  "modified_at": "2024-05-04T10:00:00Z"
}
```

#### `POST /user/podcast_rating/show`
**Purpose:** Retrieve the user’s existing rating for a podcast.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L184-L186】

```http
POST https://api.pocketcasts.com/user/podcast_rating/show
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/octet-stream

{
  "podcast_uuid": "pod-123"
}
```
```json
{
  "podcast_uuid": "pod-123",
  "podcast_rating": 5,
  "modified_at": "2024-05-04T10:00:00Z"
}
```

#### `GET /user/podcast_rating/list`
**Purpose:** List all podcasts the user has rated.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L188-L190】

```http
GET https://api.pocketcasts.com/user/podcast_rating/list
Authorization: Bearer ACCESS_TOKEN
Accept: application/octet-stream
```
```json
{
  "podcast_ratings": [
    {
      "podcast_uuid": "pod-123",
      "podcast_rating": 5,
      "modified_at": "2024-05-04T10:00:00Z"
    }
  ]
}
```

### Feedback & support

#### `POST /anonymous/feedback`
**Purpose:** Send anonymous feedback without authentication (content-type still octet-stream due to protobuf envelope).【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L192-L194】

```http
POST https://api.pocketcasts.com/anonymous/feedback
Content-Type: application/octet-stream

{
  "message": "Great app!",
  "email": "",
  "subject": "Feedback",
  "debug": "",
  "inbox": ""
}
```
```http
HTTP/1.1 204 No Content
```

#### `POST /support/feedback`
**Purpose:** Submit support feedback with authentication so it links to the account.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L196-L198】

```http
POST https://api.pocketcasts.com/support/feedback
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/octet-stream

{
  "message": "I found a bug",
  "email": "user@example.com",
  "subject": "Bug report",
  "debug": "logs...",
  "inbox": "support"
}
```
```http
HTTP/1.1 204 No Content
```

### Referrals

#### `GET /referrals/code`
**Purpose:** Fetch the user’s referral code and landing URL.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L200-L204】

```http
GET https://api.pocketcasts.com/referrals/code
Authorization: Bearer ACCESS_TOKEN
Accept: application/octet-stream
```
```json
{
  "code": "FRIEND123",
  "url": "https://pocketcasts.com/referral/FRIEND123"
}
```

#### `GET /referrals/winback_offers?platform=android`
**Purpose:** Retrieve personalised win-back offers for lapsed subscribers.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L205-L207】

```http
GET https://api.pocketcasts.com/referrals/winback_offers?platform=android
Authorization: Bearer ACCESS_TOKEN
Accept: application/octet-stream
```
```json
{
  "offer": "50% off",
  "platform": "android",
  "details": "Come back to Plus",
  "code": "WINBACK50"
}
```

#### `GET /referrals/validate`
**Purpose:** Validate a referral code supplied by another user.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L208-L211】

```http
GET https://api.pocketcasts.com/referrals/validate?code=FRIEND123
Authorization: Bearer ACCESS_TOKEN
Accept: application/octet-stream
```
```json
{
  "offer": "Free month",
  "platform": 2,
  "details": "Redeem for Android",
  "code": "FRIEND123"
}
```

#### `POST /referrals/redeem`
**Purpose:** Redeem a referral code for the authenticated account.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/sync/SyncService.kt†L213-L215】

```http
POST https://api.pocketcasts.com/referrals/redeem
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/octet-stream

{
  "code": "FRIEND123"
}
```
```json
{
  "code": "FRIEND123"
}
```

## Main server helpers (`https://refresh.pocketcasts.com`)

These endpoints use form posts via `ServiceManager` and Retrofit for OPML import and background refresh jobs.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/ServiceManager.kt†L44-L166】【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/refresh/RefreshService.kt†L10-L20】

#### `POST /podcasts/search`
**Purpose:** Perform the server-side podcast catalogue search used in the Discover tab.

```http
POST https://refresh.pocketcasts.com/podcasts/search
Content-Type: application/x-www-form-urlencoded

q=android%20news&v=1.7&av=7.60&ac=1234&dt=2&c=US&l=en&m=Pixel%208&scope=mobile
```
```json
{
  "status": "ok",
  "result": {
    "is_url": false,
    "search_results": [
      { "uuid": "pod-123", "title": "Android News", "author": "Team Pocket Casts" }
    ]
  }
}
```

#### `POST /import/export_feed_urls`
**Purpose:** Return RSS feed URLs for the supplied podcast UUIDs, allowing OPML export.

```http
POST https://refresh.pocketcasts.com/import/export_feed_urls
Content-Type: application/x-www-form-urlencoded

uuids=pod-123,pod-456&v=1.7&av=7.60&ac=1234&dt=2&c=US&l=en&m=Pixel%208&scope=mobile
```
```json
{
  "status": "ok",
  "result": {
    "pod-123": "https://example.com/feed1.xml",
    "pod-456": "https://example.com/feed2.xml"
  }
}
```

#### `POST /user/update`
**Purpose:** Legacy bulk refresh endpoint used to batch refresh podcast feeds from the main server.

```http
POST https://refresh.pocketcasts.com/user/update
Content-Type: application/x-www-form-urlencoded

uuid=pod-123&uuid=pod-456&v=1.7&av=7.60&ac=1234&dt=2&c=US&l=en&m=Pixel%208&scope=mobile
```
```json
{
  "status": "ok",
  "result": {
    "pod-123": ["ep-999", "ep-1000"],
    "pod-456": []
  }
}
```

#### `POST import/opml`
**Purpose:** Import podcast subscriptions from an OPML file or poll an existing import job.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/refresh/RefreshService.kt†L12-L13】

```http
POST https://refresh.pocketcasts.com/import/opml
Content-Type: application/json

{
  "urls": [
    "https://example.com/feed1.xml",
    "https://example.com/feed2.xml"
  ],
  "device": "device-123",
  "datetime": "2024-05-04T10:00:00Z",
  "v": "1.7",
  "av": "7.60",
  "ac": "1234",
  "dt": "2",
  "c": "US",
  "l": "en",
  "m": "Pixel 8"
}
```
```json
{
  "status": "ok",
  "result": {
    "uuids": ["pod-123", "pod-456"],
    "poll_uuids": ["poll-abc"],
    "failed": 0
  }
}
```

#### `GET api/v1/update_podcast`
**Purpose:** Request a fresh crawl of a specific podcast and optionally include the last episode UUID to de-duplicate.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/refresh/RefreshService.kt†L15-L16】

```http
GET https://refresh.pocketcasts.com/api/v1/update_podcast?podcast_uuid=pod-123&last_episode_uuid=ep-999
```
```http
HTTP/1.1 202 Accepted
```

#### `GET {poll URL}`
**Purpose:** Poll the URL returned by the update service to determine when the crawl has finished.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/refresh/RefreshService.kt†L18-L19】

```http
GET https://refresh.pocketcasts.com/api/v1/update_podcast/poll/poll-abc
```
```http
HTTP/1.1 204 No Content
```

#### `POST {share path}`
**Purpose:** Retrieve metadata for shared podcast, episode, or list links by posting to the stripped share path returned from a URL (e.g. `/share/episode/xyz`).【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/ServiceManager.kt†L114-L125】

```http
POST https://refresh.pocketcasts.com/share/episode/ep-123
Content-Type: application/x-www-form-urlencoded

v=1.7&av=7.60&ac=1234&dt=2&c=US&l=en&m=Pixel%208&scope=mobile
```
```json
{
  "status": "ok",
  "result": {
    "type": "episode",
    "uuid": "ep-123",
    "title": "Great episode",
    "podcast_uuid": "pod-123"
  }
}
```

## Podcast cache API (`https://cache.pocketcasts.com`)

All podcast metadata and search features use the cache service.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/podcast/PodcastCacheService.kt†L74-L121】

#### `GET /mobile/podcast/full/{podcastUuid}`
**Purpose:** Fetch the full podcast metadata plus all cached episodes.

```http
GET https://cache.pocketcasts.com/mobile/podcast/full/pod-123
```
```json
{
  "podcast": { "uuid": "pod-123", "title": "Android News" },
  "episodes": [
    { "uuid": "ep-123", "title": "Latest episode", "duration": 1800 }
  ]
}
```

#### `GET /mobile/show_notes/full/{podcastUuid}`
**Purpose:** Retrieve the redirect target for episode show notes (and optionally force no redirect).

```http
GET https://cache.pocketcasts.com/mobile/show_notes/full/pod-123?disableredirect=true
```
```json
{
  "url": "https://cdn.pocketcasts.com/show-notes/pod-123/ep-123.html"
}
```

#### `GET {show notes URL}`
**Purpose:** Download the HTML show notes or chapters payload at the redirected URL.

```http
GET https://cdn.pocketcasts.com/show-notes/pod-123/ep-123.html
```
```json
{
  "content": "<p>Episode summary…</p>"
}
```

#### `GET /mobile/podcast/findbyepisode/{podcastUuid}/{episodeUuid}`
**Purpose:** Resolve a specific episode and its parent podcast in one call.

```http
GET https://cache.pocketcasts.com/mobile/podcast/findbyepisode/pod-123/ep-123
```
```json
{
  "podcast": { "uuid": "pod-123", "title": "Android News" },
  "episode": { "uuid": "ep-123", "title": "Latest episode" }
}
```

#### `GET /mobile/episode/url/{podcastUuid}/{episodeUuid}`
**Purpose:** Return the stream/download URL for an episode.

```http
GET https://cache.pocketcasts.com/mobile/episode/url/pod-123/ep-123
```
```http
HTTP/1.1 302 Found
Location: https://cdn.pocketcasts.com/audio/pod-123/ep-123.mp3
```

#### `POST /mobile/podcast/episode/search`
**Purpose:** Search within a podcast for episodes by term.

```http
POST https://cache.pocketcasts.com/mobile/podcast/episode/search
Content-Type: application/json

{
  "podcastuuid": "pod-123",
  "searchterm": "Android"
}
```
```json
{
  "episodes": [ { "uuid": "ep-123" } ]
}
```

#### `POST /episode/search`
**Purpose:** Global episode search across all podcasts.

```http
POST https://cache.pocketcasts.com/episode/search
Content-Type: application/json

{
  "term": "Pixel"
}
```
```json
{
  "episodes": [
    {
      "uuid": "ep-123",
      "title": "Pixel review",
      "podcast_uuid": "pod-123",
      "podcast_title": "Android News"
    }
  ]
}
```

#### `GET /podcast/rating/{podcastUuid}`
**Purpose:** Fetch the community rating summary for a podcast (with optional no-cache variant).

```http
GET https://cache.pocketcasts.com/podcast/rating/pod-123
```
```json
{
  "average": 4.8,
  "total": 1200
}
```

#### `POST /podcast/suggest_folders`
**Purpose:** Request suggested folders for multiple podcast UUIDs.

```http
POST https://cache.pocketcasts.com/podcast/suggest_folders
Content-Type: application/json

{
  "uuids": ["pod-123", "pod-456"]
}
```
```json
{
  "pod-123": ["Technology", "News"],
  "pod-456": ["Storytelling"]
}
```

## Discover & list services

### Static CDN (`https://static.pocketcasts.com`)

Endpoints delivering cached discover assets.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/cdn/StaticService.kt†L7-L16】

#### `GET /discover/images/metadata/{podcastUuid}.json`
**Purpose:** Fetch cached dominant artwork colours for UI theming.

```http
GET https://static.pocketcasts.com/discover/images/metadata/pod-123.json
```
```json
{
  "primary": "#FF6F00",
  "secondary": "#FFC107"
}
```

#### `GET /discover/blaze/promotions.json`
**Purpose:** Retrieve marketing promotions displayed in the Discover tab.

```http
GET https://static.pocketcasts.com/discover/blaze/promotions.json
```
```json
{
  "promotions": [
    {
      "id": "spring-sale",
      "title": "Spring Plus offer",
      "imageUrl": "https://static.pocketcasts.com/promotions/spring.png"
    }
  ]
}
```

### Curated lists (`https://lists.pocketcasts.com`)

#### `GET /{listId}.json`
**Purpose:** Download a curated list of podcasts for sharing or discover collections.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/list/ListDownloadService.kt†L6-L9】

```http
GET https://lists.pocketcasts.com/top-tech.json
```
```json
{
  "title": "Top Tech",
  "podcasts": [
    { "uuid": "pod-123", "title": "Android News" }
  ]
}
```

### List sharing (`https://sharing.pocketcasts.com`)

#### `POST /share/list`
**Purpose:** Upload a user-created list for sharing with others.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/list/ListUploadService.kt†L7-L10】

```http
POST https://sharing.pocketcasts.com/share/list
Content-Type: application/json

{
  "title": "Favourite Android podcasts",
  "podcasts": [
    { "uuid": "pod-123" },
    { "uuid": "pod-456" }
  ]
}
```
```json
{
  "status": "ok",
  "result": {
    "share_url": "https://pca.st/list/abcd",
    "share_code": "abcd"
  }
}
```

### Discover web service (dynamic URLs)

The discover Retrofit service loads JSON feeds from various URLs resolved at runtime (for example the recommendations feed).【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/server/ListWebService.kt†L11-L22】

#### `GET /discover/{platform}/content_v{version}.json`
**Purpose:** Retrieve the home Discover layout for the specified platform and schema version.

```http
GET https://static.pocketcasts.com/discover/android/content_v6.json
```
```json
{
  "rows": [
    {
      "title": "Trending",
      "items": [
        { "uuid": "pod-123", "title": "Android News" }
      ]
    }
  ]
}
```

#### `GET {discover list URL}`
**Purpose:** Load a specific curated list or recommendation feed by absolute URL (with optional auth header for personalised feeds).

```http
GET https://api.pocketcasts.com/recommendations/podcast/pod-123?country=us
Authorization: Bearer ACCESS_TOKEN
```
```json
{
  "title": "Because you listen to Android News",
  "podcasts": [
    { "uuid": "pod-789", "title": "Mobile Dev Weekly" }
  ]
}
```

#### `GET {categories URL}`
**Purpose:** Fetch the list of discover categories from a supplied URL.

```http
GET https://static.pocketcasts.com/discover/categories.json
```
```json
[
  { "id": "technology", "title": "Technology" },
  { "id": "news", "title": "News" }
]
```

## Transcript fetches (dynamic URLs)

#### `GET {transcript URL}`
**Purpose:** Download transcripts for episodes using the direct CDN URL supplied in metadata, optionally forcing cache behaviour.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/podcast/TranscriptService.kt†L9-L14】

```http
GET https://transcripts.pocketcasts.com/pod-123/ep-123.vtt
Cache-Control: only-if-cached
```
```http
HTTP/1.1 200 OK
Content-Type: text/vtt

WEBVTT

00:00:00.000 --> 00:00:05.000
Welcome to the show…
```

## Anonymous bump statistics (`https://public-api.wordpress.com`)

#### `POST /rest/v1.1/tracks/record`
**Purpose:** Send anonymised usage telemetry (“bump stats”) to Automattic’s analytics service.【F:modules/services/servers/src/main/java/au/com/shiftyjelly/pocketcasts/servers/bumpstats/WpComService.kt†L7-L10】

```http
POST https://public-api.wordpress.com/rest/v1.1/tracks/record
Content-Type: application/json

{
  "events": [
    {
      "_en": "pcandroid_play_start_bump",
      "_ts": 1714800000000,
      "_ui": "ANONYMOUS",
      "_ut": "anon",
      "podcast_uuid": "pod-123"
    }
  ],
  "commonProps": {
    "_lg": "en_US",
    "_rt": 1714800000000,
    "_via_ua": "Pocket Casts Android"
  }
}
```
```json
"OK"
```
