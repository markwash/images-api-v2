##Images API v2.0

###Versioning

**Two-part versioning scheme**

The v2 API will follow the lead of the v1 API and use a major version and a minor version. For example, 'v2.3' would break down to major version 2 and minor version 3


**Backwards-compatibility**

The interface will not be reduced with subsequent minor version releases, it will only be expanded. For example, everything in v2.1 will be available in v2.2.


###HTTP Response Status Codes

The following HTTP status codes are all valid responses:

* 200 - generic successful response, expect a body
* 201 - entity created, expect a body and a Location header
* 204 - successful response without body
* 301 - redirection
* 400 - invalid request (syntax, value, etc)
* 401 - unauthenticated client
* 403 - authenticated client unable to perform action
* 409 - that action is impossible due to some (possibly permanent) circumstance

###Authentication and Authorization

This spec does not govern how one might authenticate or authorize clients of the v2 Images API. Implementors are free to decide how to identify clients and what authorization rules to apply.

Note that the HTTP status codes 401 and 403 are included in this specification as valid response codes.

###Request/Response Content Format

The v2 Images API primarily accepts and serves JSON-encoded data. In certain cases it also accepts and serves binary image data. Requests that send JSON-encoded data must have the proper media type in their Content-Type header: 'application/json'. Requests that upload image data should use the media type 'application/octet-stream'.

Each call only responds in one format, so clients should not worry about sending an Accept header. It will be ignored. Assume a response will be formatted as 'application/json' unless otherwise stated in this spec.

###Image Entities

An image entity is represented by a JSON-encoded data structure and its raw binary data. 

An image entity has an identifier (ID) that is guaranteed to be unique within the endpoint to which it belongs. The ID is used as a token in request URIs to interact with that specific image.

An image is always guaranteed to have the following attributes: id, status, visibility, protected, tags, created_at, file and self. The other attribtues defined in the `image` schema below are guaranteed to be defined, but will only be returned with an image entity if they have been explicitly set.

