# Coronium ACE

__A tiny Lua Server+API aimed at rapid service development, from IoT to full stack applications.__

<a href="https://twitter.com/share" class="twitter-share-button" data-via="develephant" data-size="large" data-hashtags="coroniumace">Tweet</a>
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0],p=/^http:/.test(d.location)?'http':'https';if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src=p+'://platform.twitter.com/widgets.js';fjs.parentNode.insertBefore(js,fjs);}}(document, 'script', 'twitter-wjs');</script>

---

*Alpha (2015.11)*

__Coronium ACE__ is a *highly-opinionated* server and framework for rapid development of REST/API like services using __Lua__. Though breaking with convention, it creates a simplified interface to the __Coronium ACE__ API. Hopefully you'll find this reductionism refreshing to develop with.

The __Coronium ACE__ binary (including server) weighs in under 40 MB. It's __Docker__ powered, so your options are wide open for development, deployment, security, and scalability.

There is also a special ARM version for Docker on the __Raspberry PI__.

---

## An echo API in __1__ step.

```bash
ace add ace echo
```

Visit the newly created endpoint and pass some data.

__API Endpoint:__
```bash
http://your.ace.host:12345/echo/test?ace=rocks&bigfoot=lives
```

__JSON Output:__ `{"ace":"rocks","bigfoot":"lives"}`

*Okay, so we kind of cheated by using the ACE package manager. But, creating an 'echo' module is pretty easy...*

```lua
-- 'echo' Lua module

local api = ace.api()

function api.get.test( in_data )
  return in_data
end

return api
```

As you can see, working with ACE is *very* approachable.
