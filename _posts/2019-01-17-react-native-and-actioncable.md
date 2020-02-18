---
title: "React Native and ActionCable"
tags: ["Action Cable", "React Native", "Ruby on Rails", "WebSocket"]
---

I needed to get my React Native app to connect with my Rails app through ActionCable. There were a couple React Native packages out there that facilitated this. Although they seemed like sound solutions, I was a little hesitant to use them since they didn’t seem like they were regularly maintained. Also, I felt it was valuable to know how everything worked so that I can fix it if something ever broke. So I coded my solution plain old JavaScript, without the need of some other external package.

First step, we need to create a connection to the server.

```javascript
// This should be wss in production ;)
ws = new WebSocket('ws://localhost:3000/cable', [
  'actioncable-v1-json',
  'actioncable-unsupported'
]);
```

I noticed that ActionCable would make the request with a couple protocols set. I’m not sure what difference it makes, but I figured I’d add them just for good measure.

Second step, we need to subscribe to a specific channel.

```javascript
ws.onopen = () => {
  ws.send(
    JSON.stringify({
      command: "subscribe",
      identifier: JSON.stringify({ channel: "ChatChannel" })
    })
  )
}
```

The nested `JSON.stringify` is necessary in order for this to work properly. Don’t forget to include it.

Third step, we need to receive data and update the state.

```javascript
const isChat = (data) => {
  return (
    data.identifier &&
    JSON.parse(data.identifier).channel == "ChatChannel" &&
    data.message
  )
}

ws.onmessage = (e) => {
  const data = JSON.parse(e.data);
  if(isChat(data)) {
    console.log(data)
    // update the state here...
  }
}
```

I broke out the logic into a separate function to clean things up a bit. There are a lot of ping messages that are received, and we only want to respond to the messages that we subscribed.

Check out the [full React Native example](https://gist.github.com/iarambula/d90429418b62a7a9e5891713eb3451b4).

