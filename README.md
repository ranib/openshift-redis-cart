OpenShift Redis Cartridge
=========================
Redis - 3.2.9
-------------

Runs [Redis](http://redis.io) on [OpenShift](https://openshift.redhat.com/app/login) using downloadable cartridge support.  To install to OpenShift from the CLI (you'll need version 1.9 or later of rhc), create your app and then run:

    rhc add-cartridge http://cartreflect-claytondev.rhcloud.com/reflect?github=ranib/openshift-redis-cart

Any log output will be generated to $OPENSHIFT_REDIS_DIR/logs/redis.log

Supports clustering and persistence, with monitoring from Redis Sentinel.  Does not automatically pull the latest security updates - please monitor the Redis upstream 2.6 branches.


How it Works
------------

`conf/redis.conf.erb` is run before Redis starts and generates a configuration file.  It will attempt to load the file `.openshift/redis.conf` if it exists.  The default configuration will snapshot the db to disk into the <code>$OPENSHIFT_DATA_DIR/.redis/dbs</code> directory.

At cart creation, a unique password will be generated to the environment variable `REDIS_PASSWORD` - the server will require this to connect.

To access `redis-cli` from the SSH session, use the $REDIS_CLI environment variable to get the correct config on the gear

    $ ssh <gear with redis>
    Connectiing to....
    $ redis-cli $REDIS_CLI
    127.0.32.124 6379>

To connect to Redis from your local shell, run the <code>cartridge-status</code> command to get the port and authorization info:

    $ rhc cartridge-status redis -a <yourapp>

    RESULT:
    Redis is running
      master (receives writes), mode sharded
      Connect to: 343928-abc.dev.rhcloud.com:35546 password: 80o9euk80oeu90834

and then run <code>rhc port-forward</code> to your app:

    $ rhc port-forward <yourapp>
    ....
    redis   127.0.0.1:35546   => 343928-abc.dev.rhcloud.com:35546

In another shell window, run <code>redis-cli</code>

    $ redis-cli -p 35546 -a 80o9euk80oeu90834
    redis 127.0.0.1:35546>

A number of Redis configuration values can be tuned via environment variables:

*  <code>REDIS_PASSWORD</code>

   The password for accessing Redis.

*  <code>REDIS_MAXMEMORY</code>

   The maximum memory this instance will allow.  No default value.

*  <code>REDIS_APPENDONLY</code>

   The value of <code>appendonly</code> in redis.conf.  Defaults to 'no'

*  <code>REDIS_APPENDFSYNC</code>

   The value of <code>appendfsync</code> in redis.conf.  Defaults to 'everysec'

Always restart each gear after setting these environment variables.


Upgrading
---------

If you install this cartridge from source, you will be using a precompiled version of Redis 3.2.9 for RHEL6.  You can run the <code>bin/control update</code> script on each gear to build and update to the latest version of the Redis 3.2.9 tree.

    $ rhc ssh <yourapp> --gears 'cd redis && ./bin/control update'
    $ rhc restart-cartridge redis -a <yourapp>'

We hope to add a more natural update process at a later point.

[more about this cartridge see here](https://github.com/smarterclayton/openshift-redis-cart "more about this cartridge")

## If you want to use Redis for Object cache in Wordpress
### 1. add redis to <yourapp>
```BASH
$ rhc cartridge add -a <yourapp> http://cartreflect-claytondev.rhcloud.com/reflect?github=ranib/openshift-redis-cart
```
### 2. edit wp-config.php for Wordpress to connect to Redis
```BASH
/** Enable Redis Cache Settings */

define('WP_CACHE_KEY_SALT', getenv('OPENSHIFT_APP_DNS'));
define('WP_REDIS_CLIENT', 'WP-CLI');
define('WP_REDIS_HOST', getenv('OPENSHIFT_REDIS_HOST'));
define('WP_REDIS_PORT', getenv('OPENSHIFT_REDIS_PORT'));
define('WP_REDIS_PASSWORD', getenv('REDIS_PASSWORD'));
define('WP_REDIS_MAXTTL', '86400');
define('WP_CACHE', true);
```
### 3. How to use plugin
* install `Redis Object Cache` plugin
* Acticate plugin and go to its settings
* click on `Enable Object-Cache` you must see Status as `Connected`
* if not go to `Redis Object Cache` plugin homepage for instructions
* click `Diagnostics` and see connection parameters
* make sure `object-cache.php` file created by this plugin is seen in `Drop-In` under `Installed Plugins`
* if not manually copy `object-cache.php` file from `/plugins/redis-cache/includes/` directory to `/wp-content/` directory
