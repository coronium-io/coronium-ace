# Coronium ACE

__A tiny Lua Server+API aimed at rapid service development, from IoT to full stack applications.__

---

*Preview 1 (2015.10)*

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

## Or the "hard" way...

```lua
-- 'echo' Lua module

local api = ace.api()

function api.get.test( in_data )
  return in_data
end

return api
```

As you can see, working with ACE is *very* approachable.
