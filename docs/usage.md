# ~ ACE Development ~

A __Coronium ACE__ instance needs an "app" directory to connect to during runtime. This folder lives on the actual __Docker Host__, ___not in a Coronium ACE container instance.___

An "app" directory can contain as many modules as you can fit, which in turn can handle many api calls. You can also run additional ACE instances for separated environments via Docker, or duplicate environments for load balancing.

> In most cases one instance of ACE will be sufficient, because of the app/module system, which is limited only by disk capacity of the host.

## Apps Directory

Your modules will be placed in the `/home/<app_name>` directory of the __Docker Host__. You can SFTP, or SCP to this location to manage files.

> Example of Docker host ACE directory structure:

```bash
/home
  /app_name
    /module_name
      api.lua
      /tpl
        tpl.html
    /module_name
      api.lua
      ...
```

> __During Preview, closing an ACE container does not remove the "app" it was bootstrapping.__

---

## Input and Output

In essence, ACE is a standard HTTP server, with a custom Lua based API. With this in mind, it's important to understand how the HTTP protocol works.

Except when facilitated by other services, an HTTP interaction with a client has a __request__ and __response__ phase. Once the response phase has taken place, that's the end of that requests lifecycle. Persistence is emulated through data storage, modules, cookies, and other methods.

> A client loses state after the response phase. Each request starts the lifecycle again.

When a client makes a request to one of the method routes in your API, they often will send additional data to be processed. Coronium ACE only supports __GET__ and __POST__ requests.

If the client request is __GET__, then any query string attached to the endpoint url will be parsed out into a hashed Lua table for use in the ace framework. For example:

```html
http://ace.docker.host:12345/users/login?username=pony&password=boy
```

When this client hits the ACE server the query string is processed into a table like so:

```lua
{
  username = "pony",
  password = "boy"
}
```

This table is then pushed to the proper method route via the `in_data` parameter of the method.

```lua
-- app/users/api.lua

local api = ace.api()

function api.get.login( in_data )
  local username = in_data.username
  local password = in_data.password
  
  --== Do some auth or something here
  
  local msg = ace.sf("Good to see you, %s", username)
  return ace.result( msg )
end

return api
```

As you can see above, the `in_data` table will have whatever data was passed in from the client.

The functionality is exactly the same for a client __POST__ request, only the system attempts to convert the "posted" body data instead. __Coronium ACE requires all posted data is in a JSON encoded format.__

```lua
-- app/users/api.lua

local api = ace.api()

function api.post.login( in_data )
  local username = in_data.username
  local password = in_data.password
  
  --== Do more stuff here.
  
end

return api
```

### The Response Phase

When it's time to return some data back to the client, the most basic return is just a straight table:

```lua
-- app/users/api.lua

local api = ace.api()

function api.get.hello( in_data )
  return { greeting = "Hello " .. in_data.username }
end

return api
```

Or through the use of ACE wraps:

```lua
-- app/users/api.lua

local api = ace.api()

function api.get.hello( in_data )
  local username = in_data.username
  local greeting = ace.sf("Glad to meet you, %s", username)
  
  -- You can use a "wrap" for quick string output (see API)
  return ace.result( greeting )
end

return api
```

__Request URL:__ `http://your.docker.host:12345/users/hello?username=Steve`

__Json Payload:__ `{"result":"Glad to meet you, Steve."}`

> Any Lua table data will converted to JSON before being sent back down the wire.

### Request Headers

Sometimes you may need to inspect the incoming headers for auth, etc. You can access the headers by just including an extra parameter for them. The headers are returned in a hashed Lua table.

```lua
local api = ace.api()

function api.get.hello( in_data, in_headers )
  
  -- Print the headers
  for name, value in pairs( in_headers ) do
    print( name, '=>', value )
  end

  return ace.result("Thanks for the headers!")
return api
```

You can access the header value directly if you already know what you're looking for:

```lua
local api = ace.api()

function api.post.login( in_data, in_headers )
  local auth_header = in_headers['X-Auth-Token']
  
  if auth_header = "12345" then
    return ace.result("You passed!")
  end
  
  return ace.error("You lose, no Auth", 101)
end

return api
```
---

## Making ACE Modules

Creating an ACE api module is very simple. As you read on, you'll see that you have just enough control within the framework, while still keeping it small, simple, and fun to work with.

> __You can also create a 'starter' module by running:__

```bash
ace new <app_name> <new_mod_name>
```

Or create one manually by following along below.

__A starter ACE api module directory structure looks like:__

```bash
module_name
  api.lua
  /tpl
```
* Keep the module name simple. No spaces. Use common characters. `user` or `products` are good examples.

* The module and method names are publicly visible as part of the url if viewed in a browser.

* There must be an `api.lua` in the folder, and it must be named specifically `api.lua`.

* Additionally, create a new folder specifically called `tpl` for any template needs.

__Setting up the api.lua__

> The `ace` library is baked into the ACE binary at a global level. No need to `require` it.

Let's look at a possible `tools` module:

```bash
tools
  api.lua
  /tpl
```

At bare minimum you'll need the following in your `api.lua`:

