# Botkit Core

Table of Contents

* [The Botkit Controller](#the-botkit-controller)
* [Receiving Messages](#receiving-messages-and-events)
* [Sending Messages](#sending-messages)
* [Multi-message Conversations](#multi-message-conversations)
* [Controller Methods List](#botkit-controller-object)
* [Bot Instance Methods List](#botkit-bot-instance-objects)
* [Conversation Methods List](#botkit-conversation-objects)


## The Botkit Controller

The robot brain inside every Botkit applications is the `controller`, an interface that is used to define all the features and functionality of an app. Botkit's core library provides a platform-independent interface for sending and receiving messages so that bots on any platform can be created using the same set of tools.

By attaching event handlers to the controller object, developers can specify what type of messages and events their bot should look for and respond to, including keywords, patterns and status events. These event handlers can be thought of metaphorically as skills or features the robot brain has -- each event handler defines a new "When a human says THIS the bot does THAT."

Once created, the controller will handle incoming messages, [spawn bot instances](#controllerspawn) and [trigger handlers](#receiving-messages-and-events).

For each platform, there is a specialized version of the controller object. These specialized controllers customize Botkit's core features to work with the platform, and add additional features above and beyond core that offer developers access platform-specific features.

Each platform has its own set of configuration options - refer to the platform connector docs for details:

* [Web and Apps](readme-web.md#create-a-controller)
* [Slack](readme-slack.md#create-a-controller)
* [Cisco Spark](readme-ciscospark.md#create-a-controller)
* [Microsoft Teams](readme-teams.md#create-a-controller)
* [Facebook Messenger](readme-facebook.md#create-a-controller)
* [Twilio SMS](readme-twiliosms.md#create-a-controller)
* [Twilio IPM](readme-twilioipm.md#create-a-controller)
* [Microsoft Bot Framework](readme-botframework.md#create-a-controller)

Below is an example showing a [Botkit Anywhere](readme-web.md) controller being created.

```javascript
var Botkit = require('botkit');

var controller = Botkit.anywhere(configuration);

// give the bot something to listen for.
controller.hears('hello','message_received',function(bot,message) {
  bot.reply(message,'Hello yourself.');

});
```
## Botkit Statistics Gathering

Versions 0.4 through 0.6.19 of Botkit transmit anonymous usage statistics which were at one time used by the Botkit team at [Howdy](http://howdy.ai) to measure and analyze the Botkit community. This statistic gathering is no longer active, and the endpoint for collecting this information has been disabled. We recommend developers upgrade to the latest version of Botkit, or disable this functionality by specifying the `stats_optout` option described below.

Note: Version 0.6.20 and above no longer transmit this information.

### Opt Out of Stats

To opt out of the stats collection, pass in the `stats_optout` parameter when initializing Botkit,
as seen in the example below:

```javascript
var controller = Botkit.anywhere({
    stats_optout: true
});
```

## Receiving Messages and Events

Once connected to a messaging platform, bots receive a constant stream of events - everything from the normal messages you would expect to typing notifications and presence change events. The set of events your bot will receive will depend on what messaging platform it is connected to.

To respond to events, use [controller.on()](#controlleron) to define a handler function that receives the event details and takes actions.
_Most_ of event handlers will receive 2 arguments - a bot instance, and the event object itself.  

Events received from messaging platforms events undergo a [normalization process](readme-pipeline.md) for use inside Botkit,
so that any type of event can be used with [bot.reply()](#botreply), [bot.startConversation()](#botstartconversation) and similar methods to send replies.

Incoming events will have _at least_ the following fields:

```javascript
{
  type: <type of event>,
  user: <unique id of user who sent the message>,
  channel: <unique id for channel or 1:1 conversation>,
  text: <text of message or primary payload value if present>,
  raw_message: <the original event data>
}
```

Note that Botkit leaves all the native fields intact, so any fields that come in from the platform are still present in the object.
However, our recommendation for accessing any platform-native fields is to use the `message.raw_message` sub-object
which contains an unmodified version of the event data. Before accessing these fields, we recommend that you first check `bot.type` to ensure that the message originated on the appropriate platform.


### Incoming Message Events

| Event | Description
|-- |--
| message_received | In a 1:1 platform, indicates an incoming message from user has been received
| direct_mention | In a multi-user platform, indicates a message was directed at the bot in a group channel "@bot hello"
| mention | In a multi-user platform, indicates a message in a group channel mentioned the bot indirectly "hello @bot"
| ambient | In a multi-user platform, indicates a message in a group channel that does not mention the bot in any way
| direct_message | In a multi-user platform, indicates a message sent privately to the bot

Here is an example of a handler for a message event:

```javascript
controller.on('message_received', function(bot, message) {
  bot.reply(message,'I heard a message');
});
```

### Middleware Error Events

| Event | Description
|-- |--
| ingest_error | An error happend while processing the message in an ingest middleware.
| normalize_error | An error happend while processing the message in a normalize middleware.
| categorize_error | An error happend while processing the message in a categorize middleware.
| receive_error | An error happend while processing the message in a receive middleware.

Here is an example of a handler for an error event:

```javascript
controller.on('receive_error', function(err, bot, message) {
    bot.reply(message, `There was an error processing your request. Please try again later. Error: ${err.toString()}`);
});
```

You can also capture errors in _any_ middleware by handling the `pipeline_error` event:

```javascript
controller.on('receive_error', function(err, bot, message, pipeline_stage) {
    bot.reply(message, `There was an error processing your request. Please try again later. Error: ${err.toString()}`);
});
```



### Conversation Lifecycle Events

| Event | Description
|-- |--
| conversationStarted | A conversation has started. handler should be in the form of function(bot, convo)
| conversationEnded | A conversation has ended. handler should be in the form of function(bot, convo)
| heard-trigger | A trigger defined with [controller.hears()](#controllerhears) was fired. handler should be in the form of function(bot, triggers, message)
| tick | The event loop has ticked. handler does not receive any parameters.

Here is an example of a handler for a message event:

```javascript
controller.on('conversationStarted', function(bot, convo) {
  console.log('A conversation started with ', convo.context.user);
});
```

### Bot Lifecycle events

| Event | Description
|-- |--
| spawned | A bot instance spawned

### Application Events

| Event | Description
|-- |--
| webserver_up | This event occurs when [controller.setupWebserver()](#controllersetupwebserver) has configured a webserver


### Platform Specific Events

For a list of the platform-specific events that Botkit emits, refer to the platform specific docs:

* [Web and Apps](readme-web.md#event-list)
* [Slack](readme-slack.md#event-list)
* [Cisco Spark](readme-ciscospark.md#event-list)
* [Microsoft Teams](readme-teams.md#event-list)
* [Facebook Messenger](readme-facebook.md#event-list)
* [Twilio SMS](readme-twiliosms.md#event-list)
* [Twilio IPM](readme-twilioipm.md#event-list)
* [Microsoft Bot Framework](readme-botframework.md#event-list)


## Matching Patterns and Keywords with `hears()`

In addition to traditional event handlers, Botkit also provides the [controller.hears()](#controllerhears) function,
which configures event handlers that look for specific keywords or phrases in the message.

Each call to `controller.hears()` sets up a separate set of patterns to listen for.
Developers may specify a single pattern to match, or an array of patterns.
By default, Botkit treats these patterns as regular expressions to be evaluated against the `message.text` field in incoming messages.
This behavior can be modified using [controller.changeEars()](#controllerchangeears)

In addition to the array of patterns, `hears()` also receives as an argument one or more event types.
Only events of the type listed will be evaluated.

It is important to note that Botkit will *stop processing handlers* when the first `hears()` trigger is matched.
Triggers are evaluated in the order in which they are defined in the code.
This is a major difference in the way most event handling systems work, which will fire all matching handlers, and differs from handlers
configured with [controller.on()](#controlleron), which behave as expected.

```javascript
controller.hears(['hi','hello','howdy','hey','aloha','hola','bonjour','oi'],['message_received'],function(bot,message) {

  // do something to respond to message
  bot.reply(message,'Oh hai!');

});
```

In many cases, developers will want to expand the capability of Botkit's hearing system
to look for different types of patterns, or to evaluate different fields in the message payload.
There are several ways to achieve this:

* Globally replace the function used to evaluate patterns by using [controller.changeEars()](#controllerchangeears)
* Replace the function on a handler-by-handler basis using [hears middleware functions](middleware.md#hear-middleware)

## Sending Messages

Botkit bots can send messages in several different ways, depending on the type and number of messages that will be sent.

Simple replies requiring only one message in response to an incoming event can be sent using the [bot.reply()](#botreply) function.

Multi-message replies, particularly those that present questions for the end user to respond to,
can be sent using the [bot.startConversation()](#botstartconversation) function and the related conversation sub-functions.

Bots can originate messages - that is, send a message based on some internal logic or external stimulus -
using [bot.say()](#botsay) method.

## Single Message Replies to Incoming Messages

Once a bot has received a message using a [controller.on()](#controlleron) or [controller.hears()](#controllerhears) event handler, a response
can be sent using [bot.reply()](#botreply).

Messages sent using `bot.reply()` are sent immediately. If multiple messages are sent via
`bot.reply()` in a single event handler, they will arrive in the  client very quickly
and may be difficult for the user to process. We recommend using [bot.startConversation()](#botstartconversation)
if more than one message needs to be sent.

You may pass either a string, or a message object to the function.

Message objects may also contain any additional fields supported by the messaging platform in use. Refer to the platform-specific docs for more information.

## Multi-message Conversations

For more complex interactions, multiple messages may be necessary to send a response,
particularly if the bot needs to collect additional information from the user.

Botkit provides a `Conversation` object type that is used to string together several
messages, including questions for the user, into a cohesive unit. Botkit conversations
provide useful methods that enable developers to craft complex conversational
user interfaces that may span a several minutes of dialog with a user, without having to manage
the complexity of connecting multiple incoming and outgoing messages across
multiple API calls into a single function.

Messages sent as part of a conversation are queued and sent in order with a short delay between each message,
which roughly simulates the time it would take for the bot to "type" the message.

### Conversation Threads

While conversations with only a few questions can be managed by writing callback functions,
more complex conversations that require branching, repeating or looping sections of dialog,
or data validation can be handled using feature of the conversations we call `threads`.

Threads are pre-built chains of dialog between the bot and end user that are built before the conversation begins. Once threads are built, Botkit can be instructed to navigate through the threads automatically, allowing many common programming scenarios such as yes/no/quit prompts to be handled without additional code.

You can build conversation threads in code, or you can use [Botkit CMS](https://github.com/howdyai/botkit-cms)'s script management tool to build them in a friendly web environment. Conversations you build yourself and conversations managed in Botkit CMS work the same way -- they run inside your bot and use your code to manage the outcome.

If you've used the conversation system at all, you've used threads - you just didn't know it. When calling `convo.say()` and `convo.ask()`, you were actually adding messages to the `default` conversation thread that is activated when the conversation object is created.

### Automatically Switch Threads using Actions

You can direct a conversation to switch from one thread to another automatically
by including the `action` field on a message object. Botkit will switch threads immediately after sending the message.

```javascript
// first, define a thread called `next_step` that we'll route to...
convo.addMessage({
    text: 'This is the next step...',
},'next_step');


// send a message, and tell botkit to immediately go to the next_step thread
convo.addMessage({
    text: 'Anyways, moving on...',
    action: 'next_step'
});
```

Developers can create fairly complex conversational systems by combining these message actions with conditionals in `ask()` and `addQuestion()`.  Actions can be used to specify
default or next step actions, while conditionals can be used to route between threads.

From inside a callback function, use `convo.gotoThread()` to instantly switch to a different pre-defined part of the conversation. Botkit can be set to automatically navigate between threads based on user input, such as in the example below.

```javascript
bot.createConversation(message, function(err, convo) {

    // create a path for when a user says YES
    convo.addMessage({
            text: 'You said yes! How wonderful.',
    },'yes_thread');

    // create a path for when a user says NO
    convo.addMessage({
        text: 'You said no, that is too bad.',
    },'no_thread');

    // create a path where neither option was matched
    // this message has an action field, which directs botkit to go back to the `default` thread after sending this message.
    convo.addMessage({
        text: 'Sorry I did not understand.',
        action: 'default',
    },'bad_response');

    // Create a yes/no question in the default thread...
    convo.addQuestion('Do you like cheese?', [
        {
            pattern: 'yes',
            callback: function(response, convo) {
                convo.gotoThread('yes_thread');
            },
        },
        {
            pattern: 'no',
            callback: function(response, convo) {
                convo.gotoThread('no_thread');
            },
        },
        {
            default: true,
            callback: function(response, convo) {
                convo.gotoThread('bad_response');
            },
        }
    ],{},'default');

    convo.activate();
});
```

### Special Actions

In addition to routing from one thread to another using actions, you can also use
one of a few reserved words to control the conversation flow.

Set the action field of a message to `completed` to end the conversation immediately and mark as success.

Set the action field of a message to `stop` end immediately, but mark as failed.

Set the action field of a message to `timeout` to end immediately and indicate that the conversation has timed out.

After the conversation ends, these values will be available in the `convo.status` field. This field can then be used to check the final outcome of a conversation. See [handling the end of conversations](#handling-end-of-conversation).

### Using Variable Tokens and Templates in Conversation Threads

Pre-defined conversation threads are great, but many times developers will need to inject dynamic content into a conversation.
Botkit achieves this by processing the text of every message using the [Mustache template language](https://mustache.github.io/).
Mustache offers token replacement, as well as access to basic iterators and conditionals.

Variables can be added to a conversation at any point after the conversation object has been created using the function `convo.setVar()`. See the example below.

```javascript
convo.createConversation(message, function(err, convo) {

    // .. define threads which will use variables...
    // .. and then, set variable values:
    convo.setVar('foo','bar');
    convo.setVar('list',[{value:'option 1'},{value:'option 2'}]);
    convo.setVar('object',{'name': 'Chester', 'type': 'imaginary'});

    // now set the conversation in motion...
    convo.activate();
});
```

Given the variables defined in this code sample, `foo`, a simple string, `list`, an array, and `object`, a JSON-style object,
the following Mustache tokens and patterns would be available:

```
The value of foo is {{vars.foo}}

The items in this list include {{#vars.list}}{{value}}{{/vars.list}}

The object's name is {{vars.object.name}}.

{{#foo}}If foo is set, I will say this{{/foo}}{{^foo}}If foo is not set, I will say this other thing.{{/foo}}
```
Botkit ensures that your template is a valid Mustache template, and passes the variables you specify directly to the Mustache template rendering system.
Our philosophy is that it is OK to stuff whatever type of information your conversation needs into these variables and use them as you please!

### Built-in Variables

Botkit provides several built in variables that are automatically available to all messages:

{{origin}} - a message object that represents the initial triggering message that caused the conversation.

{{responses}} - an object that contains all of the responses a user has given during the course of the conversation. This can be used to make references to previous responses. This requires that `convo.ask()` questions include a keyname, making responses available at `{{responses.keyname}}`

### Included Utterances

| Pattern Name | Description
|--- |---
| bot.utterances.yes | Matches phrases like yes, yeah, yup, ok and sure.
| bot.utterances.no | Matches phrases like no, nah, nope
| bot.utterances.quit | Matches phrases like, cancel, exit, stop

### Conversation Control Functions

In order to direct the flow of the conversation, several helper functions
are provided.  These functions should only be called from within a convo.ask
handler function!

`convo.sayFirst(message)` Works just like convo.say, but injects a message into the first spot in the queue
so that it is sent immediately, before any other queued messages.

`convo.stop()` end the conversation immediately, and set convo.status to `stopped`

`convo.repeat()` repeat the last question sent and continue to wait for a response.

`convo.silentRepeat()` simply wait for another response without saying anything.

`convo.next()` proceed to the next message in the conversation.  *This must be called* at the end of each handler.

`convo.setTimeout(timeout)` times out conversation if no response from user after specified time period (in milliseconds).

### Changing the pace of the conversation with custom per-message delay

Botkit will send each messages in a conversation at the rate dictated by the global tick speed - by default, this is 1.5 seconds, but can be customized using [controller.setTickDelay()](#controllersettickdelay)

It is also possible to set a custom delay on a per-message basis.  To do this, include a `delay` field in the message object.  Note that the value specified will only have an affect if it is greater than the global tick interval.

```
convo.say({text: 'I waited 3 seconds to tell you this...', delay: 3000});
```

### Handling End of Conversation

Conversations trigger events during the course of their life.  Currently,
only two events are fired, and only one is very useful: end.

Conversations end naturally when the last message has been sent and no messages remain in the queue.
In this case, the value of `convo.status` will be `completed`. Other values for this field include `active`, `stopped`, and `timeout`.

```javascript
convo.on('end',function(convo) {

  if (convo.status=='completed') {
    // do something useful with the users responses
    var res = convo.extractResponses();

    // reference a specific response by key
    var value  = convo.extractResponse('key');

    // ... do more stuff...

  } else {
    // something happened that caused the conversation to stop prematurely
  }

});
```

### Handling Conversation Timeouts

If a conversation reaches its timeout threshold (set using `convo.setTimeout()`) while waiting for a user to respond to a `convo.ask()` question, the conversation will automatically end. By default, the conversation will end immediately without sending any further messages. Developers may change this behavior in one of two ways:

*Provide a handler function with convo.onTimeout():*
Use `convo.onTimeout(handler)` to define a function that will be called when the conversation reaches the timeout threshold. This function
can be used to prevent the conversation from ending, or to take some other action before ending such as using `gotoThread()` to  change the direction of the conversation.

Note that functions used with onTimeout must call `gotoThread()`, `next()`, or `stop()` in order for the conversation to continue.

```
convo.onTimeout(function(convo) {

  convo.say('Oh no! The time limit has expired.');
  convo.next();

});
```

*Provide an `on_timeout` conversation thread:*
Instead of providing a function, developers may choose to specify a pre-defined thread to be used in the case of a timeout event.
This thread should be called `on_timeout`.

```
convo.addMessage('Oh no! The time limit has expired.','on_timeout');
convo.addMessage('TTYL.','on_timeout');
```

# Botkit Controller Object

#### controller.hears()
| Argument | Description
|--- |---
| patterns | An _array_ or a _comma separated string_ containing a list of regular expressions to match
| types  | An _array_ or a _comma separated string_ of the message events in which to look for the patterns
| middleware function | _optional_ function to redefine how patterns are matched. see [Botkit Middleware](middleware.md)
| callback | callback function that receives a message object

Example:
```javascript
controller.hears(['keyword','^pattern$'],['message_received'],function(bot,message) {

  // do something to respond to message
  bot.reply(message,'You used a keyword!');

});
```

When using the built in regular expression matching, the results of the expression will be stored in the `message.match` field and will match the expected output of normal Javascript `string.match(/pattern/i)`.

For example:

```javascript
controller.hears('open the (.*) doors',['message_received'],function(bot,message) {
  var doorType = message.match[1]; //match[1] is the (.*) group. match[0] is the entire group (open the (.*) doors).
  if (doorType === 'pod bay') {
    return bot.reply(message, 'I\'m sorry, Dave. I\'m afraid I can\'t do that.');
  }
  return bot.reply(message, 'Okay');
});
```


#### controller.on()
| Argument | Description
|--- |---
| event_name | a string or array containing an event or comma-separated list of events
| callback  | callback function in the form of form of function(bot, event) {...}

Handle events emitted by Botkit. The vast majority of events will call a callback function with 2 parameters - a bot instance, and the event object.  

[Read about receiving and handling events](core.md#receiving-messages-and-events)

```javascript
// handle a message event
controller.on('message_received', function(bot, message) {
  bot.reply(message,'Received');
});

// handle a channel join event
controller.on('channel_join', function(bot, event) {
  bot.reply(event,'Welcome to the channel!');
});
```

Note that you may also trigger your own events using [controller.trigger()](#controllertrigger) and handle them. This can be
useful for separating the trigger logic from the actual event handlers, as in the example below.

```javascript

// listen for a help request.. and then trigger a help_request event.
controller.hears('help', 'message_received', function(bot, message) {
  // this event can be triggered whenever a user needs help
  bot.trigger('help_request', [bot, message]);
});

controller.on('help_request', function(bot, message) {

  bot.reply(message,'I am here to help!');

});
```

#### controller.trigger()
| Argument | Description
|--- |---
| event_name | the name of a custom event
| parameters | an array of parameters to be passed on to any handler functions

Triggers a custom event, which can then be handled by [controller.on()](#controlleron).

The second argument to `controller.trigger()` is an array which should contain an array of parameters that will be passed on to any handler functions. While not required, it is highly recommended that custom events conform to the pattern used by native Botkit events and include a bot instance as the first parameter, and if applicable, a normalized Botkit event object as the second paramter:

```javascript
controller.trigger('my_custom_event', [bot, event]);
```

#### controller.changeEars()
| Argument | Description
|--- |---
| test_function | a function in the form function(tests_array, message_object)

This function can be used to modify the way Botkit's hearing system works by replacing the pattern matching function with a custom function.

The test function must perform synchronous tests, and should return true if the message represents a match, and otherwise return false.

Code based on Botkit's built-in regular expression test is below:

```
var hears_regexp = function(tests, message) {
    for (var t = 0; t < tests.length; t++) {
        if (message.text) {

            // the pattern might be a string to match (including regular expression syntax)
            // or it might be a prebuilt regular expression
            var test = null;
            if (typeof(tests[t]) == 'string') {
                try {
                    test = new RegExp(tests[t], 'i');
                } catch (err) {
                    botkit.log('Error in regular expression: ' + tests[t] + ': ' + err);
                    return false;
                }
                if (!test) {
                    return false;
                }
            } else {
                test = tests[t];
            }

            if (match = message.text.match(test)) {
                message.match = match;
                return true;
            }
        }
    }
    return false;
};

controller.changeEars(hears_regexp);
```

#### controller.excludeFromConversations()

| Argument | Description
|--- |---
| events | single event or an array of events to exclude from conversations

Messaging platforms send a dizzying array of event types - and new ones are added all the time!

Sometimes, it is desirable to exclude certain event types from inclusion in Botkit conversations. To do this, call `excludeFromConversations()` after creating your controller object.

For example:

```javascript
// always exclude facebook postback events from conversations
controller.excludeFromConversations('facebook_postback')
```

#### controller.spawn()
| Argument | Description
|--- |---
| configuration | an object with instance-specific configuration options
| callback | a function that will receive the new bot instance as a parameter

Spawn a new instance of the bot in order to send messages or handle replies.

Whenever the bot is doing something in response to an incoming message, Botkit will handle spawning bot instances for you.  However, in the event that your bot needs to send an alert or subscription message that is not directly in reply to an incoming message, spawning a bot is required.

The configuration options required to spawn a bot instance differ from platform to platform. In many cases, no additional configuration is required. Refer to the platform specific documnentation for more detail.

```javascript
function sendAlertToBot(alert_message) {

  controller.spawn({}, function(bot) {

    bot.say(alert_message);

  });

}
```

#### controller.defineBot()
| Argument | Description
|--- |---
| bot_constructor | function of the form function(botkit_controller, configuration) which returns a bot instance

This function is used to create new platform connectors for Botkit, and is used to define the specific behaviors and features of that platform's bot instance.

For more information, [read the guide to creating new platform connectors for Botkit](howto/build_connector.md)


#### controller.setTickDelay()
| Argument | Description
|--- |---
| delay | time between event loops

Adjust the speed of Botkit's event loops. By default, this is set to 1500ms, which means that Botkit will evaluate all active conversations and send any pending replies every 1.5 seconds.

Decreasing this time will increase the speed at which bots send replies in conversations.

For example:
```javascript
// only wait 100ms between conversation loops
controller.setTickDelay(100);
```

#### controller.startTicking()

This function begins Botkit's internal event loop, which will tick at an interval of once every 1.5 seconds unless modified using [controller.setTickDelay()](#controller-settickdelay)

In most cases, this is called internally by a platform connector and does not have to be called directly by bot code.

However, if your bot seems to freeze up after the first message in any conversation, you may need to call it!


#### controller.setupWebserver()
| Argument | Description
|--- |---
| port | a port number for the webserver
| callback | a function in the form function(err, webserver) {...}

Create an instance of the [Express.js webserver](https://expressjs.com) for use with platform-specific features like oauth and incoming webhooks.

Botkit provides a simple Express webserver for those who don't know or don't want to set up their own. However, if you want to do anything with your webserver other than those required by Botkit, it is our recommendation that you manage your own instance of Express. A good place to see this in action is [inside all of the botkit starter kits](starterkits.md).

#### controller.userAgent()

Returns a user agent string for use when making API requests to external resources.

#### controller.version()

Returns the current package version of Botkit's core library

#### controller.shutdown()

This function _stops_ the event loop from processing active conversations. It is the opposite of [controller.startTicking()](#controller-startticking)


# Botkit Bot Instance Objects

#### bot.reply()
| Argument | Description
|--- |---
| message | Incoming message object
| reply | _String_ or _Object_ Outgoing response
| callback | _Optional_ Callback in the form function(err,response) { ... }

Simple reply example:
```javascript
controller.hears(['keyword','^pattern$'],['message_received'],function(bot,message) {

  // do something to respond to message
  // ...

  bot.reply(message,"Tell me more!");

});
```


#### bot.say()
| Argument | Description
|--- |---
| message | A message object
| callback | _Optional_ Callback in the form function(err,response) { ... }

Slack-specific Example:
```javascript
bot.say(
  {
    text: 'my message text',
    channel: 'C0H338YH4' // a valid slack channel, group, mpim, or im ID
  }
);
```
Note: If your primary need is to spontaneously send messages rather than respond to incoming messages, you may want to use [Slack's incoming webhooks feature](readme-slack.md#incoming-webhooks) rather than the real time API.


Facebook-specific Example:
```javascript
bot.say(
    {
        text: 'my message_text',
        channel: '+1(###)###-####' // a valid facebook user id or phone number
    }
);
```

#### bot.startConversation()
| Argument | Description
|---  |---
| message   | incoming message to which the conversation is in response
| callback  | a callback function in the form of  function(err,conversation) { ... }

`startConversation()` is a function that creates conversation in response to an incoming message.
The conversation will occur _in the same channel_ in which the incoming message was received.
Only the user who sent the original incoming message will be able to respond to messages in the conversation.

#### bot.startPrivateConversation()
| Argument | Description
|---  |---
| message   | message object containing {user: userId} of the user you would like to start a conversation with
| callback  | a callback function in the form of  function(err,conversation) { ... }

`startPrivateConversation()` is a function that initiates a conversation with a specific user. Note function works only on platforms with multiple channels.

#### bot.createConversation()
| Argument | Description
|---  |---
| message   | incoming message to which the conversation is in response
| callback  | a callback function in the form of  function(err,conversation) { ... }

This works just like `startConversation()`, with one main difference - the conversation
object passed into the callback will be in a dormant state. No messages will be sent,
and the conversation will not collect responses until it is activated using [convo.activate()](#convoactivate).

Use `createConversation()` instead of `startConversation()` when you plan on creating more complex conversation structures using [threads](#conversation-threads) or [variables and templates](#using-variable-tokens-and-templates-in-conversation-threads) in your messages.

#### bot.createPrivateConversation()
| Argument | Description
|---  |---
| message   | incoming message to which the conversation is in response
| callback  | a callback function in the form of  function(err,conversation) { ... }

This works just like `startPrivateConversation()`, with one main difference - the conversation
object passed into the callback will be in a dormant state. No messages will be sent,
and the conversation will not collect responses until it is activated using [convo.activate()](#convoactivate).


# Botkit Conversation Objects


#### convo.activate()

This function will cause a dormant conversation created with [bot.createConversation()](#botcreateconversation) to be activated, which will cause it to start sending messages and receiving replies from end users.

A conversation can be kept dormant in order to preload it with [variables](#using-variable-tokens-and-templates-in-conversation-threads), particularly data that requires asynchronous actions to take place such as loading data from a database or remote source.  You may also keep a conversation inactive while you build threads, setting it in motion only when all of the user paths have been defined.

#### convo.addMessage
| Argument | Description
|---  |---
| message   | String or message object
| thread_name   | String defining the name of a thread

This function works identically to `convo.say()` except that it takes a second parameter which defines the thread to which the message will be added rather than being queued to send immediately, as is the case when using convo.say().

#### convo.addQuestion
| Argument | Description
|---  |---
| message   | String or message object containing the question
| callback _or_ array of callbacks   | callback function in the form function(response_message,conversation), or array of objects in the form ``{ pattern: regular_expression, callback: function(response_message,conversation) { ... } }``
| capture_options |  Object defining options for capturing the response. Pass an empty object if capture options are not needed
| thread_name   | String defining the name of a thread


When passed a callback function, conversation.ask will execute the callback function for any response.
This allows the bot to respond to open ended questions, collect the responses, and handle them in whatever
manner it needs to.

When passed an array, the bot will look first for a matching pattern, and execute only the callback whose
pattern is matched. This allows the bot to present multiple choice options, or to proceed
only when a valid response has been received. At least one of the patterns in the array must be marked as the default option,
which will be called should no other option match. Botkit comes pre-built with several useful patterns which can be used with this function. See [included utterances](#included-utterances)

Callback functions passed to `addQuestion()` receive two parameters - the first is a standard message object containing
the user's response to the question. The second is a reference to the conversation itself.

Note that in order to continue the conversation, `convo.next()` must be called by the callback function. This
function tells Botkit to continue processing the conversation. If it is not called, the conversation will hang
and never complete causing memory leaks and instability of your bot application!

The optional third parameter `capture_options` can be used to define different behaviors for collecting the user's response.
This object can contain the following fields:

| Field | Description
|--- |---
| key | _String_ If set, the response will be stored and can be referenced using this key
| multiple | _Boolean_ if true, support multi-line responses from the user (allow the user to respond several times and aggregate the response into a single multi-line value)

##### Using conversation.addQuestion with a callback:

```javascript
controller.hears(['question me'], 'message_received', function(bot,message) {

  // start a conversation to handle this response.
  bot.startConversation(message,function(err,convo) {

    convo.addQuestion('How are you?',function(response,convo) {

      convo.say('Cool, you said: ' + response.text);
      convo.next();

    },{},'default');

  })

});
```

##### Using conversation.addQuestion with an array of callbacks:

```javascript
controller.hears(['question me'], 'message_received', function(bot,message) {

  // start a conversation to handle this response.
  bot.startConversation(message,function(err,convo) {

    convo.addQuestion('Shall we proceed Say YES, NO or DONE to quit.',[
      {
        pattern: 'done',
        callback: function(response,convo) {
          convo.say('OK you are done!');
          convo.next();
        }
      },
      {
        pattern: bot.utterances.yes,
        callback: function(response,convo) {
          convo.say('Great! I will continue...');
          // do something else...
          convo.next();

        }
      },
      {
        pattern: bot.utterances.no,
        callback: function(response,convo) {
          convo.say('Perhaps later.');
          // do something else...
          convo.next();
        }
      },
      {
        default: true,
        callback: function(response,convo) {
          // just repeat the question
          convo.repeat();
          convo.next();
        }
      }
    ],{},'default');

  })

});
```

#### convo.say()
| Argument | Description
|---  |---
| message   | String or message object

convo.say() is a specialized version of `convo.addMessage()` that adds messages to the _current_ thread, essentially adding a message dynamically to the conversation. This should only be used in simple cases, or when building a conversation with lots of dynamic content. Otherwise, creating `threads` is the recommended approach.

Call convo.say() several times in a row to queue messages inside the conversation. Only one message will be sent at a time, in the order they are queued.

```javascript
controller.hears(['hello world'], 'message_received', function(bot,message) {

  // start a conversation to handle this response.
  bot.startConversation(message,function(err,convo) {

    convo.say('Hello!');
    convo.say('Have a nice day!');

  });
});
```

#### convo.ask()
| Argument | Description
|---  |---
| message   | String or message object containing the question
| callback _or_ array of callbacks   | callback function in the form function(response_message,conversation), or array of objects in the form ``{ pattern: regular_expression, callback: function(response_message,conversation) { ... } }``
| capture_options | _Optional_ Object defining options for capturing the response

convo.ask() is a specialized version of `convo.addQuestion()` that adds messages to the _current_ thread, essentially adding a message dynamically to the conversation. This should only be used in simple cases, or when building a conversation with lots of dynamic content. Otherwise, creating `threads` is the recommended approach.

In particular, we recommend that developers avoid calling `convo.ask()` or `convo.say()` inside a callbacks for `convo.ask()`. Multi-level callbacks encourage fragile code - for conversations requiring more than one branch, use threads!


#### convo.gotoThread()
| Argument | Description
|---  |---
| thread_name   | String defining the name of a thread

Cause the bot to immediately jump to the named thread.
All conversations start in a thread called `default`, but you may switch to another existing thread before the conversation has been activated, or in a question callback.

Threads are created by adding messages to them using `addMessage()` and `addQuestion()`

```javascript
// create the validation_error thread
convo.addMessage('This is a validation error.', 'validation_error');
convo.addMessage('I am sorry, your data is wrong!', 'validation_error');

// switch to the validation thread immediately
convo.gotoThread('validation_error');
```


#### convo.transitionTo
| Argument | Description
|---  |---
| thread_name   | String defining the name of a thread
| message   | String or message object

Like `gotoThread()`, jumps to the named thread. However, before doing so,
Botkit will first send `message` to the user as a transition. This allows
developers to specify dynamic transition messages to improve the flow of the
conversation.

```javascript
// create an end state thread
convo.addMessage('This is the end!', 'the_end');

// now transition there with a nice message
convo.transitionTo('the_end','Well I think I am all done.');
```

#### convo.beforeThread
| Argument | Description
|--- |---
| thread_name | String defining the name of a thread
| handler_function | handler in the form function(convo, next) {...}

Allows developers to specify one or more functions that will be called before the thread
referenced in `thread_name` is activated.

`handler_function` will receive the conversation object and a `next()` function. Developers
must call the `next()` function when their asynchronous operations are completed, or the conversation
may not continue as expected.  

Note that if `gotoThread()` is called inside the handler function,
it is recommended that `next()` be passed with an error parameter to stop processing of any additional thread handler functions that may be defined - that is, call `next('stop');`

```javascript
// create a thread that asks the user for their name.
// after collecting name, call gotoThread('completed') to display completion message
convo.addMessage({text: 'Hello let me ask you a question, then i will do something useful'},'default');
convo.addQuestion({text: 'What is your name?'},function(res, convo) {
  // name has been collected...
  convo.gotoThread('completed');
},{key: 'name'},'default');

// create completed thread
convo.addMessage({text: 'I saved your name in the database, {{vars.name}}'},'completed');

// create an error  thread
convo.addMessage({text: 'Oh no I had an error! {{vars.error}}'},'error');


// now, define a function that will be called AFTER the `default` thread ends and BEFORE the `completed` thread begins
convo.beforeThread('completed', function(convo, next) {

  var name = convo.extractResponse('name');

  // do something complex here
  myFakeFunction(name).then(function(results) {

    convo.setVar('results',results);

    // call next to continue to the secondary thread...
    next();

  }).catch(function(err) {
    convo.setVar('error', err);
    convo.gotoThread('error');
    next(err); // pass an error because we changed threads again during this transition
  });

});
```


#### convo.setVar
| Argument | Description
|---  |---
| variable_name   | The name of a variable to be made available to message text templates.
| value | The value of the variable, which can be any type of normal Javascript variable

Create or update a variable that is available as a Mustache template token to all the messages in all the threads contained in the conversation.

The variable will be available in the template as `{{vars.variable_name}}`

#### convo.onTimeout()
| Argument | Description
|--- |---
| callback | _Optional_ Callback in the form function(convo) { ... }

Provide a handler function that will be called in the event that a conversation reaches its timeout threshold without any user response.


#### convo.extractResponses()

Returns an object containing all of the responses a user sent during the course of a conversation.

```javascript
var values = convo.extractResponses();
var value = values.key;
```

#### convo.extractResponse()

Return one specific user response, identified by its key.

```javascript
var value  = convo.extractResponse('key');
```


#### convo.sayFirst()
Works just like convo.say, but injects a message into the first spot in the queue
so that it is sent immediately, before any other queued messages.

#### convo.stop()
end the conversation immediately, and set convo.status to `stopped`

#### convo.repeat()
repeat the last question sent and continue to wait for a response.

#### convo.silentRepeat()
simply wait for another response without saying anything.

#### convo.next()
proceed to the next message in the conversation.  *This must be called* at the end of each handler.

#### convo.setTimeout()
| Argument | Description
|--- |---
| time_in_ms | Time in milliseconds

times out conversation if no response from user after specified time period

#### convo.hasThread()

#### convo.successful()
