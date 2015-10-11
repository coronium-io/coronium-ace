# ~ ACE Modules ~

**In the Coronium ACE Preview release anything is subject to change.**

> Make sure to read the [Installation](install.md) and [Developing](usage.md) sections first.

## ace.api
The `ace.api` method creates an empty API object to add GET and POST "method routing" to.

```lua
local api = ace.api()

--== A "GET" echo method
function api.get.hello( in_data )
  return in_data
end

--== A "POST" echo method
function api.post.hello( in_data )
  return in_data
end

return api
```

If access to both a POST and GET route is needed for the same functionality, you can also just alias one of the HTTP methods.

```lua
local api = ace.api()

function api.get.hello( in_data )
  return in_data
end
--== Alias for POST
api.post.hello = api.get.hello

return api
```

The `hello` method will now fire on both a POST and a GET request.

_Parameters:_

None

_Returns:_

Type   | Description
------ | ----------------------------------
Object | An object to place api methods on.

---

## ace.object
An easy to use, persistent data storage object.

Methods | Description
--------|-------------
ace.object.fetch(id)   | Get a data object (obj).
obj:get(key)     | Get the value of a key.
obj:set(key,value) | Set a key with a value.
obj:del(key) | Deletes a key and value.
obj:save(close) | Saves the data object.
obj:delete() | Clears out the data object.
obj:close() | Release the data object reference.

```lua
local obj = ace.object.fectch("123")
local username = obj:get("username")
obj:set("last_login", os.time())
obj:save(true)
```

```lua
local obj = ace.object.fetch("123")
local last_login = obj:get("last_login")
obj:delete()
obj:close()
```

---

## ace.request
A module to create an outgoing HTTP request. A request has a multipart setup; create a `request` instance, build a `params` table for the request, and finally call the request with the params and wait for the result. Any returned data from the request will be in the `result.body` variable.

</br>
> An example of using the request module.

```lua
local req_object = ace.request.new("http://google.com")

local params =
{
  path = '/search',
  method = "GET"
  headers =
  {
    ["X-Auth"] = "whatever"
  }
}

local result = req_object:request( params )
if result then
  print( result.body )
end
```

> Creating a new request object.

_Parameters:_

Name | Type   | Description | Required
---- | ------ | -------------------------------- |
url  | String | The url to point the request at. | Yes
port | String | An optional port number. | No

```lua
local req_object = ace.request.new( host [, port ] )
```

_Returns:_

Type   | Description
------ | ---------------------
Object | A new ACE request object.

> Setting up the request params table.

The request command parameter table contains the following keys:

_Table:_

Name    | Type   | Description                                   | Required
------- | ------ | --------------------------------------------- | --------
path    | String | The actual resource path.                     | Yes
method  | String | The HTTP method to use [GET,POST,DELETE,PUT]. | Yes
headers | Table  | Any extra headers to send along with the request.   | No


```lua
local params =
{
  path = "/users",
  method = "GET"
}
```

> Sending the request.

And finally, call the `:request` method of the request module (with a colon, not a dot).

```lua
local result = req_object:request( params )
if result then
  print( result.body )
end
```

__If the request is successful the content will be in the `request.body` result key.__

---

## ace.tpl
A template system. This module only has one method called `render`.

```lua
local tpl_string = ace.tpl.render('home.index', {site_title="My Cool Site"})
```

When referencing templates in the __tpl__ directory, you do not need to include the __tpl/__ prefix, as shown above.

> Templates can have any extension type, for example __index.tpl__ and __index.html__ work the same.

_Parameters:_

Name      | Type  | Description | Required
--------- | ----- | ----------------------- |
tpl_path | String | The template name (with extension) or full path to the template file. | Yes
token[s] | Table | The hashed table of data to pass to the template. | No

_Returns:_

Type  | Description
----- | ------------------------------------
String | The rendered template as a string.

Your HTML template may look something like this:

```html
<!-- /tpl/home.html -->
<!-- some html code -->

<h1>{{ site_title }}</h1>

<!-- some html code -->
```

Looping values:

```lua
-- Lua
local tpl =
{
  dogs = {'Muffy','sparky','woofy'}
}
return ace.tpl.render('cogs.html', tpl)
```

```html
<!-- dogs.html -->
...
<body>
  {% for _, dog in ipairs( dogs ) %}
    <div>{{ dog }}</div>
  {% end %}
</body>
...
```
---

## ace.upload
___Not available during Preview.___

---

## ace.jtbl
A special Lua table that can output itself as JSON.

```lua
local jtbl = ace.jtbl()
jtbl.username = "Marco"
local json_str = jtbl.toJson()
```

__Json Response:__ `{"username":"Marco"}`

_Parameters:_

None

_Returns:_

Type  | Description
----- | --------------------------------------
Table | A Lua table with a `.toJson()` method.

---

## ace.jsonify
A runtime safe method to convert Lua tables to a JSON string.

```lua
local json_string, err = ace.jsonify( lua_table )
if not json_string then
  print( err )
else
  print(json_string)
end
```

_Parameters:_

Name      | Type  | Description | Required
--------- | ----- | ----------------------- |
lua_table | Table | A JSON encodable table. | Yes

_Returns:_

Type  | Description
----- | ------------------------------------
String | The JSON string or nil on error.
Error | A Lua string error. If JSON is nil.

---

## ace.tableize
A runtime safe method to convert a JSON encoded string to a Lua table.

```lua
local lua_table, err = ace.tableize( json_string )
if not lua_table then
  print( err )
else
  print(lua_table.some_key)
end
```

_Parameters:_

Name        | Type   | Description | Required
----------- | ------ | ------------------- |
json_string | String | A decodable string. | Yes

_Returns:_

Type  | Description
----- | ------------------------------------
Table | The data table or nil on error.
Error | A Lua string error. If table is nil.

---

