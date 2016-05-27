---
title: Message Templates
---

# Message Templates

A language-neutral specification for 1) capturing, and 2) rendering, structured log events in a format that’s both human-friendly and machine-readable.

[Anatomy of a message template, capturing, rendering]

A brief history of structured logging APIs

Early application logging, or `printf` debugging, produced a stream of text describing the events taking place within an application.

log("User %s logged in from %s", username, ipAddress);
  // -> 2016-05-27T13:02:11.888 User alice logged in from 123.45.67.89

This worked well for simple applications running on a single computer, and over the years many tools evolved to help support this format in ever-more-complex, and ever-more-distributed applications. Log parsing can extract most of the relevant information from the event above, but considerable effort is required for non-trivial diagnostic scenarios.

At various times, alternatives to textual logs have been proposed under the banner of “structured logging”, which considers log data a stream of fully-structured events with key/value properties. Structured log events are much easier to work with in large, complex and distributed applications, because relevant events can be identified using queries over their properties, rather than by regular expressions over a text payload.

One method of recording structured log events, found particularly in languages with terse key/value property syntax, records the event directly as a map of key/value pairs:

log({eventId: "user_logged_in", username: username, ip_address: ipAddress });
   // -> {"time": "2016-05-27T13:02:11.888", "eventId": "user_logged_in", "username": "alice", "ip_address": "123.45.67.89"}

(JSON is used as an example rendering here, but the concept is not tied to a particular representation.)

This does greatly improve machine-readability, enabling queries like `username = 'alice'` directly on the log stream, bu the event, though human-readable, is not _human-friendly_. Log events recorded in a human language, `printf`-style log, make full use of our language processing and parttern recognition facilities to ease cognition. A long stream of key/value pairs is painful to visually process.

The common practice of attaching contextual data such as application, thread, and process ids to structured events exacerbates this issue by obscuring the event-specific properties alongside the general contextual ones. 

Another approach employs convention to identify properties within a `printf`-style message:

log("User username=%s logged in from ip_address=%s", username, ipAddress);
  // -> 2016-05-27T13:02:11.888 User username=alice logged in from ip_address=123.45.67.89

In principle, this technique benefits from greater readability, but in practice the looseness of the format can cause ambiguities when parsing, and inconsistent usage within development teams. Ambiguities arise when a property inserted into the event contains embedded spaces and/or `=` characters. The lack of a defined grammar (and lack of awareness of the grammar in the logging tool) means developers have no opportunity to validate their use of the syntax – breaking the convention is easy and goes unnoticed.

Finally, niche logging systems like Event Tracing for Windows have taken a middle ground, combining event ids in the log event with a manifest of format specifiers describing how a particular event can be rendered as text. Despite the near-perfect balance of performance and machine-readability, the effort associated with assigning event ids and managing manifests have an impact on development-time ergonomics that limit the appeal of this system.

All of these issues and shortcomings have kept fully-structured logging from reaching widespread adoption, but the need for it has continued to grow.

Message templates take the best of all these approaches to provide optimal human-friendliness, perfect machine-readability and excellent development-time ergonomics.

Overview

A message template is a format specifier with _named holes_ for event data:

```
"User {username} logged in from {ip_address}"
```

Message templates are unique in that they provide a means of _capturing_ the event, as well as a means of _rendering_ it into human-friendly text.

Logging APIs capture log events by positionally matching arguments with the named holes:

log("User {username} logged in from {ip_address}", username, ipAddress)
  // -> {
             "time": "2016-05-27T13:02:11.888",
"template": ("User {username} logged in from {ip_address}", 
"username": "alice", 
"ip_address": "123.45.67.89"
           }

An event captured using a message template is not immediately rendered into text. Instead, the template itself is captured, along with property values for each argument.

Because the template remains constant regardless of the actual property values substituted, it is possible to treat the template as an _event type_ that identifies all events from the same template.

When the event is displayed to a human user, or searched for text, it is _rendered_ by replacing each of the named holes with the corresponding property:

// -> User alice logged in from 123.45.67.89 


Syntax (v1.0)

Capturing (v1.0)

Rendering (v1.0)

Implementations



