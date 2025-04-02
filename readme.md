# Open Positive Response Standard

This standard describes the REST API implementation for 811 center positive response.



# Response API

**Endpoint**
The endpoint should be called "response", and accept one response objects in the body of any POST.

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

**API Responses**

## HTTP 404
This should be returned if the required infomration (ticketNumber, memberCode, facilityList, action) does not exist or is incorrect.  Any other issue (incorrect code or facilities, for instance) should return a 409 instead.

## HTTP 409
Any validation issues aside from an invalid ticket, should be communicated back with a 409 status code, with a json return document explaining the validation issues

```
{
  "validation":["invalid facility type","invalid action", "comments exceed allowable length"]
}
```
## HTTP 201
If the response is accepted but there is additional information that needs to be communicated, such as not accepting enhanced positive response, this should return a 201 (Created) with the information in the body of the response.  Utilizing this code allows the center to record a required response even if the sender adds information the center does not yet accept on their system.

```
{
  "validation":["Response accepted, but references are not supported and have been discarded"]
}
```

## HTTP 204
If the response is accepted in its entirety, then an HTTP 204 can be returned.  No document should be returned with this.
