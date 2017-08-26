# GRUMP 1.0
Guillaume's Really Useful MUD Protocol

Intended to be found within normal TELNET output from a game, with
special
delimiters.  Handshaking can be possible at login using TELNET protocol
specifications; however, in-game toggling is also possible via the
game's
own settings.

Updates in the protocol are called frames, which allow you to do two
actions:
* Key - A type of update whose overwrites are intended to overwrite
* Mask - A type of update that is partial/merge, and includes additional
  notations

## Sending a GRUMP Frame

Frames can consist of both a key- and a mask-type update, but it is
expecting
that it will only receive one of each or both.  If multiple updates are 


```
<grump>{"overwrite": "this overwrote an entire base key", "abc": [123,
456]}</grump>
<grump-mask>{"+abc":["added a string to the array"]}</grump-mask>
```

### Keyframes

These are frames whose trees denote they are used for recreating base
nodes.
All nodes from the base downward will be removed.  Only siblings at the
root
level will be kept intact.

Examples below have extra whitespace for readability.

Example:

Frame 1
```js
<grump>
{
    "group": {
        "members": {
            "123": {"name": "Gui", "health": 100, "effects":["poison"]},
            "555": {"name": "Steve", "health": 200, "effects":[]}
        },
        "owner_name": "Gui"
    },
    "inventory": []
}
</grump>
```

Frame 2:
```js
<grump>
{"group": {
    "members": {
        "123": {"name": "Gui", "health": 50, "effects":[]}
    }
}}
</grump>
```

Expectation: The `group.members` list will only have one person in it,
but the
"owner_name" node will also be removed, but the "inventory" base node
would
remain intact because it's a sibling root node.

```js
{
    "group": {
        "members": {
            "123": {"name": "Gui", "health": 50, "effects":[]}
        }
    },
    "inventory": []
}
```


### Mask Frames
Not yet defined, because it might be too complicated to be worth it.

Basic concept:
Define a prefixing symbol for fields to denote an overwrite, merge, or
"set" notation.

Primarily aimed for lists, this will denote how to handle a merge if a
field already exists.

Any fields not prefixed with a symbol will be non-destructive, and keep
sibling fields.



Prefixes:
* `+` Addition - If a key exists, add the two values together
* `-` Removal - If a key exists with a value, remov ethe vlu
* `$` Set - If a value exists in both, keep it, but don't add a new one

Example
Frame 1
```js
<grump>
{"group": {
    "members": {
        "123": {"name": "Gui", "health": 100, "effects":["poison",
"weaken"]},
        "555": {"name": "Steve", "health": 200, "effects":[]},
        "980": {"name": "Bilbo", "health": 10, "effects":["bleeding"]}
    },
}}
</grump>
```

Frame 2
```js
<grump-mask>
{
    "group.members.123.health": 5,
    "+group.members.123.effects": ["weaken"],  # add items, can have
duplicates
    "$group.members.123.effects": ["bleeding", "blinded"],  # add items,
as a set
    "-group.members.555.effects": ["poison"],  # delete items from a
list
    "-group.members.980": true,  # delete a node entirely
    "=group.members.555": "group.members.777",  # copy
    ">group.members.777": "group.members.888",  # move
    "?group.members.123.name": "Gui"  # test
}
</grump-mask>
```

* Gui - Health to 5, added `bleeding` and `blinded` and removed
  `poison`, also a second `weaken`.
* Delete Bilbo
* Copy Steve, we now have two Steves, great
* Move Steve from node 777 to 888
* Only apply this patch as a whole, if the name of 123 member is "Gui"


## Defined Protocol Standards

Any protocol standards defined here are prefixed with "g" to denote they
use
the standard.  The standard definitions are not exhaustive, and
additional
fields can always be added to further enhance your game.

### gResources

Health, mana, energy, movement, currency, etc.

```js
{
    "gResources": {
        "health": {  # custom identifier of resource
            "name": "Health",  # String (optional) assume undefined if
not provided
            "value": 10,  # Number the current value
            "minValue": 0,  # Number (optional) assume 0 if not provided
            "maxValue": 1000,  # Number (optional) assume undefined if
not provided
        }

    }
}
```

## Potential Expansions
* Keyframe identification number, an incrementing number that does not
  need to
    increment by a set value, but is used for ordering any frames that
might be
    received at the same time by the client-- but need ordering due to
an
    asynchronous task being the cause for the order to change?

* Compression, denoted by <grump-gzip></grump-gzip> to put the data in a 


* Notes on TELNET implementation
IAC SB GRUMP MASK {"room":{"exits": ... }} IAC SE
IAC SB GRUMP SET {"room": {"name": "The Hall of Heroes", ... }} IAC SE

* Masking similar to http://jsonpatch.com/

