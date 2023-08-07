## /events/:eventId/contents

## Data Model

### EventContent

| JSON             | SQL                     | rw/ro | Notes                                        |
| ---------------- | ----------------------- | ----- | -------------------------------------------- |
| object_id        | images.image_id         | (ro)  |                                              |
| event_id         | images.event_id         | (ro)  | foreign key into events                      |
| src              | images.cdn_link         | (ro)  | Probably needs formatting with a scheme etc  |
| state            | images.status           | (ro)  | ENUM(visible/hidden) Image is visible or not |
| width            | images.width            | (ro)  |                                              |
| height           | images.height           | (ro)  |                                              |
| creator          | images.creator          | (ro)  |                                              |
| thumbnail        | images.thumbnail        | (ro)  |                                              |
| color            | images.color            | (ro)  | dominant color                               |
| last_update_date | images.last_update_date | (ro)  |                                              |
| create_date      | images.create_date      | (ro)  |                                              |
| image_key        | images.image_key        | (ro)  | Image Key is the unique ID for the image in  |
|                  |                         |       | the CDN system to access different sizes.    |
| timing_date      | images.timing_date      | (ro)  | The date the image was taken - from EXIF     |

## Notes

- Pagination required on GET response
- API should consider a calling user's tenant_id and only allow access to events owned by that tenant_id

## `GET /events/:eventId/contents`

Detailed event contents

### Request Params

| Param     | Values                     | Default | Description                                  |
| --------- | -------------------------- | ------- | -------------------------------------------- |
| start     | int > 0                    | 1       | Start at $start iteration                    |
| limit     | int > 0                    | 20      | Size of result set                           |
| state     | ENUM(hidden, visible, all) | all     | filter on $state                             |
| recent    | boolean                    | false   | Filter contents created in the last 24 hours |
| thumbnail | boolean                    | false   | Include preview b64 thumbnails               |
| order     | ENUM(asc, desc)            | desc    | Order of results by ingestion date           |

### Request Body

None

### Response Body

#### 200:OK

```
{
  "success" : true,
  "event" : {
    "event_id" : 1,
    "contents" : [
      {
        "object_id" : 123,
        "src" : "https://whatever",
        "state" : "visible", #visible/hidden
        "width" : 600,
        "height" : 600,
        "creator" : 1, #user-id,
        "create_date" : "2010-01-01 18:00:00",
        "last_updated_date" : "2010-01-01 18:00:00",
        "thumbnail" : "{BASE64}",
        "color" : "#RRGGBB",
        "image_key" : "129837_123_kjsdfhdksjhfjkdshfjk.jpg",
        "timing_date" : "2010-01-01 18:00:00"
      }
    ...
    ],
  },
  "meta" : {TX-ID}
}
```

#### 404: EventNotFound

```
{
  "success" : "false",
  "error" : "Event Not Found",
  "meta" : {TX-ID}
}
```

## `POST /events/:eventId/contents`

### Request Body

`multipart/form-data`

### Response Body

#### 200:OK

#### 401: EventNotAuthourized
```
{
  "success" : "false",
  "error" : "Event Not Authourized",
  "meta" : {TX-ID}
}
```

#### 404: EventNotFound

```
{
  "success" : "false",
  "error" : "Event Not Found",
  "meta" : {TX-ID}
}
```


## `DELETE /events/:eventId/contents/:objectId`

### Request Params

None

### Response

#### 200:OK

```
{
  "success" : true,
  "message" : "Object Removed",
  "meta" : {TX-ID}
}
```

### `POST /events/:eventId/contents/set-state`

#### Request Params

None

#### Request Body

```
{
  "objects" : [
    123,
    456
  ],
  "state" : "hidden" #hidden/visible
}
```

### Response Body

#### 200:OK

```
{
 "success" : true,
 "event" : {
    "event_id" : 1,
    "updated_contents" : [
      {
        "object_id" : 123,
        "src" : "https://whatever",
        "state" : "visible", #visible/hidden
        "width" : 600,
        "height" : 600,
        "creator" : 1, #user-id,
        "create_date" : "2010-01-01 18:00:00",
        "last_updated_date" : "2010-01-01 18:00:00",
        "thumbnail" : "{BASE64}",
        "color" : "#RRGGBB",
        "image_key" : "129837_123_kjsdfhdksjhfjkdshfjk.jpg",
        "timing_date" : "2010-01-01 18:00:00"
      }
    ...
    ]
  },
  "meta" : {TX-ID}
}
```

(EventContents as for /GET but no thumbnail or color detail)

#### 404: EventNotFound

```
{
  "success" : false,
  "error" : "Event Not Found",
  "meta" : {TX-ID}
}
```

### EventContentSummary

| JSON               | SQL                                                        | rw/ro | Notes                                             |
| ------------------ | ---------------------------------------------------------- | ----- | ------------------------------------------------- |
| event_id           | events.event_id                                            | (ro)  |                                                   |
| size               | events.bucket_size                                         | (ro)  | Max number of items (0 means unlimied)            |
| objects            | `COUNT(images.image_id) where event_id = X`                | (ro)  | Number of images in event                         |
| hidden             | `COUNT(images.image_id) where event_id = X and status = 0` | (ro)  | number of images in hidden state                  |
| visible            | `COUNT(images.image_id) where event_id = X and status = 1` | (ro)  | number of images in visible state                 |
| recent             | whatever mechanism is used to calculate what is recent     | (ro)  | number of "recent" images                         |
| last_activity_date | `MAX(images.last_update_date) where event_id = X`          | (ro)  | Datetime of last action on an image in this event |
| last_content_date  | `MAX(images.create_date) where event_id = X`               | (ro)  |                                                   |
| first_content_date | `MIN(images.create_date) where event_id = X`               | (ro)  |                                                   |
| first_content_date | `MIN(images.create_date) where event_id = X`               | (ro)  |                                                   |
| first_content_date | `MIN(images.create_date) where event_id = X`               | (ro)  |                                                   |
| first_content_date | `MIN(images.create_date) where event_id = X`               | (ro)  |                                                   |

## GET ## /events/:eventId/contents/summary

### Request Body

None

### Response Body

#### 200: EventContentSummary

```
{
  "success": true,
  "summary": {
    "event_id" : 26,
    "size" : 0,
    "objects" : 20,
    "hidden" : 1,
    "visible" : 19,
    "recent" : 5,
    "last_activity_date": "2018-08-03 05:50:33",
    "last_content_date": "2018-08-03 05:50:33",
    "first_content_date": "2018-08-01 02:50:33"
  },
  "meta" : {TX-ID}
}
```

#### 404: EventNotFound

```
{
  "success" : false,
  "error" : "Event Not Found",
  "meta" : {TX-ID}
}
```
unsolved issues now
1. before creating a new user, verify that email is unique, not existing in another tenant ✔
2. send verification code to the email address and provide link to reset password
7. visited home at the weekend
3. delete user from cognito - asked to jamie
6. 14100 successfully deliverd
4. after setting the owner role, how to handle the token? -asked jamie ✔
8. she was good state, said after trouble happiness comes
5. how to create public events? - asked to jamie