---
layout: post
title: Voice Recognition Enabled Discord Bot
excerpt: "Mainly used for trolling"
categories: [tech]
comments: true
image:
  feature: ../img/discord.png
  credit:
  creditlink:
---

# Existing Solutions

After much searching on the internet, I couldn't find that many guides for writing decent voice-enabled 
[Discord bots](https://discordbots.org/). Unfortunately, at the time of writing this, the 
[guide](https://refruity.xyz/writing-discord-bot/) I primarily followed seems to be down.

After going through several code bases 
(
[1](https://github.com/dtinth/discord-transcriber),
[2](https://github.com/ReFruity/EzBot)
), I've managed to put together a bot of my own that I find to be pretty clean. Here's how you can do it.

# Outline
1. Programming the Bot
    - Pre-Requisites
        - Node
        - File Hierarchy
        - Installing Node Modules
        - Final Configurations
    - Programming the Bot
        - Hooking up a Command / Event Handler
        - Handling Actual Events
        - Capturing Guild Member Audio
        - Capturing Text Commands
2. Conclusion

# Programming The Bot
## Pre-Requisites
### Node
First and foremost, we need to make sure that we have the correct version of node installed. For my bot, I used 
v10.16.10. You can check your versions from the following commands:

```
$ node -v
v10.16.10
```

### File Hierarchy

Start a new project and create the following files / folders 
```
rin/
    /audio/
    /commands/
        - ping.js
    /events/
        - guildMemberSpeaking.js
        - message.js
        - ready.js
    /promised/
        - Dispatcher.js
    config.json
    index.js
```

**Note!!** If you choose to enable voice recognition for your bot (which is honestly the whole point of this guide), then we need 
to grab one more file from the internet. I won't outline the steps, but they can be easily followed and found [here](https://cloud.google.com/speech-to-text/docs/quickstart-protocol)
under the "Before you begin" section. All you need to do is follow those steps until you get a google credentials file.
I forgot the default name, but we'll use `google-credentials.json` for the purposes of this guide. 
{: .notice}

Go ahead and place that file anywhere. I put it in my project root, so now my file hierarchy looks like this:
```
rin/
    /audio/
    /commands/
        - ping.js
    /events/
        - guildMemberSpeaking.js
        - message.js
        - ready.js
    /promised/
        - Dispatcher.js
    config.json
    index.js
    google-credentials.json
```


### Installing Node modules

Copy and paste the following into your `package.json`. Don't forget to change things
like the `name` or `description`. Feel free to do this at the end of the guide as well.  

```json
{
  "name": "rin",
  "version": "1.0.1",
  "description": "Discord bot",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "lint": "./node_modules/.bin/eslint .",
    "lint-fix": "./node_modules/.bin/eslint . --fix",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "pre-commit": [
    "lint"
  ],
  "author": "bryngo",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/bryngo/rin/issues"
  },
  "homepage": "https://github.com/bryngo/rin/blob/master/README.md",
  "dependencies": {
    "@google-cloud/speech": "2.1.1",
    "discord.js": "github:discordjs/discord.js",
    "dotenv": "^6.1.0",
    "enmap": "^5.1.0",
    "ffmpeg": "^0.0.4",
    "i18n": "^0.8.3",
    "mongodb": "^3.1.8",
    "node-opus": "^0.3.2",
    "opusscript": "0.0.6",
    "pino": "^5.13.0",
    "pino-pretty": "^3.2.0"
  },
  "devDependencies": {
    "eslint": "^5.8.0",
    "eslint-config-standard": "^12.0.0",
    "eslint-plugin-import": "^2.14.0",
    "eslint-plugin-node": "^8.0.0",
    "eslint-plugin-promise": "^4.0.1",
    "eslint-plugin-standard": "^4.0.0",
    "pre-commit": "^1.2.2"
  }
}
```

Now, make sure your in the project root and run `npm install` from the command line. This might take a while.

Note that at the time of writing, the version of discord.js installed is discord.js@12.0.0-dev. It's the
newest "production worthy" push that hasn't been shipped out yet. I used it for my bot because I found a lot of voice 
and audio improvements from it. 
{: .notice}

There shouldn't be any problems installing all of the node modules. There should now be a folder called `node_modules`
in your project root. Just make sure you have discord.js@12.0.0-dev
installed (or maybe even just discord.js@12.0.0). You can check this by running the following:
```
$ npm list discord.js
...discord.js@12.0.0-dev ...
```

### Final configurations

Copy and paste the following into your `config.json`
```json
{
  "discordApiToken": "{DISCORD API TOKEN HERE}",
  "guildId": "{GUILD ID HERE}",
  "voiceChannelName": "{VOICE CHANNEL HERE}",
  "textChannelName": "{TEXT CHANNEL NAME HERE}",
  "languageCode": "en_US",
  "twice-clip": "audio/twice-bryan-guinn-ezra-brandon.mp3",
  "prefix": "?"
}
```

You can get your `discordApiToken` by 

1. going to the [discord developer portal](https://discordapp.com/developers/applications/), 
2. selecting your bot
3. selecting "Bot"
4. And then copying the token on the right hand side that's hidden by default.

You can get your `guildId` by right clicking a discord server of choice and clicking on `copy ID`. You need to 
enable developer mode in Discord which can also be found in the client itself. 

`voiceChannelName` and `textChannelName` are just the plaintext channel name strings. These will be the channel our
bot listens in and outputs text to.

For the `twice-clip`, that's just a little something I did initially for my bot. Since we'll be using Google's 
Speech to Text API, we'll be looking for a user to say a trigger word, and have our bot play an audio clip in response.
Feel free to replace this audio clip with anything you'd like of course :). Some other clips can be found in my 
[git repo](https://github.com/bryngo/rin). 

`prefix` is just whatever character you want to prepend to your text commands. Choosing `?` means one of my commands will
look like this: `?ping`. 

Almost done with configurations. Lastly, we need to set up an environment variable for the Google Speech to Text API
to work. We'll be using the [dotenv](https://www.npmjs.com/package/dotenv) node module to help us with that. Since we
should have already installed it in the above step, all we have to do is create a file called `.env` and put the following
line in it:
```
GOOGLE_APPLICATION_CREDENTIALS="google-credentials.json"
``` 

The value of `GOOGLE_APPLICATION_CREDENTIALS` should just be the relative file path to the google credentials file you 
got from the internet. Though we will never explicitly access this environment variable, the Google Speech to Text API
knows where to find it. 

## Programming the Bot 
### Hooking up a Command / Event Handler
Hopefully, you installed everything with 0 problems (usually unlikely in my experience). Copy and paste the following into
`index.js` 

```js
require('dotenv').config();
const Discord = require('discord.js');
const config = require('./config');
const fs = require('fs');
const Enmap = require("enmap");

const discordClient = new Discord.Client();

// read in all of our configurations
discordClient.config = config;

// link all the events
// explanation for how this works can be found here:
// https://anidiots.guide/first-bot/a-basic-command-handler
fs.readdir("./events/", (err, files) => {
    if (err) return console.error(err);
    files.forEach(file => {
        const event = require(`./events/${file}`);
        let eventName = file.split(".")[0];
        discordClient.on(eventName, event.bind(null, discordClient));
    });
});

discordClient.commands = new Enmap();

// read in all the custom commands
fs.readdir("./commands/", (err, files) => {
    if (err) return console.error(err);
    files.forEach(file => {
        if (!file.endsWith(".js")) return;
        let props = require(`./commands/${file}`);
        let commandName = file.split(".")[0];
        discordClient.commands.set(commandName, props);
    });
});


discordClient.login(config.discordApiToken);
```

There's quite a few things going on here, and I won't take too much time explaining it, but we're essentially hooking up
a pretty clean and simple event / command handler for our bot. This keeps our files a lot more organized. 

Additionally, we login to our server. 

### Handling Actual Events
A discord client has many, many [events](https://discord.js.org/#/docs/main/master/class/Client). We'll be focusing on 
`guildMemberSpeaking`, `message`, and `ready`. In `events/ready.js`, paste the following:

```js
module.exports = async (client) => {
    console.log(`Ready to serve in ${client.channels.size} channels on ${client.guilds.size} servers, for a total of ${client.users.size} users.`);

    console.log(`Logged in as ${client.user.tag}!`);

    const guild = client.guilds.get(client.config.guildId);
    if (!guild) {
        throw new Error('Cannot find guild.')
    }
    const voiceChannel = guild.channels.find(ch => {
        return ch.name === client.config.voiceChannelName && ch.type === 'voice'
    });
    if (!voiceChannel) {
        throw new Error('Cannot find voice channel.')
    }
    console.log(`Voice channel: ${voiceChannel.id} ${voiceChannel.name}`);

    const textChannel = guild.channels.find(ch => {
        return ch.name === client.config.textChannelName && ch.type === 'text'
    });
    if (!textChannel) {
        throw new Error('Cannot find text channel.')
    }
    console.log(`Text channel: ${textChannel.id} ${textChannel.name}`);

    client.voiceConnection = await voiceChannel.join();

};
```

We're basically just joining the voice channel here whenever our bot is ready. We should be able to run this and see
console output now. To do so, simply run `npm start` on the command line. 

**Note** The file naming for events and commands are very important. To capture a particular event, the file **must**
be named that event. For instance, to capture the `guildMemberSpeaking` event, the file must be named `guildMemberSpeaking.js`. 
{: .notice}

### Capturing Guild Member Audio
In `events/guildMemberSpeaking.js`, add the following lines:

```js

const { Transform } = require('stream');

const Dispatcher = require('../promised/Dispatcher');
const googleSpeech = require('@google-cloud/speech');
const googleSpeechClient = new googleSpeech.SpeechClient();

module.exports = async (client, member, speaking) => {

    if (!speaking || !client.speechEnabled) return;

    console.log(`I'm listening to ${member.displayName}`);

    const voiceConnection = client.voiceConnection;
    const receiver = voiceConnection.receiver;

    // this creates a 16-bit signed PCM, stereo 48KHz stream
    const audioStream = receiver.createStream(member, {mode: "pcm"});
    const requestConfig = {
        encoding: 'LINEAR16',
        sampleRateHertz: 48000,
        languageCode: 'en-US'
    };
    const request = {
        config: requestConfig
    };
    const recognizeStream = googleSpeechClient
        .streamingRecognize(request)
        .on('error', console.error)
        .on('data', async response => {
            const transcription = response.results
                .map(result => result.alternatives[0].transcript)
                .join('\n')
                .toLowerCase();

            console.log(`Transcription: ${transcription}`);

            // play an audio file if keyword is detected
            if (transcription.includes("twice")) {
                await Dispatcher.playFile(voiceConnection, client.config["twice-clip"]);
            }
        });

    const convertTo1ChannelStream = new ConvertTo1ChannelStream();

    audioStream.pipe(convertTo1ChannelStream).pipe(recognizeStream);

    audioStream.on('end', async () => {
        console.log(`I'm done listenting to ${member.displayName}`);
    })
};

function convertBufferTo1Channel(buffer) {
    const convertedBuffer = Buffer.alloc(buffer.length / 2);

    for (let i = 0; i < (convertedBuffer.length / 2) - 1; i++) {
        const uint16 = buffer.readUInt16LE(i * 4);
        convertedBuffer.writeUInt16LE(uint16, i * 2)
    }

    return convertedBuffer
}

class ConvertTo1ChannelStream extends Transform {
    constructor(source, options) {
        super(options)
    }

    _transform(data, encoding, next) {
        next(null, convertBufferTo1Channel(data))
    }
}
```

In `promised/Dispatcher.js`, paste the following lines:

```js
async function playFile(connection, filePath) {
  return new Promise(async (resolve, reject) => {
    const dispatcher = await connection.play(filePath)
    dispatcher.setVolume(1);
    dispatcher.on('end', () => {
      resolve()
    });
    dispatcher.on('error', (error) => {
      reject(error)
    })
  })
}

module.exports = { playFile };
```

At this point, your bot should be able to transcribe messages! Start your bot with `npm start` and watch it transcribe.
Make sure you're in the same voice channel as it, of course. 

### Capturing Text Commands
In `events/message.js` paste in the following lines.

```js
module.exports = (client, message) => {

    // Ignore all bots
    if (message.author.bot) return;

    // Ignore messages not starting with the prefix (in config.json)
    if (message.content.indexOf(client.config.prefix) !== 0) return;

    // Our standard argument/command name definition.
    const args = message.content.slice(client.config.prefix.length).trim().split(/ +/g);
    const command = args.shift().toLowerCase();

    // Grab the command data from the client.commands Enmap
    const cmd = client.commands.get(command);

    // If that command doesn't exist, silently exit and do nothing
    if (!cmd) return;

    // Run the command
    cmd.run(client, message, args);
};
```

If you recall, we hooked up event / command handler in `index.js`. This event will detect whether or not a user entered
a command (by checking for the prefix) and run a specific file in the `events` folder. For simplicity, we'll implement
a ping command. 

In `commands/ping.js` paste in the following lines.
```js
exports.run = (client, message, args) => {
    message.channel.send(`pong! ${args}`).catch(console.error);
};
```

Pretty simple! All commands will take the same form. The file name will be the name of the command, and the function 
that's ran will have the same signature as above. 


# Conclusion
Now you should have a fully function bot that's able to capture audio and textual input! Please feel free to contact me
if you have any questions.