A client may set arbitrarily-named attributes on their images if the `image` json-schema allows it. These user-defined attributes will appear like any other image attributes. See [documentation](http://tools.ietf.org/html/draft-zyp-json-schema-03#section-5.4) of the additionalProperties json-schema attribute.

###JSON Schemas

The necessary [json-schema](http://tools.ietf.org/html/draft-zyp-json-schema-03) documents will be provided at predictable URIs. A consumer should be able to validate server responses and client requests based on the published schemas. The schemas contained in this document are only examples and should not be used to validate your requests. A client should **always** fetch schemas from the server.


###Metadata API

####Get Images Schema

**GET /v2/schemas/images**

Request body ignored. 

Response body will contain a json-schema document representing an `images` entity (a container of `image` entities). For example:

    {
        "name": "images",
        "properties": {
            "images": {
                "items": {
                    "type": "array",
                    "name": "image"
                    "properties": {
                        "id": {"type": "string"}, 
                        "name": {"type": "string"}, 
                        "visibility": {"enum": ["public", "private"]},
                        "status": {"type": "string"},
                        "protected": {"type": "boolean"},
                        "tags": {
                            "type": "array",
                            "items": {"type": "string"}
                        },
                        "checksum": {"type": "string"}, 
                        "size": {"type": "integer"}, 
                        "created_at": {"type": "string"}, 
                        "updated_at": {"type": "string"}, 
                        "file": {"type": "string"}, 
                        "self": {"type": "string"}, 
                        "schema": {"type": "string"}
                    },
                    "additionalProperties": {"type": "string"}, 
                    "links": [
                        {"href": "{self}", "rel": "self"}, 
                        {"href": "{file}", "rel": "enclosure"}, 
                        {"href": "{schema}", "rel": "describedby"}
                    ]
                },
            }, 
            "schema": {"type": "string"}, 
            "next": {"type": "string"}, 
            "first": {"type": "string"}
        },
        "links": [
            {"href": "{first}", "rel": "first"}, 
            {"href": "{next}", "rel": "next"}, 
            {"href": "{schema}", "rel": "describedby"}
        ]
    }

####Get Image Schema

**GET /v2/schemas/image**

Request body ignored. 

Response body will contain a json-schema document representing an `image`. For example:

    {
        "name": "image",
        "properties": {
            "id": {"type": "string"}, 
            "name": {"type": "string"}, 
            "visibility": {"enum": ["public", "private"]},
            "status": {"type": "string"},
            "protected": {"type": "boolean"},
            "tags": {
                "type": "array",
                "items": {"type": "string"}
            },
            "checksum": {"type": "string"}, 
            "size": {"type": "integer"}, 
            "created_at": {"type": "string"}, 
            "updated_at": {"type": "string"}, 
            "file": {"type": "string"}, 
            "self": {"type": "string"}, 
        	"schema": {"type": "string"}
        },
        "additionalProperties": {"type": "string"}, 
        "links": [
            {"href": "{self}", "rel": "self"}, 
            {"href": "{file}", "rel": "enclosure"}, 
        	{"href": "{schema}", "rel": "describedby"}
    	]
	}


####Create an Image

**POST /v2/images**

Request body must be JSON-encoded and conform to the `image` JSON schema. For example:

    {
        "id": "e7db3b45-8db7-47ad-8109-3fb55c2c24fd",
        "name": "Ubuntu 12.10",
        "tags": ["ubuntu", "quantal"]
    }

Successful HTTP response will be 201 Created with a Location header containing the newly-created URI for the image. Response body will represent the created `image` entity. For example:

    {
        "id": "e7db3b45-8db7-47ad-8109-3fb55c2c24fd",
        "name": "Ubuntu 12.10",
        "status": "queued",
        "visibility": "public",
        "tags": ["ubuntu", "quantal"],
        "created_at": "2012-08-11T17:15:52Z", 
        "updated_at": "2012-08-11T17:15:52Z", 
        "self": "/v2/images/e7db3b45-8db7-47ad-8109-3fb55c2c24fd", 
        "file": "/v2/images/e7db3b45-8db7-47ad-8109-3fb55c2c24fd/file", 
        "schema": "/v2/schemas/image"
    }


####Update an Image

**PUT /v2/images/\<IMAGE_ID\>**

Request body must be JSON-encoded and conform to the `image` JSON schema. Using **PUT /v2/images/e7db3b45-8db7-47ad-8109-3fb55c2c24fd** as an example:

    {
        "name": "Fedora 17",
        "tags": ["fedora", "beefy"]
    }

Response body will represent the updated `image` entity. For example:

    {
        "id": "e7db3b45-8db7-47ad-8109-3fb55c2c24fd",
        "name": "Fedora 17",
        "status": "queued",
        "visibility": "public",
        "tags": ["fedora", "beefy"],
        "created_at": "2012-08-11T17:15:52Z", 
        "updated_at": "2012-08-11T17:15:52Z", 
        "self": "/v2/images/e7db3b45-8db7-47ad-8109-3fb55c2c24fd", 
        "file": "/v2/images/e7db3b45-8db7-47ad-8109-3fb55c2c24fd/file", 
        "schema": "/v2/schemas/image"
    }


####Add an Image Tag

**PUT /v2/images/\<IMAGE_ID\>/tags/\<TAG\>**

The the tag you want to add should be encoded into the request URI. For example, to tag image e7db3b45-8db7-47ad-8109-3fb55c2c24fd with 'miracle', you would **PUT /v2/images/e7db3b45-8db7-47ad-8109-3fb55c2c24fd/tags/miracle**. The request body is ignored. 

An image can only be tagged once with a specific string. However, multiple attempts to tag an image with the same string will result in a single instance of that string being added to the image's tags list.

An HTTP status of 204 will be returned.


####Delete an Image Tag

**PUT /v2/images/\<IMAGE_ID\>/tags/\<TAG\>**

The tag you want to delete should be encoded into the request URI. For example, to remove the tag 'miracle' from image e7db3b45-8db7-47ad-8109-3fb55c2c24fd, you would **DELETE /v2/images/e7db3b45-8db7-47ad-8109-3fb55c2c24fd/tags/miracle**. The request body is ignored.

An HTTP status of 204 will be returned. Subsequent attempts to delete the tag will result in a 404.



####List All Images

**GET /v2/images**

Request body ignored.

Response body will be a list of images available to the client. For example:

    {
        "images": [
            {
                "id": "da3b75d9-3f4a-40e7-8a2c-bfab23927dea", 
                "name": "cirros-0.3.0-x86_64-uec-ramdisk", 
                "status": "active", 
                "visibility": "public", 
                "size": 2254249, 
                "checksum": "2cec138d7dae2aa59038ef8c9aec2390", 
                "tags": ["ping", "pong"],                
                "created_at": "2012-08-10T19:23:50Z", 
                "updated_at": "2012-08-10T19:23:50Z", 
                "self": "/v2/images/da3b75d9-3f4a-40e7-8a2c-bfab23927dea", 
                "file": "/v2/images/da3b75d9-3f4a-40e7-8a2c-bfab23927dea/file", 
                "schema": "/v2/schemas/image"
            }, 
            {
                "id": "0d5bcbc7-b066-4217-83f4-7111a60a399a", 
                "name": "cirros-0.3.0-x86_64-uec", 
                "status": "active", 
                "visibility": "public", 
                "size": 25165824,
                "checksum": "2f81976cae15c16ef0010c51e3a6c163", 
                "tags": [],
                "created_at": "2012-08-10T19:23:50Z", 
                "updated_at": "2012-08-10T19:23:50Z",
                "self": "/v2/images/0d5bcbc7-b066-4217-83f4-7111a60a399a", 
                "file": "/v2/images/0d5bcbc7-b066-4217-83f4-7111a60a399a/file",
                "schema": "/v2/schemas/image"
            }, 
            {
                "id": "e6421c88-b1ed-4407-8824-b57298249091", 
                "name": "cirros-0.3.0-x86_64-uec-kernel", 
                "status": "active", 
                "visibility": "public", 
                "size": 4731440, 
                "checksum": "cfb203e7267a28e435dbcb05af5910a9", 
                "tags": [], 
                "created_at": "2012-08-10T19:23:49Z", 
                "updated_at": "2012-08-10T19:23:49Z", 
                "self": "/v2/images/e6421c88-b1ed-4407-8824-b57298249091", 
                "file": "/v2/images/e6421c88-b1ed-4407-8824-b57298249091/file", 
                "schema": "/v2/schemas/image"
            }
        ], 
        "first": "/v2/images?limit=3",
        "next": "/v2/images?limit=3&marker=e6421c88-b1ed-4407-8824-b57298249091",
        "schema": "/v2/schemas/images"
    }

TODO: document pagination query parameters and schema bits

TODO: document filtering query parameters

TODO: document sorting query parameters

####Get an Image

**GET /v2/images/\<IMAGE_ID\>**

Request body ignored.

Response body will be a single image entity. Using **GET /v2/image/da3b75d9-3f4a-40e7-8a2c-bfab23927dea** as an example:

    {
        "id": "da3b75d9-3f4a-40e7-8a2c-bfab23927dea", 
        "name": "cirros-0.3.0-x86_64-uec-ramdisk", 
        "status": "active", 
        "visibility": "public", 
        "size": 2254249, 
        "checksum": "2cec138d7dae2aa59038ef8c9aec2390", 
        "tags": ["ping", "pong"],                
        "created_at": "2012-08-10T19:23:50Z", 
        "updated_at": "2012-08-10T19:23:50Z", 
        "self": "/v2/images/da3b75d9-3f4a-40e7-8a2c-bfab23927dea", 
        "file": "/v2/images/da3b75d9-3f4a-40e7-8a2c-bfab23927dea/file", 
        "schema": "/v2/schemas/image"
    }


####Delete an Image

**DELETE /v2/images/\<IMAGE_ID\>**

Encode the ID of the image into the request URI. Request body is ignored.

Images with the 'protected' attribute set to true (boolean) cannot be deleted and the response will have an HTTP 403 status code. You must first set the 'protected' attribute to false (boolean) and then perform the delete.

The response will be empty with an HTTP 204 status code.


### Binary Data API

####Store Image File

**PUT /v2/images/\<IMAGE_ID\>/file**

NOTE: An image record must exist before a client can store binary image data with it.

Request Content-Type must be 'application/octet-stream'. Complete contents of request body will be stored and become accessible in its entirety by issuing a GET request to the same URI.

Response status will be 204.

####Get Image File

**GET /v2/images/\<IMAGE_ID\>/file**

Request body ignored.

Response body will be the raw binary data that represents the actual virtual disk. The Content-Type header will be 'application/octet-stream'.

The [Content-MD5](http://www.ietf.org/rfc/rfc1864.txt) header will contain an MD5 checksum of the image data. Clients are encouraged to verify the integrity of the image data they recieve using this checksum.

If no image data has been stored, an HTTP status of 204 will be returned.

###Appendix: cURL Examples

This section is intended to provide a series of commands a typical client of the API might use to create and modify an image. 

These commands assume the implementation of the v2 Images API is using the OpenStack Identity Service for authentication and authorization. The X-Auth-Token header is used to communicate the authentication token provided by that separate identity service.

The strings $OS_IMAGE_URL and $OS_AUTH_TOKEN represent variables defined in the client's environment. $OS_IMAGE_URL is the full path to your image service endpoint, i.e. 'http://localhost:9292'. $OS_AUTH_TOKEN represents an auth token generated by the OpenStack Identity Service, i.e. '6583fb17c27b48b4b4a6033fe9cc0fe0'.

####Create an Image
```
% curl -i -X POST -H "X-Auth-Token: $OS_AUTH_TOKEN" -H "Content-Type: application/json" \
       -d '{"name": "Ubuntu 12.10", "tags": ["ubuntu", "12.10", "quantal"]}' \
       $OS_IMAGE_URL/v2/images
```

```
HTTP/1.1 201 Created
Content-Length: 434
Content-Type: application/json; charset=UTF-8
Location: http://localhost:9292/v2/images/7b97f37c-899d-44e8-aaa0-543edbc4eaad
Date: Tue, 14 Aug 2012 00:46:48 GMT

{
    "id": "7b97f37c-899d-44e8-aaa0-543edbc4eaad",
    "name": "Ubuntu 12.10",
    "status": "queued",
    "visibility": "private",
    "protected": false,
    "tags": ["ubuntu", "12.10", "quantal"],
    "created_at": "2012-08-14T00:45:05Z",
    "updated_at": "2012-08-14T00:45:05Z",
    "file": "/v2/images/7b97f37c-899d-44e8-aaa0-543edbc4eaad/file",
    "self": "/v2/images/7b97f37c-899d-44e8-aaa0-543edbc4eaad",
    "schema": "/v2/schemas/image"
}
```

####Update the Image

TODO

####Upload Binary Image Data

TODO

####Download Binary Image Data

TODO

####Delete Image

TODO