```lua
-- app/tools/api.lua

--== Get a new api instance from ACE ==--
local api = ace.api()


return api

```

And now your ready to start adding your api methods to the module.

> A simple `echo` api method that will trigger on a __GET__ request:

```lua
-- app/tools/api.lua

local api = ace.api()

--== An echo api GET function
function api.get.echo( in_data )
  return in_data
end

return api
```

The API endpoint would end up looking something like:

`http://your.docker.host:12345/tools/echo?username=Chris`

The `in_data` is the query string parsed into a Lua table. Here we just send the `in_data` back to the client, which is perfectly valid. Any outgoing data in a table will be JSON encoded automatically. Any incoming data is converted, and placed in a Lua table.

The result sent is JSON by default and will contain `{"username":"Chris"}`. JSON tools for working with result data are available for practically every programming language.

__A note about the `in_data`__

The `in_data` table object that gets passed into your methods will contain either the parsed query string as a hash table for __GET__ calls, or the content body keyed in a hash table for __POST__.

> ___The ACE API will only accept a JSON object body for POST requests___.

### The api.lua file

As you can see, the `api.lua` is where we set up our endpoints or "routing" methods.

___ACE is based on the HTTP request/response protocol. The api file is the first and last point of contact with the client. All client data must enter and exit through the api.lua file. This is very important to remember.___

For example, if you wish to add additional code to a module via `require` then you will need to make sure to pass your final output data back to the `api.lua` to return to the client.

__Always return something to the client, if not, you risk the possibility of leaving a connection hanging.__

---

## External Lua Modules

Let's look at an example of using some external code. In this case we have added a `utils.lua` file. The module directory now looks like so:

```lua
tools
  api.lua
  utils.lua
  /tpl
```

__The `utils.lua` file__

```lua
-- app/tools/utils.lua

local m = {}
  
function m.add( a, b )
  return ( a + b ) --this must be returned to api.lua for output.
end
  
return m
```

__The `api.lua` file__

```lua
-- app/tools/api.lua

local utils = require('tools.utils') --module name needed in path
local api = ace.api()

function api.get.add( in_data )
  --collect the incoming data
  local a = in_data.a
  local b = in_data.b
  
  --call the add utils method
  local r = utils.add(a,b)
  
  --return the result to client
  return ace.result( r )

end

return api
```
The API endpoint would look like:

`http://your.docker.host:12345/tools/add?a=21&b=12`

The JSON payload would be: `{"result":33}`

> Remember to include the module name in the require statement

```lua
local utils = require('tools.utils')
```

---

## Using Content Flags

Coronium ACE is deceptive in its capabilities. The "Coronium" ideal is to provide the simplest approach, chasing away unneeded complexity for most projects, and then allow for more advanced elements as needed.

#### Content Types

ACE can output three different types of responses to the client; JSON, HTML, or plain text. This is done through the use of content flags.

> The default output is JSON and does not need to be specified for normal operations.

A content flag is added to the outgoing response, after the initial payload.

### ace.JSON

This flag will try and convert the outgoing content to valid JSON for client consumption.

```lua
return { username = "Charles", likes = "Bikes" } [, ace.JSON ] --JSON is the default content type.
```

### ace.HTML

This flag will output HTML rendered content.

```lua
return "<h1>Hey There, can you see me?</h1>", ace.HTML
```

If using a GET route method, the client will see this as rendered HTML.

### ace.TEXT

This flag will output plain text. This has a variety of uses for IoT.

```lua
return "Here is some plain text even with <i>tags</i>", ace.TEXT
```

The output will be plain text, and the `<i></i>` tags will be visible to the client.

---

## Returning Headers

Taking into consideration the examples above, you can also pass header values back to the client. This can be helpful for a variety of uses, including client authentication.

```lua
local extra_headers =
{
  ["X-App-Auth-Id"] = "12345",
}

return { food = "Yum" }, ace.HTML, extra_headers

```

> You must provide the content flag if you want to add headers.

---

## HTTP Status Codes

If you need to pass an HTTP status code, which can be helpful for communicating certain information, you can do so after the headers.

```lua
local extra_headers =
{
  ["X-App-Auth-Id"] = "12345",
}
local http_status = 404
return {message="Can't find it."}, ace.JSON, extra_headers, http_status
```

> If you don't need additional headers, but you do need to pass a status, then use an empty table as a placeholder:

```lua
local http_status = 200 --all good
return {winner=True}, ace.JSON, {}, http_status
```

---

## Using Flags in Wraps

All of the extra options shown above can also be used in the ACE Wraps. This can be very useful for error returns.

__In ace.result:__

```lua
local headers =
{
  ["X-App-Auth-Id"] = "12345",
}
local http_status = 200
return ace.result("Hello There", ace.JSON, headers, http_status)
```

__In ace.error:__

```lua
local headers =
{
  ["X-Is-Denied"] = "true",
}
local http_status = 401
return ace.error("<strong>Access Denied</strong>", -99, ace.HTML, headers, http_status)
```

___The extra options are not usable in the generic error `return ace.error()`. You must provide an error string and code.___
