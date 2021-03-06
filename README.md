# [Wordhop](https://www.wordhop.io) - SDK For Human + AI Conversational Experiences
## For Bots Built in Python

Bots enable businesses to respond to customers immediately but they often fail to understand user intent.  According to Facebook, bots fail 70% of the time. Wordhop helps solve this problem with a toolkit to easily keep humans in the loop when AI fails your customers.    The solution includes an SDK for bot developers to connect their bots to Slack, and a Slack app to get alerts, then pause and take over a bot.

![Solution](https://cloud.githubusercontent.com/assets/7429980/22609969/491afe58-ea31-11e6-8928-27e1a1f1d6bd.png)



You can integrate Wordhop in minutes and it begins working immediately.
This module has been tested with Messenger, Slack, Skype, and Microsoft Webchat. Please see our [examples](./examples/).
It supports bot developers working in Node, Python and Ruby.

### What you can do with Wordhop:
You can view a full list of features at (https://www.wordhop.io).  It's core purpose can be explained with this single GIF  

![Takeover](https://cloud.githubusercontent.com/assets/7429980/22609935/22e39740-ea31-11e6-8286-e5a3ae545565.gif)

### What you need to get started:
* [A Slack Account](http://www.slack.com)
* [Wordhop for Slack](https://slack.com/oauth/authorize?scope=users:read,users:read.email,commands,chat:write:bot,chat:write:user,channels:read,channels:history,files:write:user,channels:write,bot&client_id=23850726983.39760486257)
* [A Chatbot built in Python](./examples/)

##### Operational Dependencies:
1.  You'll need an API key from Wordhop and for each Chatbot a Bot Token.  You can get both of those (free) when you add Wordhop to Slack and through a conversation with Wordhop. 
2.  If you're building a Messenger Chatbot, you'll need to setup a Facebook App, Facebook Page, get the Page Access Token from Facebook and link the Facebook App to the Facebook Page for Wordhop to work.


### Installation

```bash
$ pip install wordhop
```


### Usage

```python
from wordhop import Wordhop
apiKey = xxxxxxxxxxxxxxxxxxxxxx # <= key provided by Wordhop for Slack
clientKey = xxxxxxxxxxxxxxxxxxxxxx # <= key provided by Wordhop for Slack
botPlatform = 'messenger'; # <= possible values: 'messenger', 'slack'
token = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx # <= only required for Messenger bots.

wordhop = Wordhop(apiKey, clientKey, botPlatform, token)
```
##### Incoming Message Schema:
Throughout this documentation, you will see references to `incomingMessage`. Depending on whether you have a Messenger or Slack bot, the schema will be different. The value of `incomingMessage` should be equal to the message you receive directly from either the Messenger webhook response, or from the Slack RTM event response.

```python
# Example of a Slack Incoming Message
{
    "type": "message",
    "channel": "D024BE91L",
    "user": "U2147483697",
    "text": "Hello world",
    "ts": "1355517523.000005"
}

# Example of a Messenger Incoming Message
{
  "sender":{
    "id":"USER_ID"
  },
  "recipient":{
    "id":"PAGE_ID"
  },
  "timestamp":1458692752478,
  "message":{
    "mid":"mid.1457764197618:41d102a3e1ae206a38",
    "seq":73,
    "text":"hello, world!",
    "quick_reply": {
      "payload": "DEVELOPER_DEFINED_PAYLOAD"
    }
  }
}  
```

##### Outgoing Message Schema:
Throughout this documentation, you will see references to `outgoingMessage`. Depending on whether you have a Messenger or Slack bot, the schema, as defined by each platform, will be different. Every time you track an outgoing message, the schema requirements match the respective platform.

```python
# Example of Slack Outgoing Message
{
    "channel": "C024BE91L",
    "text": "Hello world"
}

# Exmaple of Messenger Outgoing Message
{
  "recipient":{
    "id":"USER_ID"
  },
  "message":{
    "text":"hello, world!"
  }
}
```

##### Tracking received messages:

When your bot receives an incoming message, you'll need to log the data with Wordhop by calling to `wordhop.hopIn`. 
__Note__: Wordhop can pause your bot so that it doesn't auto response while a human has taken over. The server response from your `hopIn` request will pass the `paused` state. Use that to stop your bot from responding to an incoming message. Here is an example:

```python
hopInResponse = wordhop.hopIn(incomingMessage)
# If your bot is paused, stop it from replying
if hopInResponse.get('paused'):
    return "Success"
    ...
```

##### Tracking sent messages:

Each time your bot sends a message, make sure to log that with Wordhop by calling to `wordhop.hopOut`. Here is an example of a function that we're calling `sendIt` that tracks an outgoing message and at the same time, has the bot say the message:
```python
def sendIt(channel, text):
    # schema matches Messenger
    outgoingMessage = {'recipient': {'id': channel},'message': {'text': text}}
    wordhop.hopOut(outgoingMessage)
    bot.send_text_message(channel, text) # <= example of bot sending reply
    ...
```

##### Log Unknown Intents:

Find the spot in your code your bot processes incoming messages it does not understand. Within that block of code, call to `wordhop.logUnkownIntent` to capture these conversational ‘dead-ends’. Here's an example:

```python
# let the user know that the bot does not understand
sendIt(recipient_id, 'Huh?')
# capture conversational dead-ends.
wordhop.logUnknownIntent(incomingMessage) 
```
##### Dial 0 to Speak With a Live Human Being:

Wordhop can trigger alerts to suggest when a human should take over for your Chatbot. To enable this, create an intent such as when a customer explicitly requests live assistance, and then include the following lines of code where your bot listens for this intent:

```python
# match an intent to talk to a real human
if text == 'help':
    # let the user know that they are being routed to a human
    sendIt(recipient_id, 'Hang tight. Let me see what I can do.')
    # send a Wordhop alert to your slack channel
    # that the user could use assistance
    wordhop.assistanceRequested(incomingMessage);
```

##### Human Take Over:

To enable the ability to have a human take over your bot, add the following code:

```python
# Handle forwarding the messages sent by a human through your bot
def onChatResponse(self, args):
    channel = args["channel"]
    text = args["text"]
    bot.send_text_message(channel, text) # <= example of bot sending message
    
wordhop.on('chat_response', onChatResponse)
```

Go back to Slack and wait for alerts. That's it! 
[Be sure to check out our examples.](./examples/)


### Looking for something we don't yet support?  
* [Join our mailing list and we'll notifiy you](https://www.wordhop.io/contact.html)
* [Contact Support](mailto:support@wordhop.io)
