# Openresty

## Redis

```nginx
worker_processes 1;
error_log logs/error.log;
events {
    worker_connections 1024;
}

http {
    server {
        listen 8080;
        location / {
            default_type appcation/json;
            access_by_lua_file lua/test.lua;
        }
    }
}
```

```lua
local headers, err = ngx.req.get_headers()
local key = headers["key"]
if not key then
    ngx.log(ngx.ERR, "no key found")
    return ngx.exit(400)
end

local redis = require "resty.redis"
local red = redis:new()

red:set_timeout(1000) -- 1 second

local ok, err = red:connect("127.0.0.1", 6379)
if not ok then
    ngx.log(ngx.ERR, "failed to connect to redis: ", err)
    return ngx.exit(500)
end

local value, err = red:get(key)
if not value then
    ngx.log(ngx.ERR, "failed to get redis key: ", err)
    return ngx.exit(500)
end

if value == ngx.null then
    ngx.log(ngx.ERR, "no value found for key ", key)
    return ngx.exit(400)
end

local result_json = {}

result_json[key] = value

local cjson = require "cjson"

ngx.say(cjson.encode(result_json))
```

## 获取 body

```lua
ngx.req.read_body()
local data = ngx.req.get_body_data()
if not data then
    ngx.say("no body data found")
    return
end
ngx.say(data)
```

## 动态proxy

```nginx
worker_processes 1;
error_log logs/error.log info;
events {
    worker_connections 1024;
}

http {
    server {
        listen 8080;
        location / {
            set $target '127.0.0.1:10000';
            access_by_lua_file lua/test.lua;
            proxy_pass http://$target;
        }
    }

    server {
        listen 10000;
        location / {
            default_type text/plain;
            return 200 "default backend";
        }
    }

    server {
        listen 10001;
        location / {
            default_type text/plain;
            return 200 "10001";
        }
    }

    server {
        listen 10002;
        location / {
            default_type text/plain;
            return 200 "10002";
        }
    }
}
```

```lua
local headers, err = ngx.req.get_headers()
local host = headers["Host"]
if not host then
    ngx.log(ngx.ERR, "host: ", host)
    return ngx.exit(500)
end

local hosts = {}
hosts["10001.test.com"] = "127.0.0.1:10001"
hosts["10002.test.com"] = "127.0.0.1:10002"

local server = hosts[host]
if not server then
    return
end
ngx.var.target = server
```
