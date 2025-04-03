# Open Positive Response Standard

This standard describes the REST API implementation for 811 center positive response.  This API should enforce strict schema validation.  This means that if any required information is missing, or if additional information not defined in this specification is sent, the API will return a failure response.  This is important to ensure public safety.  We don't want to make assumptions.

However, there is some optional information in this specification.  If optional information is not included, this will not be considered an error, and the submitted positive response will be processed normally.


# Response API

**Endpoint**
The endpoint should be called "/response", and accept one response objects in the body of any POST.

**Method**
The POST method should be used to add responses.  This is the standard method for creating new content.
```
POST [...]/response
```

**Body**
The body of the POST is a JSON document containing the Positive Response information.
```
{
      "ticketNumber": "200131-001002",
      "memberCode": "XYZ01",
      "facilityList": [
            "Water"
      ],
      "action": "MARKED",
      "comment": "Comment for this action",
      "session:":"456654812",
      "attachmentList":[
		  {
		      "name":"Satellite",
		      "mimeType":"image/png",		      
		      "url":"https://media.georgia811.com/cSqMy1+jAN"
		  },
		  {
		      "name":"Project Plan",
		      "mimeType":"application/pdf",
		      "value":"NTk1OVowgcoxHTAbBgNVBA8MFFByaXZhdGUgT3JnYW5pemF0aW9uMRMwEQYLKwYB"
		  }
      ],
      "geometry":{
            "wkt":"LINESTRING(...)",
            "geoJson":""
}
```
# Required Fields
The required fields for positive response are:
- ticketNumber
- memberCode
- facilityList
- action

All other fields are optional.

**API Responses**
# HTTP Response JSON Documents
When an HTTP response should return additional information, the API should return the information in JSON documents in the body of the HTTP response.  These should include a "status" field, and optionally a "messageList" string array containing additional messages, as required by the appropriate response below.

## HTTP 201 Created
If the response is accepted completely, the API should return a 201 (Created) with the status of "success" in the HTTP response.

```
{
  "status":"success"
}
```

## HTTP 202 Accepted
If the response is accepted but there is additional information that needs to be communicated, such as not accepting enhanced positive response, this should return a 202 (Accepted).  The body of the return json document should indicate the reason that it is not a 201.  For instance, if a file attachment is sent, but the center does not accept file attachments, this should be indicated in the response.  Utilizing the 202 response instead of the 201 response indicates that a portion of the received information has not been applied to the positive response.
```
{
  "status":"success",
  "messageList":[
    "Response accepted, but file attachments are not supported by this center.  File attachments have been discarded."
  ]
}
```

## HTTP 400 Bad Request
This should only be returned if the JSON sent is malformed or invalid.  For instance, if required fields are not included, or unknown fields are included (for instance, misspellings or additional information not defined in this document are sent).

As a best practice, if there is a misspelled or other unknown field, return the name of the field back to the sender for quicker troubleshooting.
```
{
  "status":"failed",
  "messageList":[
    "Unknown field ticketnum"
  ]
}
```
OR
```
{
  "status":"failed",
  "messageList":[
    "malformed document"
  ]
}
```

## HTTP 409 Conflict
Any validation issues aside from an invalid ticket, should be communicated back with a 409 status code, with a json return document explaining the validation issues.  This includes rejection of duplicate postive responses.  This is considered an error because the same response has been recorded at an earlier date. Accepting a duplicate response later could potentially change the meaning of the response (for instance, if it comes in after the due date).  Therefore, any duplicates should result in an error back to the sender that can then be diagnosed.

As a best practice, ALL validation errors should be returned.  If there are multiple errors, this allows the sender to correct all validation errors before attempting to send corrected information.
```
{
  "status":"invalid",
  "messageList":[
    "invalid action",
    "comments exceed allowable length"
  ]
}
```

## HTTP 422 Unprocessable Entity
This code should be returned if the information provided is valid, but the information provided (ticketNumber, memberCode, facilityList, action) does not refer to a respondable entity.  For instance, if all the values ar valid, but the provided facilityList doesn't match the facilityList for that member on the ticket (such as "Water" instead of "Sewer").

As a best practice, all issues with the required information should be returned to the sender.
```
{
  "status":"unprocessable",
  "messageList":[
    "memberCode does not exist on the indicated ticket",
    "facilityList is not valid for this memberCode on this ticket"
  ]
}
```
 
# Batch Response API
Some API's rely on batching from the sender's side only.  For 811 tickets, we prefer to provide more ways for members to comply with the process as possible, so a batch endpoint should be provided.  However, there may be more work on the sender's side than batching themselves.  For instance, in the batch API, if there are any issues with the batch, the sender will need to parse the returned information for each positive response sent to find out what succeeded and what failed.  However, this can still be more efficient, especially for large members, by eliminating the overhead of creating multiple HTTP connections.
 
 **Endpoint**
 The endpoint should be called "/response/batch", and accept one or more response objects in the body of any POST.
 
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
 
 ## HTTP 200 OK
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
                   "result":"accepted",
                   "messageList":[]
             },
             {
                   "ticket": "200131-001002",
                   "code": "XYZ02",
                   "facilities": [
                         "Sewer"
                   ],
                   "action": "MARKED",
                   "result": "duplicate",
                   "messageList": [
	              "Duplicate responses are not allowed"
                    ]
             }
       ]
 }
 ```

## HTTP 207 Multi-Status
If there is a mix of success and failures in the batch, return a 207, and include in the body the status of each positive response.

## HTTP 400 Bad Request
This should only be returned if the JSON sent is malformed or invalid.  For instance, if required fields are not included, or unknown fields are included (for instance, misspellings or additional information not defined in this document are sent).

As a best practice, if there is a misspelled or other unknown field, return the name of the field back to the sender for quicker troubleshooting.
```
{
  "status":"failed",
  "messageList":[
    "Unknown field ticketnum"
  ]
}
```
OR
```
{
  "status":"failed",
  "messageList":[
    "malformed document"
  ]
}
```
