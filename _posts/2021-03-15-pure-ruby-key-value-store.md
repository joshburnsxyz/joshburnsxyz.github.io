---
layout: post
title: A Pure Ruby Key/Value Store
subtitle: Remaking redis
date: "2021-03-15"
---

For the last little while, I've been building a Key/Value store, why? mostly for shits and giggles, but also to play with
rubys `socket` library. Minikv is really just a pure ruby implementation of a redis-like in-memory key/value store system. It acts
as its own api client which is a really neat feature.

## How it works

At its core, Minikv is a ruby hash object sitting on a TCP socket, that can be interacted with via writing strings to that socket,
those in-turn are parsed via an internal method and then stored as a specific data type.

```ruby
require "socket"

# Assign a blank hash to a variable
@data = {}

# Define a parser that can determine what type of data is being stored.
# To accomplish this we look at the first bit, which is then checked against
# a case statement. Strings are prefix by a +, integers are prefixed with :, and so on.
# Minikv also implements arrays and bulk strings, however these are processed by some more
# complicated recursion logic that probably deserves its own blog post.
#
# commands look like this: command=key|value, so we need to seperate those parts out.
# we should also strip the first bit (+,:, etc.).
def parser(value)
  command = value.partition("=").first
  key = value.partition("|").first
  value = value.partition("|").last
  case value[0] # value[0] represents the first character in the string
  when "+"
    val = value.partition("+").last # strip the first character out
    @data[key.to_sym] = val.to_s # save value as a string in the hash
  when ":"
    val = value.partition(":").last
    @data[key.to_sym] = val.to_i
  when "-"
    val = value.partition("-").last
    @data[key] = Error.new(val.to_s)
  else
    puts Error.new("Data type not implemented") # generic catch all error
  end
end

# Create a TCP server
@server = TCPServer.new("localhost", 1337)

loop do
  Thread.start(@server.accept) do |client|
    while (line = client.gets)
      parser(line) # parse client input
    end
    client.close
  end
end

```

This is simply just an overview of how the system operates, it should be treated as pseudocode, simply illustrating the idea. 
The full source for minikv is available [here](https://github.com/joshburnsxyz/minikv).

