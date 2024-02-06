# Open Positive Response Standard

This standard describes the REST API implementation for 811 center positive response.



# Single Response API

**Endpoint**
The endpoint should be called "response", and accept one response objects in the body of any POST.

**Method**
The POST method should be used to add responses.
```
POST [...]/response
```

**Body**
```
{
      "ticket": "200131-001002",
      "code": "XYZ01",
      "facilities": [
            "Water"
      ],
      "action": "MARKED",
      "comment": "Comment for this action",
      "session:":"456654812",
      "references":[
            {
                  "name": "Locate Photos",
                  "value":"https://locator.org/ref?48941231487"
            },
            {
                  "name": "Additional Comments",
                  "value":"https://locator.org/ref?48941231488"
            }
      ],
      "locatedWkt:"LINESTRING(...)"
}
```
**Response**

## HTTP 404
This should be returned if the ticket does not exist.  Any other issue (incorrect code or facilities, for instance) should return a 409 instead.

## HTTP 409
Any validation issues aside from an invalid ticket, should be communicated back with a 409 error, with a document explaining the validation issues

```
{
  "validation":["invalid facility type","invalid action", "comments exceed allowable length"]
}
```
## HTTP 201
If the response is accepted but there is additional information that needs to be communicated, such as not accepting enhanced positive response, this should return a 201 (Created) with the information in the body of the response.

```
{
  "validation":["Response accepted, but references are not supported and have been discarded"]
}
```

## HTTP 204
If the response is accepted in its entirety, then an HTTP 204 can be returned.  No document should be returned with this.




# Batch Response API

**Endpoint**
The endpoint should be called "response/batch", and accept one or more response objects in the body of any POST.

**Method**
The POST method should be used to add responses.
```
POST [...]/response/batch
```

**Body**
```
{
      "responses": [
            {
                  "ticket": "200131-001002",
                  "code": "XYZ01",
                  "facilities": [
                        "Water"
                  ],
                  "action": "MARKED",
                  "comment": "Comment for this action",
                  "session:":"456654812",
                  "references":[
                        {
                              "name": "Locate Photos",
                              "value":"https://locator.org/ref?48941231487"
                        },
                        {
                              "name": "Additional Comments",
                              "value":"https://locator.org/ref?48941231488"
                        }
                  ],
                  "locatedWkt:"LINESTRING(...)"
            },
            {
                  "ticket": "200131-001002",
                  "code": "XYZ02",
                  "facilities": [
                        "Sewer"
                  ],
                  "action": "MARKED",
                  "comment": "More comments",
                  "session:":"456654813",
                  "references":[
                        {
                              "name": "Locate Photos",
                              "value":"https://locator.org/ref?48941231489"
                        },
                        {
                              "name": "Additional Comments",
                              "value":"https://locator.org/ref?48941231490"
                        }
                  ],
                  "locatedWkt":"LINESTRING(...)"
            }
      ]
}
```

**Response**
Normally, we could use 404, 409, 201, and 204 to communicate back to the POSTer about individual responses as is done in the single response API.  To do so with a batch API is inefficient because any single issue could then prevent the entire
batch from succeeding.  It is more efficient to accept everything that is good, and send only the rejections back to the sender.  This allows good responses to be posted to the appropriate tickets while the sending system deals with the invalid
responses.

## HTTP 200
The POST to /response should always return a confirmation to the submitter for each response.  Therefore a 200 (OK) response should be returned, rather than a 204 (No Content)
```
{
 "responses": [
            {
                  "ticket": "200131-001002",
                  "code": "XYZ01",
                  "facilities": [
                        "Water"
                  ],
                  "action": "MARKED",
                  "result":"accepted"
                  "validation":["compliant"]
            },
            {
                  "ticket": "200131-001002",
                  "code": "XYZ02",
                  "facilities": [
                        "Sewer"
                  ],
                  "action": "MARKED",
                  "result": "rejected",
                  "validation": ["Duplicate responses are not allowed"]
            }
      ]
}
```
