---
layout: post
title: Jport and JSON messaging basics
subtitle: Wtf is JSON messaging and why should I care?
date: "2021-01-05"
---

JSON messaging is the idea of transporting small packets of data in a JSON object, 
things like GPS location updates, server error reports, seurity alarm events, anything that 
requires, constant small packets of data to be communicated often. 

I've been playing with this idea for some time with various different contexts. 
The outcome was Jport, A generic JSON reciever that takes in data over a TCP connection and then ships it to a 
"Transporter" process. Currently I have implemented 4 different types of transporters.

- Textfile - Writes the message and timestamp to a logfile.
- TCP - Writes the message to a TCP connection.
- UDP - Writes the message to a UDP connection.
- Redis - Writes the message to a Redis database.

## JSON Messaging "Spec"

The specification i've developed for what I now refer too as JSON messaging is as follows.

### All *messages* should follow this format.

```js
{
    "content": String,
    "target": String,
    "type": String
}
```

- The `content` field is simply the text content of the message.
- The `target` field defines which transport the server (Jport) will use to process and store the message. 
(Jport currently supports `textfile`, `tcp`, `udp`, and `redis` as valid options here).
- The `type` field is another arbitary field that defines the "type" of message, this is intended to be used for filtering purposes.

### How Messages are recieved
The server (Jport) sits on a TCP port (i.e 8080) and exists idle. A TCP connection is made from the client that
is writing the message. The raw JSON string is then written via the TCP connection, and the TCP connection is 
closed. The server (Jport) handles the rest.

