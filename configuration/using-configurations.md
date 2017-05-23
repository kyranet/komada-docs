# Using Configurations

Komada comes with a fully useable configuration system that requires no fiddling around with databases or setups. All you need to do is to start using it!

## How are confs saved?

Configurations with Komada are saved inside of JSON files in the `./bwd/conf/` folder created the first time it runs. It consists of `default.json` which holds all the default values, as well as one JSON file for each server which is `<serverID>.json`

#### Aren't JSON files prone to corruption?

If used incorrectly, yes. JSON files can fail when they are constantly under assault with I/O operations - when you are reading and writing to and from them all the time. We don't do that - each configuration is loaded to memory on boot-up, and written only when a configuration actually changes. Which, under normal operations, doesn't actually happen that often. Since confs are per-server, a single file is only changed when one server changes its config. 

> In the very ultimate worst case scenario, if one file becomes corrupted, it'll be for a specific guild, and that will cause that *one* guild's configuration to reset. Unless the file system itself becomes corrupted (the `./bwd/conf` folder is lost) there is no chance of a global failure. Unlike databases. 

#### But isn't it inefficient and slow compared to Databases?

Not the way we do it. When a configuration is "requested" we are essentially loading the "Default" configuration, and overwriting guild-specific keys only when they are present. In fact, if a guild does not change any configuration, the conf file isn't written for that guild. When a single key is changed for one guild, **only** that key is written to the JSON. So in essence, it's *more* efficient than a database where each key is present in the DB, whether or not it has contents. 

## Adding a new configuration key

To add a new configuration key, you can simply run the following little bit of code:

```js
client.funcs.confs.addKey("key_name", defaultvalue, keytype);
// key_name must be a string
// defaultvalue must correspond to the keytype
// keytype defaults to "String"
```

Keys can be of 4 different data types: 

* [`String`](configuration/string-types.md) : A regular JavaScript String. Will be converted to string when inserted into the configurations.
* [`Boolean`](configuration/boolean-types.md) : A true/false value. Can be `true` or `false`.
* [`Number`](configuration/number-types.md) : Any valid JavaScript number type.
* [`Array`](configuration/array-types.md) : Creates an array, in which any value can be added or removed.

### Checking if a key exists

Here's an example of code that can be added to the `init` function of a command, monitor or inhibitor, that will add the key if it does not exist:

```js
  let conf = client.funcs.confs.get(msg.guild);
  if(!conf.ban_level) {
    conf.ban_level = 10;
    client.funcs.confs.addKey("ban_level", 10);
  }
```

This not only makes sure the key exists but, the first time, will also make sure your command can run using the default value. 

## Changing the default value

// FIX ME //

## Reading Keys

Configurations can be read in 2 different ways. First, from *any* piece you can access a guild's configuration by using the following command:

```js
let conf = client.funcs.confs.get(guild);
```

The other way is specifically in messages. Any message will have access to the configuration for a guild directly through the use of `msg.guildConf`

Both of those method will yield the exact same result, which is an Object which contains all of the combined default keys + guild overrides. The object looks something like this:

```json
{ prefix: 'g!',
  disabledCommands: [],
  modRole: 'Mods',
  adminRole: 'Devs',
  lang: 'en',
  modLog: 'test-modlog',
  ban_level: 10,
  modLog-embed: true }
```

So to access the `ban_level` item, you could simply refer to `msg.guildConf.ban_level` , or to `client.funcs.confs.get(guild).ban_level`.

## Modifying Key Values

Writing a key value to the database depends on the key type. 

* `String`: `msg.guildConf.keyName.set("new value");`
* `Boolean`: `msg.guildConf.keyName.toggle();`
* `Number`: `msg.guildConf.keyName.set(10);` (where `10` is just an example number, any number will do)
* `Array`: `msg.guildConf.keyName.push("newvalue");` (where `newvalue` is *any* valid JavaScript value of any type)

Komada makes no effort to convert these values for you. If you attempt to set a String key to a numerical value, or set() a boolean to another value, your code will fail.

## Resetting a key to default values

In the case where a configuration has been modified to a specific value, it's possible to remove the override for that guild and reset it to the default value. This essentially removes the key from the guild's JSON config, which means if the key's global default is modified, that new default will be used in the guild.

```js
client.guildConfs.get(guild).reset("keyname");
```

## Global Configuration Keys

> This feature is new in 0.20 and higher.

Global configurations 

```js
client.settings.add(key, value, { type, global, min, max, possibles });
```

