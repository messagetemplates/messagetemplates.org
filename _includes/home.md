# Message Templates

A language-neutral specification for 1) _capturing_, and 2) _rendering_, structured log events in a format that’s both human-friendly and machine-readable.

![Message Templates](https://messagetemplates.org/img/MessageTemplates.png){: .hidpi }

This document explains the reasons message templates are used, and provides a specification of their syntax, capturing behavior, and rendering behavior, to assist in creating implementations for various programming languages and logging interfaces.

## A brief history of structured logging APIs

Early application logging, or **`printf` debugging**, produced a stream of text describing the events taking place within an application.

```c
log("User %s logged in from %s", username, ipAddress);
  // -> 2016-05-27T13:02:11.888 User alice logged in from 123.45.67.89
```

This worked well for simple applications running on a single computer, and over the years many tools evolved to help support this format in ever-more-complex, and ever-more-distributed applications. Log parsing can extract most of the relevant information from the event above, but considerable effort is required for non-trivial diagnostic scenarios.

At various times, alternatives to text logs have been proposed under the banner of [structured logging](http://gregoryszorc.com/blog/2012/12/06/thoughts-on-logging---part-1---structured-logging/), which considers log data to be a stream of fully-structured events with key/value properties. Structured log events are much easier to work with in large, complex and distributed applications, because relevant events can be identified using queries over their properties, rather than by regular expressions over a text payload.

The obvious method of recording structured log events, found particularly in languages with terse key/value property syntax, is to **encode the event directly** as a map of key/value pairs:

```javascript
log({eventId: "user_logged_in", username: username, ip_address: ipAddress });
   // -> {"time": "2016-05-27T13:02:11.888", "eventId": "user_logged_in",
   //     "username": "alice", "ip_address": "123.45.67.89"}
```

JSON is used as an example rendering here, but the concept is not tied to a particular representation.

This greatly improves machine-readability, enabling queries like `username = 'alice'` on the raw log stream, but the event itself is not _human-friendly_. Log events recorded in prose make full use of our language processing and pattern recognition abilities to ease cognition. A long stream of key/value pairs is painful to visually process. Compare the representative output of the two examples above to experience this effect - what's happening in the second event?

Another approach employs **convention** to identify properties within a `printf`-style message:

```c
log("User username=%s logged in from ip_address=%s", username, ipAddress);
  // -> 2016-05-27T13:02:11.888 User username=alice logged in from ip_address=123.45.67.89
```

In principle, this technique benefits from greater readability, but in practice the looseness of the format causes ambiguities when parsing, and inconsistent usage within development teams. Ambiguities arise when a property inserted into the event contains embedded spaces and/or `=` characters. The lack of a defined grammar (and lack of awareness of the grammar in the logging tool) means developers have no opportunity to validate their use of the syntax – breaking the convention is easy and goes unnoticed.

Finally, niche logging systems like [Event Tracing for Windows](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363668.aspx) have taken a middle ground, combining event ids in the log event with a manifest of format specifiers describing how a particular event can be rendered as text. Despite the near-perfect balance of performance, human-friendliness and machine-readability, the effort associated with assigning event ids and managing manifests have an impact on development-time ergonomics that limit the appeal of this system.

All of these issues and shortcomings have kept fully-structured logging from reaching widespread adoption, but the need for it has continued to grow.

## Message templates overview

Message templates take the best of all these approaches to provide optimal human-friendliness, perfect machine-readability and excellent development-time ergonomics.

A message template is a format specifier with _named holes_ for event data:

```
User {username} logged in from {ip_address}
```

Message templates are unique in that they provide a means of _capturing_ the event, as well as a means of _rendering_ it into human-friendly text.

Logging APIs capture log events by positionally matching arguments with the named holes:

```c
log("User {username} logged in from {ip_address}", username, ipAddress)
  // -> {
  //      "time": "2016-05-27T13:02:11.888",
  //      "template": ("User {username} logged in from {ip_address}", 
  //      "username": "alice", 
  //      "ip_address": "123.45.67.89"
  //    }
```

An event captured using a message template is not immediately rendered into text unless it is being displayed directly to a user. Instead, the template itself is captured, along with property values for each argument. This is stored in an intermediate representation like JSON for processing.

Because the template remains constant regardless of the actual property values substituted, it is possible to treat the template as an _event type_ that identifies all events from the same template. In this example, log-ins from various users will carry the different `username` values, but the consistent message template shared by all of them means they can be retrieved as a group.

When the event is displayed to a human user, or searched for text, it is _rendered_ by replacing each of the named holes with the corresponding property:

```c
// -> User alice logged in from 123.45.67.89 
```

## Syntax

A message template is a block of text with embedded _holes_ that name a property to be captured and inserted into the text.

* Property names are written between `{` and `}` brackets
* Brackets can be escaped by doubling them, e.g. {% raw %}`{{`{% endraw %} will be rendered as `{`
* Property names may be prefixed with an optional operator, `@` or `$`, to control how a property is captured
* Property names may be suffixed with an optional format, e.g. `:000`, to control how the property is rendered; the formatting semantics are application-dependent, and thus require the formatted value to be captured alongside the raw property if rendering is to take place in a different environment

### Template

![Template](https://messagetemplates.org/img/railroad/Template.png){: .hidpi }

```
Template ::= ( Text | Property )*
```

### Text

![Text](https://messagetemplates.org/img/railroad/Text.png){: .hidpi }

```
{% raw %}Text     ::= ( [^\{] | '{{' | '}}' )+{% endraw %}
```

### Property

![Property](https://messagetemplates.org/img/railroad/Property.png){: .hidpi }

```
Property ::= '{' ( '@' | '$' )? ( PropertyName | Index ) ( ',' Alignment )? ( ':' Format )? '}'
```

### PropertyName

![PropertyName](https://messagetemplates.org/img/railroad/PropertyName.png){: .hidpi }

```
PropertyName
         ::= [0-9A-z_]+
```

### Index

![Index](https://messagetemplates.org/img/railroad/Index.png){: .hidpi }

```
Index    ::= [0-9]+
```

### Format

![Format](https://messagetemplates.org/img/railroad/Format.png){: .hidpi }

```
Format   ::= [^\{]+
```

### Alignment

![Alignment](https://messagetemplates.org/img/railroad/Alignment.png){: .hidpi }

```
Alignment
         ::= '-'? [0-9]+
```

## Capturing

A message template may be used to _capture_ properties when provided a list of argument values. Each property in the message template is associated with exactly one value in the argument list.

* Templates that use numeric property names like `{0}` and `{1}` exclusively imply that arguments to the template are captured by numeric index
* If any of the property names are non-numeric, then all arguments are captured by matching left-to-right with holes in the order in which they appear

## Rendering

## Examples

## Implementations

The following logging libraries have support for message templates.

 * [Emit](https://github.com/emit-rs/emit) (Rust*)
 * [FsMessageTemplates](https://github.com/messagetemplates/messagetemplates-fsharp) (F#)
 * [LibLog](https://github.com/damianh/LibLog) (C#)
 * [Logary](https://github.com/logary/logary) (F#)
 * [MessageTemplates](https://github.com/messagetemplates/messagetemplates-csharp) (C#)
 * [Microsoft.Extensions.Logging](https://github.com/aspnet/Logging) (C#)
 * [Semlogr](https://github.com/semlogr/semlogr) (Ruby)
 * [Seqlog](https://seqlog.readthedocs.io/en/latest/) (Python)
 * [Serilog](https://serilog.net) (C#)
 * [serilogj](https://github.com/80dB/serilogj) (Java)
 * [structured-log](https://github.com/structured-log/structured-log) (JavaScript)

  _* Converts a custom capturing syntax to message templates for rendering and storage_

Library missing from this list? [Let us know](https://github.com/messagetemplates/messagetemplates.org/issues/new) so that we can add it.
