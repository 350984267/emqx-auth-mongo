
emqttd_plugin_mongo
===================

Authentication with MongoDB

Build
-----

This project is a plugin for emqttd broker. In emqttd project:

If the submodule exists:

```
git submodule update --remote plugins/emqttd_plugin_mongo
```

Orelse:

```
git submodule add https://github.com/emqtt/emqttd_plugin_mongo.git plugins/emqttd_plugin_mongo

make && make dist
```

Configuration
-------------

File: etc/plugin.config

```erlang
[
  {emqttd_plugin_mongo, [

    {mongo_pool, [
      {pool_size, 8},
      {auto_reconnect, 3},

      %% Mongodb driver opts
      {host, "localhost"},
      {port, 27017},
      %% {login, ""},
      %% {password,""},
      {database, "mqtt"}
    ]},

    %% Superuser Query
    {superquery, [
        {collection, "mqtt_user"},
        {super_field, "is_superuser"},
        {selector, {"username", "%u"}}
    ]},

    %% Authentication Query
    {authquery, [
        {collection, "mqtt_user"},
        {password_field, "password"},
        %% Hash Algorithm: plain, md5, sha, sha256, pbkdf2?
        {password_hash, sha256},
        {selector, {"username", "%u"}}
    ]},

    %% ACL Query: "%u" = username, "%c" = clientid
    {aclquery, [
        {collection, "mqtt_acl"},
        {selector, {"username", "%u"}}
    ]},

    %% If no ACL rules matched, return...
    {acl_nomatch, deny}

  ]}
].
```

Load Plugin
-----------

```
./bin/emqttd_ctl plugins load emqttd_plugin_mongo
```

MongoDB Database
----------------

```
use mqtt
db.createCollection("mqtt_user")
db.createCollection("mqtt_acl")
db.mqtt_user.ensureIndex({"username":1})
```

mqtt_user Collection
--------------------

```
{
    username: "user",
    password: "password hash",
    is_superuser: int (1 true, 0 false),
    created: "datetime"
}
```

mqtt_acl Collection
-------------------

```
{
    username: "username",
    clientid: "clientid",
    publish: ["topic1", "topic2", ...],
    subscribe: ["subtop1", "subtop2", ...],
    pubsub: ["topic/#", "topic1", ...]
}
```

License
-------

Apache License Version 2.0

Author
------

feng at emqtt.io

