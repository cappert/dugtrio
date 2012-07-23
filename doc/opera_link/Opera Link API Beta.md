# [ Opera Link API Beta ](http://www.opera.com/docs/apis/linkrest/)

_Last update: 2012-06-14_

On this page...

* [Overview](#overview)
  * [Forming requests](#forming-requests)
  * [Accessing data](#accessing-data)
  * [Mutating data](#mutating-data)

* [Importing structured data](#importing-structured-data)
  * [Datatype information](#datatype-information)
  * [Target folders](#target-folders)
  * [Receiving responses](#receiving-responses)

* [Authentication](#authentication)

##  Overview

The REST API for Opera Link is intended to provide read and write access to a
user's Opera Link data storage over an ` HTTP` interface. The API is similar
to the APIs of popular web services and should be fairly straightforward to
use.

All `HTTP` methods are supported and work as expected—`GET`, `POST`, `PUT`,
`DELETE`. If you are uncomfortable with or cannot use the less typical methods
`PUT` and `DELETE`, you can still use the API fully, by specifying actions
using a parameter in a standard `POST` request.

`GET` requests are exclusively read-only; that is, the request will not have
any side-effects and will never change the state of the requested item on the
server. `POST`, `PUT`, and `DELETE` requests are exclusively for modifying
user data; they will always result in modification of the user data, given
that they are successful.

The REST API requests are stateless and complete—no mechanisms are used to
track progress of an action beyond the scope of a single request.

The REST API exposes and manipulates the following data types:

  * Bookmarks
  * Notes
  * Speed Dial

Other data types and further functionality may be included in further
releases.

###  Forming requests

All REST requests will have the following URI pattern:

    
    
    https://link.api.opera.com/rest/<datatype>/(<item_id>)
    

`datatype`

     This is one of the datatypes stored in Opera Link: "`bookmark`", "`speeddial`", "`note`" and similar.
`item_id`

     When used, this would signify the unique identifier by which a specific item in the system is referenced.

All requests have a list of required parameters and can be categorized into
three groups:

  1. **Authentication parameters**
    * These are used to authorise access and identify the user data.
    * They are required parameters in every request.
    * See the "Authentication" section for details.
  2. **API parameters**
    * These are used to specify the details of the request and desired response settings.
    * These methods are always prefixed with the `string api_` as in the following items: 
      * **`api_method`** (optional, POST-only): This parameter explicitly specifies the method you want to execute and overrides the default create action.
      * **`api_output`** (optional): This specifies the output format for the responses. It overrides any other methods for specifying the output. See the following section "Choosing the output format" for more details.
  3. **Method parameters**
    * These parameters are additional data needed for the completion of the requests.
    * They are dependent on the method and/or the datatype being manipulated and are usually datatype-related values - titles, descriptions, positions, etc.
    * These parameters are clarified in each method's description.

All other parameters sent in the request that are not recognized will be
silently ignored. They will not be processed, will not raise any errors, and
will not influence the request in any way.

##  Accessing data: all `HTTP GET` requests

For example, a simple GET request to fetch the properties of a bookmark with
`id 319A38DB4581426DA48CAB58C2528FD4` will look like this:

    
    
    GET https://link.api.opera.com/rest/bookmark/319A38DB4581426DA48CAB58C2528FD4/ HTTP/1.1
    

Direct item request

     This returns the properties of the requested item.
`/children/`

     This returns all children of the item requested in the URI. The method is not recursive and will not return any sub-items. If the node does not have any contained items, or by definition cannot contain any items, this call will return an empty list.
`/descendants/`

     This returns all descendants of the item requested in the URI, recursively. All entries that contain another level of items within themselves will encapsulate them in a children property. Keep in mind that a full tree of contained items can be prohibitively big in some situations.
     If the node does not have any contained items, or by definition cannot contain any items, this call will return an empty list.

If the `item_id` part of the URI is skipped in a request, then for bookmark
and note datatype `/children/` and `/descendants/` suffixes query about a root
folder of the tree structure. To get a list of all dials on the Speed Dial
page, `item_id` should be skipped and the suffix `/children/` should be used.

##  Mutating data: `POST`, `PUT`, and `DELETE` requests

In a REST interface, HTTP methods result in their expected behavior:

  * **`POST`** for creating new items. See the method `create` below.
  * **`PUT`** for updating existing items. See the method `update` below.
  * **`DELETE`** for removing existing items. See the method `delete` below.

Since not all libraries support HTTP methods that are different from GET and
POST and since the Link API provides additional methods of manipulating the
Opera Link data, the API provides an extra interface for specifying which
operation you want to execute. This is done via a POST request through the
addition of a parameter called `api_method`. It can take any of the following
values:

`create`

     This creates a new item of the datatype specified in the URI. The request must contain all values mandated by that datatype. This method has a few datatype-specific differences, which are further described in the "Datatype information" section. Generally, when creating a new item, the item will be returned in the response with a new unique identifier as `id`. You need this identifier if you want to further manipulate the item. If an `item_id` is specified in the URI, the system will attempt to create the new item under the one specified in the URI. If that item is a trailing node, which cannot contain any children, this method will fail.
     **Required parameters:**

  * `item_type`: This is the type of the object being created; for example `bookmark, bookmark_folder, bookmark_separator, speeddial`, etc.
  * Datatype specific parameters: see each data type description for details.
     **Returns:**

  * The newly added item, as it would be returned by a `get` request

`delete`

     This means the item is permanently and irrecoverably removed from the system. If the item does not exist, the request will succeed, and the response will be "200 OK". In the case of a request to delete trash folder, the "400 Bad Request" error is raised.
     **Required Parameters:**

  * None
     **Returns:**

  * 204 Deleted

`trash`

     This moves the item to the trash container of this datatype. This method is not defined for datatypes that have no notion of trash folders—for example, in the case of Speed Dial, search engines and typed history items. In the case of a request with action trash on trash folder, the "400 Bad Request" error is raised.
     **Required Parameters**: 

  * None
    **Returns**: 

  * The moved item, as it would be returned by a `get` request

`update`

     This modifies the contents of the item. Note that this method does not require parameters to function; however it will only process parameters that are accepted by the specific datatype. See the "Datatype information" section for more details. If you want to remove a specific property of an item completely, send an empty value for it; empty properties are considered missing and will be removed. If read-only parameters are sent with changed values, then the action results in a "400 Bad Request" error.
    **Required Parameters**: 

  * None
     **Returns**: 

  * The updated item, as it would be returned by a `get` request

`move`

     This moves the item to a different position in the structures of the datatype, relative to a given reference item. Relative position "`into`" moves the item to the end of the reference item's list of children (if the reference item is not a folder, or is a subfolder of the element to be moved, a 400 Bad Request is returned). Use an empty string as `reference_item` to move an item to the root folder. The relative position "`before`" moves the element before the reference item. Likewise, the relative position "`after`" moves the element after the reference item. If there is no item with the given reference ID, then error 404 Not Found is returned. 
    **Required Parameters**: 

  * **`reference_item`**: The ID of the item being a point of reference to the new location of the moved element
  * **`relative_position`**: One of the values `into`, `after`, and `before`
    **Returns**: 

  * The moved item, as it would be returned by a `get` request

###  Input formats

The Opera Link API supports input in different formats. Apart from the default
`x-www-form-urlencoded` input, you can send `POST` requests with XML or JSON
serialized data as well. The API will automatically detect and process the
input, based on the `Content-Type` header of the request.

These three inputs are equivalent:

  * **Content-Type: `application/x-www-form-urlencoded`**
    
    
    ?item_type=bookmark&title=MyOpera&uri=www.myopera.com
    

  * **Content-Type: `application/json`**
    
    
    {
        "item_type": "bookmark",
        "title": "MyOpera",
        "uri": "www.myopera.com"
    }

  * **Content-Type: `application/xml`**
    
    
    <request>
      <item_type>bookmark</item_type>
      <title>MyOpera</title>
      <uri>www.myopera.com</uri>
    </request>
    

##  Importing structured data

In order to make importing larger amounts of items faster and easier, some
datatypes support an extra `import` method. Currently you can use it for
bookmarks, notes, and url filters. This method allows you to import large
amounts of items as a single operation.

Since you need to submit structured data (lists or trees) to the API, the
import methods _cannot_ be made with ` x-www-form-urlencoded` input, but
_only_ with XML or JSON data. The input format matches the output format
described in the "Choosing the output format" section.

Importing of structured data via a `POST` request is done to this address:

    
    
    https://link.api.opera.com/rest/<datatype>/(<folder_id>/)import/
    

###  JSON (JavaScript Object Notation) import format

Importing of structured data via JSON (JavaScript Object Notation) is done in
this format:

    
    
    [
        {
            "item_type": "bookmark",
            "properties": {
                "uri": "http://duckduckgo.com/",
                "title": "DuckDuckGo"
                }
        },
        {
            "item_type": "bookmark_folder",
            "properties": {
                "title": "Opera"
                },
            "children": [
                {
                    "item_type": "bookmark",
                    "properties": {
                        "uri": "http://www.opera.com",
                        "title": "Opera - The Fastest Internet Browser"
                        },
                },
                {
                    "item_type": "bookmark",
                    "properties": {
                        "uri": "www.myopera.com",
                        "title": "My Opera"
                        }
                }
            ]
        }
    ]
    

###  XML import format

Importing of structured data via XML is done in this format:

    
    
    <request>
      <resource>
        <item_type>bookmark</item_type>
        <properties>
          <uri>www.google.com</uri>
          <title>Google Search</title>
        </properties>
      </resource>
      <resource>
        <item_type>bookmark_folder</item_type>
        <properties>
          <title>Opera</title>
        </properties>
        <children>
          <resource>
            <item_type>bookmark</item_type>
            <properties>
              <title>Opera Software</title>
              <uri>http://opera.com</uri>
            </properties>
          </resource>
          <resource>
            <item_type>bookmark</item_type>
            <properties>
              <title>Opera Community</title>
              <uri>http://www.my.opera.com/</uri>
            </properties>
          </resource>
        </children>
      </resource>
    </request>
    

##  Datatype information

Although most operations in the Opera Link API are consistent throughout the
different types of data stored in the system, some extra policies are enforced
on some of them to keep consistency with your desktop and mobile clients.
Below, you can see detailed descriptions of the specifics of every datatype.

###  Bookmarks

Bookmarks support all the functionality described in the API, including
container items (folders), trash folder, and automatic creation of unique
identifiers. The defined item types for this datatype are:

`bookmark`

     Properties: 

  * **`title`:** self-explanatory **[Required]**
  * **`uri`:** self-explanatory **[Required]**
  * **`description`:** the bookmark's long description
  * **`nickname`:** the browser shortcut for quick access from the address bar
  * **`icon`:** base64-encoded, PNG favicon
  * **`created`:** creation timestamp (default - current timestamp) 
    * See the explanation on the timestamp format
  * **`visited`:** last visited timestamp 
    * See the explanation on the timestamp format
  * **`show_in_panel`:** tells the browser to show the item in the bookmark panel
  * **`show_in_personal_bar`:** tells the browser to show the item in the personal bar
  * **`panel_pos`:** tells the browser which position in the bookmark panel this item should be shown: _only valid if `show_in_panel` is `true`, otherwise it's usually set to `-1`_
  * **`personal_bar_pos`:** tells the browser which position in the personal bar this item should be shown: _only valid if `show_in_personal_bar` is `true`, otherwise it is usually set to `-1`_

`bookmark_folder`

     Properties: 

  * **`title`:** self-explanatory **[Required]**
  * **`description`:** the bookmark's long description
  * **`nickname`:** the browser shortcut for quick access from the address bar
  * **`type`:** read-only; if present, the only valid value is "`trash`", which means that the folder acts as the Trash folder for the datatype
  * **`target`:** read-only; signifies that this folder is accessed by a device with limited capabilities; see the "Target folders" section for more details
  * **`created`:** creation timestamp (default - current timestamp) 
    * See the explanation on the timestamp format

`bookmark_separator`

     Properties: 

  * This item has no usable properties.

###  Notes

Notes support all the functionality described in the API, including container
items (folders), trash folder, and automatic creation of unique identifiers.
The defined item types for this datatype are:

`note`

     Properties: 

  * **`content`:** the text of the note **[Required]**
  * **`created`:** creation timestamp (default - current timestamp) 
    * See the explanation on the timestamp format
  * **`uri`:** link to the site from which the note was copied, if any

`note_folder`

     Properties: 

  * **`title`:** self-explanatory **[Required]**
  * **`type` :** Read-only—if present, the only valid value is "`trash`", which means that the folder acts as the Trash folder for the datatype.
  * **`target`:** Read-only—this signifies that this folder is accessed by a device with limited capabilities; see the Target folders section for more details.

`note_separator`

     Properties: 

  * This item has no usable properties.

###  Search engines

Search Engines are not a structured data type; there is no ordering or
nesting. The supported operations are:

  * create
  * update
  * delete

Operations like move and trash are not defined, since the Search Engine does
not have a nested structure.

The only defined item type for this datatype is:

`search_engine`

     Properties: 

  * **`title`:** Self-explanatory **[Required]**
  * **`uri`:** Search result URI; the formatting operator `%s` will be replaced by the search terms **[Required]**
  * **`encoding`:** The character encoding of the search field (default - "`utf-8`")
  * **`is_post`:** Is the search field using the `POST` method
  * **`post_query`:** The `POST` parameters for the request
  * **`icon`:** Is base64-encoded, PNG favicon
  * **`key`:** A shortcut to the search engine which can be used in the address field of the browser
  * **`show_in_personal_bar`:** tells the browser to show the item in the personal bar
  * **`personal_bar_pos`:** tells the browser which position in the personal bar this item should be shown: _only valid if `show_in_personal_bar` is `true`, otherwise it is usually set to `-1`_

###  Speed Dial

Speed Dial as a data type behaves slightly differently. Its items do not get
automatically generated IDs assigned and do not support some of the
operations. The supported operations are:

  * create
  * update
  * delete

Operations like `move` and `trash` are not defined, since the Speed Dial does
not have a nested structure.

The only defined item type for this datatype is:

`speeddial`

     Properties: 

  * **`title`:** self-explanatory **[Required]**
  * **`uri`:** self-explanatory **[Required]**
  * **`position`:** read-only, the location of the dial
  * **`icon`:** base64-encoded, PNG favicon
  * **`reload_enabled`:** tells the browser to reload the Speed Dial thumbnail automatically
  * **`reload_interval`:** tells the browser how many seconds it should wait between reloads: _ignored if `reload_enabled` is not `true`_
  * **`reload_only_if_expired`:** tells the browser to only reload automatically if the site has changed: _ ignored if `reload_enabled` is not `true`_

###  Url filters

Url Filters are not a structured data type; there is no ordering or nesting.
The supported operations are:

  * create
  * update
  * delete

Operations like move and trash are not defined, since the Url Filter does not
have a nested structure.

The only defined item type for this datatype is:

`urlfilter`

     Properties: 

  * **`content`:** the filter itself **[Required]**
  * **`type`:** include/exclude - for whitelist/blacklist **[Required]**

A quick description of the url filter format can be found at: [ Opera's Kiosk
Mode: URL filtering](http://www.opera.com/support/mastering/kiosk/#url-filter)

##  Target folders

Target folders are special folders that contain extra properties, related to
their special use in some devices. Typically there is some device with limited
capabilities that sees the content of those folders as the only content
available. The only currently defined target is "mini".

###  Target properties

The properties of target folders describe certain rules related to their
proper usage. They are just hints for the clients and are not enforced by the
server, but note that `sub_folders_allowed` and `separators_allowed` apply not
only to the target folder itself, but to any subfolder inside it.

`move_is_copy`

     When the user moves an item from outside the target to any folder inside it, the item **SHOULD** be copied instead of moved, keeping the original intact.
`deletable`

     This specifies whether the folder is allowed to be deleted.
`sub_folders_allowed`

     This specifies whether the folder is allowed to contain sub-folders.
`separators_allowed`

     This specifies whether the folder is allowed to contain separators.
`max_items`

     This specifies the maximum number of items that can be contained within this folder. The special value `0` indicates no limit.

##  Timestamp format specification

Created and visited properties should have the timestamp format specified by [
RFC 3339 - Date and Time on the Internet:
Timestamps](http://tools.ietf.org/html/rfc3339). For example, a valid
timestamp is:

    
    2010-07-22T05:40:59Z

##  Receiving Responses

The Link API uses HTTP status codes in its responses. In the case of a
successful request, a `200 OK` response will contain the requested data. All
other response codes will not have a message body.

###  Response codes

Below is a list of all used HTTP status codes:

`200 OK`

     This is a successful request.
`204 Deleted`

     The item was successfully deleted.
`400 Bad Request`

     The request is invalid and cannot be processed. The cause of this is often missing a required parameter or trying to execute an invalid method on an item.
`401 Unauthorized`

     The request cannot be allowed, possibly because your authentication information is invalid or because too many requests sent in a short period made the throttling ban them.
`404 Not Found`

     The item you seek or wish to manipulate was not found.
`500 Internal Server Error`

     This an unexpected server error. If you get such errors, you should consider filing a bug.
`501 Not Implemented`

     You are trying to execute a method that is not implemented. This can happen when you execute a method that is not supported by the specific datatype or if you misspelled a method name in the request.

###  Response structure

All responses have an unified structure, regardless of the format in which
they are presented. Every response contains a list of resources that
represents the results of the API calls. Usually those will be items of the
corresponding Link datatype for which the call was made.

Every resource is identified by its _type_ and _id_.

  * The **_type_** ("`item_type`" property) describes what the item is: a bookmark, a note folder, a (Speed Dial) dial item.
  * The **_id_** ("`id`" property) is the unique identifier assigned to every item within the system.

Every resource has a set of properties that shows the actual data contained
within each item. All properties are datatype specific and their number can be
different for each item. Empty properties are considered to be missing and
will not be returned.

####  Choosing the output format

There are currently two supported output formats: JSON and XML. By default the
API will reply with the `Content-Type` used for the request, or with JSON if
none was provided.

You have two ways to select an output format: the `api_output` parameter and
the Accept HTTP header.

  * To set the output format with the `api_output` parameter, simply pass "`json`" or "`xml`" to it.
  * To set it with the Accept header, set it to "`application/json`" or "`application/xml`".

Setting the output format explicitly _always overrides_ the default setting.
The order in which the different possibilities are evaluated is:

  * `api_output` parameter > Accept header > Content-type header > Default (JSON)

####  JSON (JavaScript Object Notation) Response Format

If nothing else is specified, the REST API will return response data in JSON
format, which is the default output format for the system. The response is a
list that contains objects, each representing one result of the API call. The
content of all items will be what you would expect in a complex JSON
structure: it is contained within objects and lists. The following script is
an example of JSON output:

    
    
    [
      {
        "item_type": "bookmark_folder",
        "id": "46A52AB96FDD4991943193797186E4D5",
        "properties": {
          "nickname": "comics",
          "title": "Daily Comics"
        }
      }
    ]
    

####  XML Response Format

The Opera Link API supports output in XML format. This output can be requested
using the methods described above. The root of the document is an element that
contains the aforementioned list of results, each contained within an element.
The following markup is an example of XML output:

    
    
    <response>
      <resource>
        <item_type>speeddial</item_type>
        <id>1</id>
        <properties>
          <uri>www.google.com</uri>
          <title>Google Search</title>
        </properties>
      </resource>
    </response>
    
##  Authentication

Since the requests are stateless, every request MUST contain authentication
information. If authentication information is not provided, the server will
automatically reply with HTTP code 401 Unauthorized.

For clarity, all authentication parameters are omitted throughout the
documentation. They MUST be provided in _every_ request.

Opera Link's Public API uses **auth.opera.com (Auth)** as its authentication
service. All users registered in Opera Auth (i.e., any account registered for
any Opera service, such as My Opera, Opera Unite, Opera Portal, and, of
course, Opera Link itself) can use the Opera Link service and its public API.

Please note Opera Unite will be disabled by default in Opera 12.00 and
unavailable as of Opera 12.50. More information about this is available
[here](http://my.opera.com/addons/blog/2012/04/24/sunsetting-unite-and-
widgets).

###  OAuth parameters

To obtain the necessary tokens, go to [https://auth.opera.com/service/oauth/ap
plications/](https://auth.opera.com/service/oauth/applications/). You can find
more information about OAuth in the [OAuth website](http://oauth.net/).

All requests to Opera Link's public API servce MUST therefore be signed with
HMAC_SHA1 algorithm as described in the OAuth guide or they will be
automatically rejected.

In most cases, you would want to use an already existing library for creating
OAuth requests, and these are available for most languages.

The OAuth procedure requires you to obtain a valid access token and secret and
to use those to sign your requests.

`oauth_timestamp`

     This is a UNIX epoch timestamp, obtained from the system at the time of making the request. Most OAuth packages will do that automatically.
`oauth_nonce`

     This is a randomly generated string that must be unique for every request made to the system. The OAuth server will check and enforce the uniqueness for this parameter, so you MUST create a unique random string on each request. Most OAuth packages will do that automatically.
`oauth_token`

     This is the access token, as obtained by the Opera Auth system.
`oauth_version`

     Set this to "1.0".
`oauth_signature`

     This is the signature for the request, and it is created as OAuth rules dictate. This is usually automatically created for you by the OAuth package that you are using.

For more details about OAuth authentication and examples in Perl and Python,
you can visit our [ OAuth examples
page](http://www.opera.com/docs/apis/linkrest/oauthexamples/).

Copyright © 2012 [Opera Software ASA](http://www.opera.com/). All rights
reserved. 

More than **270 million** users worldwide
