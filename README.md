## node-eiscp

This is a fork of git://github.com/tillbaks/node-eiscp.git, that removes the command mapping.  I found this caused the library to
frequencty not work correctly with new receivers.  I'm sure there is value mapping IESCP to less crypt forms but for my use case
that is not the siutation.

### How to use it?

#### Until first release you can add the git url to your package.json like this

```
{
  "dependencies": {
    "eiscp": "git://github.com/jpetrocik/node-eiscp.git"
  }
}
```

#### See the examples
https://github.com/jpetrocik/node-eiscp/tree/master/examples

### Command syntax

You send the IESCP command directly, see https://docs.google.com/spreadsheets/d/1BC932JWJYDB9IABvPjXZ6Zu4wEidP7AoRvnPoTpn-F8/edit?usp=sharing

Set Volume:

eiscp.command("MVL","22");

Power Off

eiscp.command("PWR","00");

### Events

You will to listen to events to use this module.

#### .on(command, callback)

Data from connected receiver.

**command:** _(string)_ - Any supported command like "volume" or "system-power"

**callback:** _(function)_ - One argument containing the result of the command. If command is volume you would get the volume as a number.

#### .on("data")

Data from connected receiver.

**data:**
- command _(string|array)_ - (array if command has aliases) contains the high-level command
- argument _(string)_ - string contains the high-level argument
- iscp_command _(string)_ - string contains the low-level command

#### .on("connect")

Will fire when connected.

#### .on("close")

Will fire when disconnected.

#### .on("error")

Will fire when error is encountered.

**message:** _(string)_ - contains an error message

#### .on("debug")

Will fire when debug message is encountered. Use this when developing, you will get useful messages for debugging.

**message:** _(string)_ - contains a debug message

### API

#### connect ([ options ])

If you only have one receiver on your network there is no need to provide any options, it will be discovered automatically.

**options:**

- `host` _(default: undefined)_ - Hostname or IP of receiver
- `port` _(default: 60128)_ - Port of receiver
- `reconnect` _(default: true)_ - Reconnect to receiver if connection is lost
- `reconnect_sleep` _(default: 5)_ - Time in seconds between reconnection attempts
- `verify_commands` _(default: true)_ - Whether to verify high-level commands for model compatibility

#### discover (options, callback)

Sends a broadcast packet and waits for response

**options:**

- `address` _(default: "255.255.255.255")_ - Hostname or IP of receiver
- `port` _(default: 60128)_ - Listening port **(NOTE: try changing this if you have trouble connecting)**
- `timeout` _(default: 2)_ - Time in seconds to wait for respoonses after broadcast is sent

**callback:** _(function)_ - You will receive one argument cointaining an array of devices

- `devices` _(array)_ - An array of objects
  - `host` - Receiver IP
  - `port` - Receiver Port
  - `model` - Receiver Model
  - `areacode` - Area Code?
  - `message` - Raw message that was received


#### raw (data [, callback])

Sends a low-level command like _PWR01_ or _MVL1A_

**data:** _(string)_ - Command to send

**callback:**

NOTE: Callback does not tell you if the command was successfull only that it was sent to the receiver.

- `result` _(object)_
  - `result` - `true` or `false` if command was sent to receiver
  - `msg` - If there is a message attached to the result


#### command (command [, callback])

Sends a high-level command like **system-power=query** or **zone2.volume=1A**
more info about command syntax can be found above the API

**command:** _(string)_ - Command to send

**callback:**

NOTE: Callback does not tell you if the command was successfull only that it was sent to the receiver.

- `result` _(object)_
  - `result` - _true_ or _false_ if command was sent to receiver
  - `msg` - If there is a message attached to the result


#### get_commands (zone, callback)

Retreives an array of all commands in the provided zone

**zone:** _(string)_ - zone can be "main", "zone2" etc..

**callback:**

- `commands` _(array)_ - Array of all commands


#### get_command (command, callback)

Retreives an array of all arguments in the provided command

**command:** _(string)_ - command can be "system-power", "zone2.volume" etc.. (NOTE: If you don't specify zone "main" will be assumed)

**callback:**

- `arguments` _(array)_ - Array of all arguments


### Info about yaml/json commands stuff

All commands are in a json file "eiscp-commands.json" and are converted from a YAML file from https://github.com/miracle2k/onkyo-eiscp thanks dude!

###### Download YAML file:
```
wget https://raw.github.com/miracle2k/onkyo-eiscp/master/eiscp-commands.yaml
```

###### Convert YAML file to json:
Run in node-eiscp directory. This will create "eiscp-commands.json". Warning this script might be badly coded. [:
```
node eiscp-commands-convert.js
```

### Changes

* (owagner) added new option "verify_commands", defaulting to true, which yields the previous behavior: High-level
  commands are verified for model compatibility, and rejected otherwise. Although in theory a good idea, this makes
  high-level commands unusable for new models which are not yet presented in the yaml file or for which no Onkyo
  documentation is available yet (as the time of this writing, for example, the latest V 1.26 .xls does not cover
  the NR535)
* (owagner) change "reconnect" property to default to true, to match documentation
