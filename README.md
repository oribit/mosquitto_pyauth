mosquitto_pyauth
================

Mosquitto auth plugin that lets you write your auth plugins in Python.

Compiling
=========

You need mosquitto version 1.2.1 or higher.

Make sure you have Python dev package installed (`apt-get install
python-dev` under Debian/Ubuntu).

You must either have mosquitto header (`apt-get install libmosquitto-dev`)files installed globally in
`/usr/include`, etc. or clone this repository at the top of the
mosquitto source directory. Then:

    cd mosquitto_pyauth
    make

Alternatively you can pass full path to mosquitto sources using
`MOSQUITTO_SRC` variable:

    make MOSQUITTO_SRC=/path/to/mosquitto-src

If all goes ok, there should be `auth_plugin_pyauth.so` file in the
current directory. Use `make install` or copy it under path accessible for mosquitto daemon,
e.g.: `/usr/local/lib/mosquitto/`.

### Troubleshooting

If you get errors while compiling the plugin about `-lmosquitto` then you have a missing link to libmosquitto.
Just check the file `/usr/lib/libmosquitto.so` or `/usr/lib/mosquitto.so.1` (also it could be in the directory
`/usr/lib/x86_64-linux-gnu`) exists and create a symlink:

    ln -s /usr/lib/libmosquitto.so.1 /usr/lib/libmosquitto.so

If you get errors while compiling the plugin about `-lcares` then you could have a missing link to libcares.
Just check the file `/usr/lib/libcares.so.2` (also it could be in the directory
`/usr/lib/x86_64-linux-gnu`) exists and create a symlink:

    ln -s /usr/lib/libcares.so.2 /usr/lib/libcares.so

And make again the plugin. This time should work.

Running
=======

Add following line to `mosquitto.conf`:

    auth_plugin /path/to/auth_plugin_pyauth.so

You must also give a pointer to Python module which is going to be
loaded (make sure it's in Python path, use `PYTHONPATH` env variable
to the rescue):

    auth_opt_pyauth_module some_module

Python module
=============

Python module should do required initializations when it's imported
and provide following global functions:

* `plugin_init(opts)`: called on plugin init, `opts` holds a tuple of
  (key, value) 2-tuples with all `auth_opt_` params from mosquitto
  configuration (except `auth_opt_pyauth_module`)

* `plugin_cleanup()`: called on plugin cleanup with no arguments

* `unpwd_check(username, password)`: return `True` if given
  username and password pair is allowed to log in. It's called every time a
  message is published and only one time when the subscriptor subscribes.

* `acl_check(clientid, username, topic, access)`: return `True` if
  given user is allowed to subscribe (`access =
  mosquitto_auth.MOSQ_ACL_READ`) or publish (`access =
  mosquitto_auth.MOSQ_ACL_WRITE`) to given topic (see `mosquitto_auth`
  module below)

* `psk_key_get(identity, hint)`: return `PSK` string (in hex format without heading 0x) if given
  identity and hint pair is allowed to connect else return `False` or `None`

* `security_init(opts, reload)`: called on plugin init and on config
  reload

* `security_cleanup(reload)`: called on plugin cleanup and on config
  reload

Auxiliary module
================

Authentication module can import an auxiliary module provided by mosquitto:

    import mosquitto_auth

The module provides following function:

* `topic_matches_sub(sub, topic)`: it mirrors
  `mosquitto_topic_matches_sub` from libmosquitto C library - the
  function checks whether `topic` matches given `sub` pattern (for
  example, it returns `True` if `sub` is `/foo/#` and `topic` is
  `/foo/bar`) and is mostly useful is `acl_check` function above

The following constants for `access` parameter in `acl_check` are
provided:

* `MOSQ_ACL_NONE`

* `MOSQ_ACL_READ`

* `MOSQ_ACL_WRITE`
