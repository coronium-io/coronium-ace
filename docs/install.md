# ~ ACE Installation ~

__Docker 1.8+ Droplet__ on [DigitalOcean](https://www.digitalocean.com/?refcode=cddeeddbbdb8) or __any Docker enabled device.__

DigitalOcean has a pre-built Docker instance available, select it in the "Applications" tab when deploying your droplet.

___Use this [DigitalOcean](https://www.digitalocean.com/?refcode=cddeeddbbdb8) link and get 2 months free credit to test ACE. It also helps support Coronium ACE development.___

### Starting the ACE Stack

SSH into the __Docker__ host once it is active, and run the following in the terminal:

```bash
wget https://s3.amazonaws.com/coronium-ace/installs/ace;chmod +x ace;mv ace /usr/local/bin;ace
```

And then...

```bash
ace init
```

> ACE is Wild!

The default app is named: __ace__

During the initialization a test module was added. You can access the "echo" test for your app by pointing a browser to your Docker host and ACE port with the following path and parameters.

```bash
http://your.docker.host:<ace_port>/echo/test?ace=rocks
```

You should get `{"ace":"rocks"}` back in your browser. If so, you're good to go.

---

## ACE Card

The `ace` tool is your entry point to your underlying ACE particulars. It's not something you'll use often, but it has a couple of options that may be needed in special, or informational situations.

__To view the most current `ace` options and usage, enter:__

```bash
ace help
```

__You can also use `ace h` as a shortcut__. You will be presented with the current list of options available to you.

### Preview Notes

__Removing an ACE container:__

___This is currently a destructive operation at the Preview stage. The container is permanently deleted, but your "app" folder will remain.___

```bash
ace stop <app_name>
```

If you want to remove the "app" folder as well when stopping the container, use:

```bash
ace banish <app_name>
```

___You will not be able to recover your "app" folder if you use `ace banish`___

__Restarting an ACE container:__

___Be aware that Docker reassigns ports when a container is restarted. This is a something that will be addressed in future ACE versions.___

```bash
ace restart <app_name>
```

You can find "app" names by using the `ace apps` option, or by viewing your `/home` directory on the Docker host.

__The default app name on an initial install is oddly enough; `ace`__

*There are many more ACE options you can view by running `ace h` in the terminal.*

---

###Spawn a New ACE Instance

While you can run multiple ACE containers, as a general rule, the less containers running, the more power available to serve clients. Unless you need concrete separation between two clients -- or closed all of your ACE containers -- you should not have much need for this command.

___If more containers are needed, run no more than 2-3 max for 512MB of system memory.___

```bash
ace spawn <app_name>
```

__ACE instances must have unique names or they will not be created! You can reuse a name by issuing the `ace stop <app_name>` command first, essentially removing that instance name from the system.__

### Module Package Manager

___This is an experimental feature, and might not work at some times during the Preview.___

There are currently four test packages for the Preview.

| Module | Description|
|--------|------------|
| __data__ | Shows an example using the Object storage.
| __tools__ | Shows an example of using external modules.
| __user__ | Shows HTML template display and auth concept.
| __echo__ | The default echo test.

__On some modules, you can view test endpoints after install at /mod_name/\_doc__

To `add` or `remove` modules from your app, you use `ace`:

###Adding the 'data' module to the 'ace' app

```bash
ace add ace data
```

> Params: __add__ *app_name* *module_name*

###Removing a module from the 'ace' app

```bash
ace remove ace echo
```

> Params: __remove__ *app_name* *module_name*

__View `ace h` for more ACE MPM options.__

*You can remove the pre-installed 'echo' module after confirming your instance.*

# ~ ACE Raspberry PI ~

See [Coronium Ace RPi](ace_pi.md)

---

# ~ ACE Client Starters ~

You might be asking yourself, "Where are the client SDKs?" And the answer is, there are none. Coronium ACE is built as a toolkit, and provides structured flexibility, allowing you to create whatever client works best for you and your platform.

By using the standards of HTTP and JSON, nearly any programming language can communicate with the ACE api. To build a client your platform only needs to support the following:

- An HTTP client library with GET and/or POST abilities.
- A JSON data encoder / decoder library.

---

Here are some "starter" examples for some programming environments.

### AngularJS
__An AngularJS Sample using [$http service](https://docs.angularjs.org/api/ng/service/$http) (JS)__

```javascript

//--other code here

$http.get("http://my.docker.host:12345/users/hello?username=Chris")
.then( function( res ) {
  console.log( res.data.greeting )
}, function( err ) {
  console.log( err )
})


//--other code here

```

---

### Corona SDK
__A Corona SDK Sample using [network.request](https://docs.coronalabs.com/api/library/network/request.html) (Lua)__

```lua
local json = require('json')

local function onReponse( evt )
  if not evt.isError then
    local data = json.encode( evt.response )
    print( data.greeting )
  else
    print( 'network error' )
  end
end

local url = "http://my.docker.host:12345/users/hello?username=Chris"
local method = "GET"
local params =
{
  headers =
  {
    ["X-Auth-Token"] = "12345"
  }
}

network.request( url, method, onResponse, params )
```

---
### Python 3
__A Python sample using [HTTPConnection](https://docs.python.org/3.1/library/http.client.html#examples)__ (Python)

```python
import http.client
conn = http.client.HTTPConnection("http://my.docker.host:12345")
conn.request("GET", "/users/hello?username=Chris")
r1 = conn.getresponse()
data1 = r1.read()
print(data1.result.greeting)
conn.close()
```

---

### JQuery
__A JQuery sample using [$.getJSON](https://api.jquery.com/jQuery.getJSON/) (JS)__

```javascript
// --some code

$.getJSON( "http://my.docker.host:12345/users/hello", { username: "Chris", time: "2am" } )
  .done(function( json ) {
    console.log( "JSON Data: " + json.result.greeting );
  })
  .fail(function( jqxhr, textStatus, error ) {
    var err = textStatus + ", " + error;
    console.log( "Request Failed: " + err );
});

// --some code
```

---

### Simple Website
__As a simple website with templates (html)__

```lua
--== home/app/site/api.lua

local api = api.ace()
--path: /site/index
function api.get.index( in_data )
  local html_tpl = ace.tpl.render( 'index.html', { title = "Super Site", menu={"home","about"}} )
  return html_tpl, ace.HTML
end

--path: /site/about
function api.get.about( in_data )
  local html_tpl = ace.tpl.render( 'about.html', {})
  return html_tpl, ace.HTML
end

return api
```
The structure for the "site" above would look like:

```bash
home
  /app_name
    /site
      api.lua
      /tpl
        index.html
        about.html
```
