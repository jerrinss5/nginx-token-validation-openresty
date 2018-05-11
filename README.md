# nginx-token-validation-openresty

This is just an implementation referenced of https://github.com/zmartzone/lua-resty-openidc

The above link goes in detail about different ways to validate token at Nginx openresty framework using lua script

My implementation is just an extension of that, using the hardcoded public key approach.

Advantage:
1. Token validation would happen even before it reaches the application.
2. If the token is invalid the application would not have to worry about the connection, freeing it of other important stuff.
3. Nginx openresty using lua script can do coarse-grained access control at the load-balancing or reverse proxy end.
4. Anytime a new application needs to be spawned behing the nginx server, it won't need to implement token validation.
5. Single point of configuration changes for token validation

Steps:
1. Run the container off the image: docker run -d --name=token_validation -p 8181:8080 openresty/openresty:trusty
2. Bash into the container to modify the conf file: docker exec -it token_validation /bin/bash
3. Add the configuration file as seen in sample in location: /usr/local/openresty/nginx/conf/
4. Add lua idc module inside the container: luarocks install lua-resty-openidc
5. Reload and token validation is active.

Some points:
lua_shared_dict introspection 10m;
- determines how long the token validation result would be cached in the in-memory database

secret = [[-----BEGIN CERTIFICATE-----
                        -----END CERTIFICATE-----]]
- it is the public key of the kid issued in the token for the user or the application to be verified against

local res, err = require("resty.openidc").bearer_jwt_verify(opts)
- call bearer for OAuth 2.0 JWT validation

TODO:
. Scope value to be verified against can be an array value, and would need to be looped over to validate

In Depth understanding: https://github.com/zmartzone/lua-resty-openidc
