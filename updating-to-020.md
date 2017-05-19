# Updating to Version 0.20

Komada version 0.20 makes a few important modifications that will require you to change your code. This article gives you all the details on what needs to be changed in your code to update. Don't worry, there's no *that* much to do!

## Changing app.js

Because 0.20 now generates a class instead of just a module, you will need to change your main script (generally, `app.js`) with the new initialize method: 

```js
const Komada = require('komada');
const client = new Komada({
  "ownerID" : "insert-owner-id-here",
  "clientID": "insert-client-id-here",
  "prefix": "!",
  "clientOptions": {
    "disableEveryone": true
  }
});

client.login("insert-bot-token-here");
```

See? Nothing to it!

## Editing commands to be async

Because of the new **finalizer** feature, all commands will need to be set to use `async`. The change is super easy. Here's an example test command - just add `async` before the exports.run parenthesis!

```js
exports.run = async (client, msg) => {
  msg.channel.send("Test successful!");
};
```

## Log command changed

Previously, you could send a message using the internal `log` function which included a nice color and timestamp. Internal functions have changed, and the way to send logs has moved. You can now send it as an event instead:

```js
client.emit('log', message<String>, type<String:Optional>);
// type can be "debug", "log", "error", or "warn"

//Example:
client.emit("log", "Something's wonky, mate!", "warn");
```

## Using the new `send` wrapper for editable commands

Brand new in this version is the ability to edit commands you've already sent, and the command will be edited to the new command result. For instance, if you send `+help conf` and the bot answers with the conf help, you can then *edit* your message to `+help help` and the bot's answer now reflects the new help requested.

To enable this feature, you must add `"cmdEditing" : true` to the config object:

```js
const client = new Komada({
  "ownerID" : "insert-owner-id-here",
  "clientID": "insert-client-id-here",
  "prefix": "!",
  "cmdEditing" : true,
  "clientOptions": {
    "disableEveryone": true
  }
});
```

If you want this feature to be enabled for your own commands, you will need to use special Komada aliases for sending your message, instead of the default `channel.send()`. These aliases are: 

```js
msg.send("message content"); // extended support, see below
msg.sendMessage("This is a simple message");
msg.sendEmbed(embed);
msg.sendCode("xl", "This is code");
```

> The `msg.send()` method supports the new merged/unified Discord.js method for sending, including embeds, files, etc. For more details, see [Discord.js documentation](https://discord.js.org/#/docs/main/stable/class/TextChannel?scrollTo=send)


## Command cooldown

Each command can now be set with their own individual cooldown. The cooldown will prevent the command from being run again before it runs out. To enable cooldown on any command, simply add the `cooldown: <time in seconds>` property to the command's `exports.conf` object: 

```js
exports.conf = {
  enabled: true,
  cooldown: 60, // Command can only be run every 60 seconds
  runIn: ["text"],
  aliases: [],
  permLevel: 0,
  botPerms: [],
  requiredFuncs: [],
};
```

> While the new cooldown prevents actually executing the command, the bot will still respond to every command with the time remaining, which can lead to a pretty spammy channel if someone attempts to use a command repeatedly. We suggest using cooldowns in combination with a [slowmode inhibitor](https://github.com/dirigeants/komada-pieces/blob/master/inhibitors/commandSlowMode.js) for best results!

## Full Changelog

The complete changelog is available [here](https://github.com/dirigeants/komada/blob/indev/CHANGELOG.md)