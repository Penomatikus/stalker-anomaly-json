# stalker-anomaly-json

Strong and resilient script, to produce valid json strings or logfiles in S.T.A.L.K.E.R. Anomaly. 

Provides two global functions to create valid JSON from the following Lua types: `boolean`,`number`, `string`, `table`, `userdata` and `nil`.
Where as userdata only allows engine class `CTime` which will be converted to RFC3339+02:00 (because the monolith is in urkrain). Otherwise it justs adds the value: "unsupported userdata type". 

The implementation is using the dispatcher pattern to decouple concerns by defining a handler for each type listed above. It maps these types to corresponding handlers, which are called to process the input.

_- yes you can just dump your m_data in it_  

### USAGE
#### to_json(value, keyname)  
This will procude a string representation of the JSON, which might come in handy for logging.  

```lua
--[[ Result string with actual linebreaks and intent:
{
    "faction": "actor_zombied"
}
--]]
local faction_json = utils_json.to_json(db.actor:character_community(), "faction")
```

#### to_logfile(value, filename)
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

#### to_logfile(value, filename) function showcase
It is also possible to retreive rich function information as json. 
```lua
--[[ Content of ./appdata/logs/function_showcase.json:
{
    "function_showcase": {
	    "func_ref_in_table": {
			"random_coc_func": {
				"type": "Lua-Function",
				"name": "r_CTime",
				"args": "p, caller",
				"local": false,
				"source": ".\gamedata\scripts\utils_data.script",
				"line": 277
			}
		},
		"func_reference": {
			"type": "Lua-Function",
			"name": "new_func_for_reference",
			"args": null,
			"local": true,
			"source": ".\gamedata\scripts\bind_stalker.script",
			"line": 441
		},
		"func_anonymous": {
			"type": "Lua-Function",
			"name": "anonymous",
			"args": "a, b",
			"local": false,
			"source": ".\gamedata\scripts\bind_stalker.script",
			"line": 444
		}
	}
}
--]]
local function new_func_for_reference() end
local table = {
	["func_reference"] = new_func_for_reference,
	["func_anonymous"] = function(a, b) print("hi mom! <3") end,
	["func_ref_in_table"] = { ["random_coc_func"] = utils_data.r_CTime }
}
utils_json.to_logfile(table, "function_showcase")
```

## Resilience
- *Safe Dispatching*: Prior to invoking any type handler, the dispatcher first searches for a registered handler corresponding to the type of the given value. If no such handler is found, the JSON key's value will invariably read "no json handler implemented for: _provided_type_".
- *Encapsulation*: To prevent misuse, strong encapsulation is employed in the implementation by defining both the dispatcher itself and all associated handlers as local. 
- *Duck typing*: For userdata, the corresponding handler attempts to call a ctime method on the provided value, applying RFC3339 as the JSON key's value if available and invariably read 'unsupported userdata type' if not. See also todo.
- *Divide and conquer*: When handling tables, the corresponding handler uses implicit recursive calls to itself via the dispatcher, employing a 'divide and conquer' approach, instead of handling type dispatching by itsself.
- *Nil-safe*: By treating nil values as first-class citizens, the script is able to handle them properly, thereby preventing any crashes in the script engine.
- *Fallback parameters*: A missing key- or filename will invariably read as JSON key "missing key". A missing filename invariably lead to "log.json".

## TODO
- *Optional option pattern*: Adding optional custom userdata handlers, so one can add other engine classes described in lua_help.scipt like `profile_store` 
