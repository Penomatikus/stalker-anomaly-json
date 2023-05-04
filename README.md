# stalker-anomaly-json

Strong and resilient script, to produce valid json strings or logfiles in S.T.A.L.K.E.R. Anomaly. 

Provides two global functions to create valid JSON from the following Lua types: `boolean`,`number`, `string`, `table`, `userdata` and `nil`.
Where as userdata only allows engine class `CTime` which will be converted to RFC3339+02:00 (because the monolith in in urkrain). Otherwise it justs adds the value: "unsupported userdata type". 

The implementation is using the dispatcher pattern to decouple concerns by defining a handler for each type listed above. It maps these types to corresponding handlers, which are called to process the input.

_- yes you can just dump your m_data in it_
## USAGE
### to_json(value, keyname)  
This will procude a string representation of the JSON, which might come in handy for logging.  

```lua
--[[ Result string with actual linebreaks and intent:
{
    "faction": "actor_zombied"
}
--]]
local faction_json = utils_json.to_json(db.actor:character_community(), "faction")
```

### to_logfile(value, filename)
This will write a new .json file named by `filename` to `./appdata/logs`.

```lua
--[[ Content of ./appdata/logs/actor_miscellaneous.json:
{
    "actor_miscellaneous": {
        "actual_rept": 30,
        "actual_rank": 148
    }
}
--]]
utils_json.to_logfile(game_statistics.actor_miscellaneous, "actor_miscellaneous")
```

## Resilience
- *Safe Dispatching*: Prior to invoking any type handler, the dispatcher first searches for a registered handler corresponding to the type of the given value. If no such handler is found, the JSON key's value will invariably read 'no json handler implemented for: _provided_type_'.
- *Encapsulation*: To prevent misuse, strong encapsulation is employed in the implementation by defining both the dispatcher itself and all associated handlers as local. 
- *Duck typing*: For userdata, the corresponding handler attempts to call a ctime method on the provided value, applying RFC3339 as the JSON key's value if available and invariably read 'unsupported userdata type' if not. More supported userdata will be added in the future.
- *Divide and conquer*: When handling tables, the corresponding handler uses implicit recursive calls to itself via the dispatcher, employing a 'divide and conquer' approach.