## ace.log
Log a message to the ACE instance log. You can view these logs using the [ace-card](install.md#ace-card) on your Docker host.

```lua
ace.log("Something totally cool happened")
```

_Parameters:_

Name        | Type   | Description | Required
----------- | ------ | ------------------- |
log_message | String | A plain string to log. | Yes

_Returns:_

Nothing


*You can use the [ace-card](install/#ace-card) to view the app logs.*

---

## ace.uuid
Returns a universally unique identifier.

```lua
local new_uuid = ace.uuid()
```

_Parameters:_

None

_Returns:_

Type  | Description
----- | --------------------------------------
String | A unique identifier.

---

# ~ ACE Utilities ~

## ace.sf
This tool is an alias to the [Lua string.format](http://stackoverflow.com/questions/1811884/lua-string-format-options) functionality.

```lua
local endpoint = "hello"
local formatted = ace.sf("some/path/%s", endpoint )

--== Result: "some/path/hello"
```

_Parameters:_

Name       | Type   | Description | Required
---------- | ------ | --------------------------------------------------------------------- |
format_exp | String | A string expression with specially marked tokens for data. | Yes
token[s]   | String | A comma delimited list of tokens to replace in the format expression. | Yes

_Returns:_

Type   | Description
------ | -------------------------
String | A token formatted string.

---

## ace.trim
Trim extra space from both ends of a string.

```lua
local trimed = ace.trim( " hi there  ")

--== trimed = "hi there"
```

_Parameters:_

Name        | Type   | Description | Required
----------- | ------ | ------------------- |
str_content | String | A string to trim. | Yes

_Returns:_

Type   | Description
------ | ------------------
String | An trimed string.

---

## ace.split
Split a string based on a delimiter and return a table array.

```lua
local tbl_array = ace.split("dog*cat*bunny*frog*fish", '*')

print( tbl_array[4] ) -- frog
```

```lua
--== With no delimiter, defaults to "space"
local tbl_array = ace.split("a b c d e f")

print( tbl_array[2] ) -- b
```

_Parameters:_

Name        | Type   | Description | Required
----------- | ------ | ------------------- |
str_content | String | A delimited string to split. | Yes
spliter | String | The delimiter to split on. Default: single space " " | No

_Returns:_

Type   | Description
------ | ------------------
Table | A table array with the split parts.


---

## ace.escape
Used for escaping HTML entities.

```lua
local escaped = ace.escape( "Sugar & spice & everything nice." )

--== escaped = "Sugar &amp; spice &amp; everything nice."
```

_Parameters:_

Name        | Type   | Description | Required
----------- | ------ | ------------------- |
str_content | String | A string to escape. | Yes

_Returns:_

Type   | Description
------ | ------------------
String | An escaped string.

---

## ace.unescape
Used for unescaping HTML entities.

```lua
local unescaped = ace.unescape( "Sugar &amp; spice &amp; everything nice.")

--== unescaped = "Sugar & spice & everything nice."
```

_Parameters:_

Name        | Type   | Description | Required
----------- | ------ | --------------------- |
str_content | String | A string to unescape. | Yes

_Returns:_

Type   | Description
------ | --------------------
String | An unescaped string.

---

## ace.encode64
Apply Base 64 encoding to a string.

```lua
local base64_string = ace.encode64( "Super fun is fun to be had!" )

--== base64_string = "U3VwZXIgZnVuIGlzIGZ1biB0byBiZSBoYWQh"
```

_Parameters:_

Name        | Type   | Description | Required
----------- | ------ | ------------------- |
plain_string | String | A string to Base 64 encode. | Yes

_Returns:_

Type   | Description
------ | ----------------------
String | An Base 64 encoded string.

---

## ace.decode64
Decode a Base 64 encoded string.

```lua
local decoded = ace.decode64( "U3VwZXIgZnVuIGlzIGZ1biB0byBiZSBoYWQh" )

--== decoded = "Super fun is fun to be had!"
```

_Parameters:_

Name        | Type   | Description | Required
----------- | ------ | --------------------- |
base64_string | String | A Base 64 string to decode. | Yes

_Returns:_

Type   | Description
------ | --------------------
String | A plain string.

---

## ace.json
Provides direct access to the [__cjson__](http://www.kyne.com.au/~mark/software/lua-cjson-manual.html#_api_functions) library, which allows extended, less used, configuration options. You should only need to reference this module for JSON data that is misbehaving. Take note, that if an encode or decode process fails using __cjson__, your app will throw a runtime error.

> Unless your JSON needs special handing, use [ace.jsonify](#acejsonify) and [ace.tableize](#acetableize) for __runtime safe JSON encoding and decoding.__

---

# ~ ACE Wraps ~

Though you can easily return a Lua table structure to the client directly, there are a couple of helper functions, called "wraps", that you can utilize to provide for more structured responses.

## ace.result

Send a response to the client with a `result` key.

___String___

```lua
return ace.result("Some String Data")
```

__Json Response:__ `{"result":"Some String Data"}`

___Hashed Lua Table___

```lua
return ace.result({username:"Ed"})
```

__Json Response:__ `{"result":{"username":"Ed"}}`

___Lua Table array___

```lua
return ace.result({'fred','velma','shaggy'})
```

__Json Response:__ `{"result":["fred","velma","shaggy"]}`

The following aliases can be used in place of `ace.result`

- `ace.response`
- `ace.answer`
- `ace.say`
- `ace.out`

Despite the name, they all return a regular payload: `{"result":"..."}`. The choice is yours as to which one you care to use.

### Overriding the "result" key

There may be special situations where you would like to change the `result` key to something else. This can be done by passing the replacement key after your content.

```lua
return ace.result("Hello to you!", "greeting")
```

__Json Response:__ `{"greeting":"Hello to you!"}`

---

## ace.error

You can pass a formatted error back with this wrap.

```lua
return ace.error("The database blew up!", 911)
```

__Json Response:__ `{"error":"The database blew up!","errorCode":911}`

_Parameters:_

Name        | Type   | Description | Required
----------- | ------ | --------------------- |
error | String | The human readable error string. | Yes
error_code | Number | An optional related error code. | No

### Sending a generic error response

If you just need to pass a general error back, use:

```lua
return ace.error()
```

__Json Response:__ `{"error":"Unknown error occurred","errorCode":-99}`
