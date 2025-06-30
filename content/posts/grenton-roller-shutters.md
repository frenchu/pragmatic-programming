---
author: Paweł Weselak
title: Script for Grenton roller shutters
description: Using Object Oriented Paradigm in Lua to control Grenton RollerShutter module
date: 2024-08-31
tags:
  - "grenton"
  - "smarthome"
categories:
  - "home automation"
images
  - "/images/shutters.jpg"
---

> [!Warning] Warning!  
> The Grenton CLU does not validate scripts. `Lua` is not a strongly typed language, and many issues in scripts only surface at runtime. The CLU may enter _emergency mode_ if an error in the script is encountered during execution. In such cases, you should fix the script or restore a previously working configuration. I recommend making a backup of your project before uploading complex scripts and making or applying changes to scripts incrementally.

{{< figure alt="Old fashioned and rusty shutters" src="/images/shutters.jpg" caption="Shutters" >}}

In `Lua`, objects are _de facto_ tables. A table is a map of key-value pairs, and the map's keys can be treated like object fields. You can refer to these fields using the `.` operator. With the use of `Metatables` and `Metamethods`, you can create classes and instances of those classes. This technique is used in the `Shutter:new` function. The special variable `self` is equivalent to `this` in other programming languages. `Lua` is a dynamic language and in many ways resembles `JavaScript`.

## Controlling Roller Shutters

The example below demonstrates how to use object-oriented features of `Lua` to control shutters and groups of shutters. The variable `Shutter` acts as an interface for the objects representing individual shutters. The default implementation of the `up` and `down` methods iterates over nested objects. The implementation provided at the end of this post allows you to create hierarchies/nesting of shutter objects. A leaf object in the hierarchy has conrete implementations of `up` and `down` methods that control a specific module. Consider the following hierarchy as an example:

```
|-GroundFloor
  |-Garage
  |-Kitchen
    |-Left
    |-Right
  |-LivingRoom
    |-North
    |-East
|-Upstairs
  |-Bedroom
    |-Bigger
    |-Smaller
  |-Office
```

Shutters can be controlled individually or with groups using the appropriate nesting level. Example calls for the configuration above:

```lua
-- lower all shutters in the house:
zWave->Shutters:down()

-- lower shutters on the ground floor:
zWave->Shutters.GroundFloor:down()

-- lower shutters in the bedroom:
zWave->Shutters.Upstairs.Bedroom:down()

-- raise a specific shutter:
zWave->Shutters.GroundFloor.Kitchen.Left:up()
```

Complete code implementing a different example hierarchy:

```lua
local Shutter = {
  up = function(self)
    self.op(self, function(o) o:up() end)
  end,
  
  down = function(self)
    self.op(self, function(o) o:down() end)
  end,
  
  op = function(self, fclosure)
    for k, shutter in pairs(self) do
      if type(shutter) == "table" then
        fclosure(shutter)
      end
    end
  end
}

function Shutter:new (o)
  o = o or {}
  setmetatable(o, self)
  self.__index = self
  return o
end

local shutters = Shutter:new{
  Office = Shutter:new{
    up = function()
      zWave->x100000001_ROLLER_SHUTTER_01->MoveUp(0)
    end,
    down = function()
      zWave->x100000001_ROLLER_SHUTTER_01->MoveDown(0)
    end,
  },
  LivingRoom = Shutter:new{
    East = Shutter:new{
      up = function()
        zWave->x100000001_ROLLER_SHUTTER_02->MoveUp(0)
      end,
      down = function()
        zWave->x100000001_ROLLER_SHUTTER_02->MoveDown(0)
      end,
    },
    South = Shutter:new{
	  Left = Shutter:new{
	    up = function()
	      zWave->x100000002_ROLLER_SHUTTER_01->MoveUp(0)
	    end,
	    down = function()
	      zWave->x100000002_ROLLER_SHUTTER_01->MoveDown(0)
	    end,
	  },
	  Right = Shutter:new{
	    up = function()
	      zWave->x100000002_ROLLER_SHUTTER_02->MoveUp(0)
	    end,
	    down = function()
	      zWave->x100000002_ROLLER_SHUTTER_02->MoveDown(0)
	    end,
	  }
	},
    West = Shutter:new{
      Left = Shutter:new{
        up = function()
          zWave->x100000003_ROLLER_SHUTTER_01->MoveUp(0)
        end,
        down = function()
          zWave->x100000003_ROLLER_SHUTTER_01->MoveDown(0)
        end,
      },
      Right = Shutter:new{
        up = function()
          zWave->x100000003_ROLLER_SHUTTER_02->MoveUp(0)
        end,
        down = function()
          zWave->x100000003_ROLLER_SHUTTER_02->MoveDown(0)
        end,
      }
    }
  }
}

zWave->Shutters = shutters
```

## Installation

1. Create a user-defined variable in `user features` of type `OTHER` with an initial value of `0`
2. Create a new script as described above
3. Set the script to run on the CLU's `OnInit` event
4. Set the function call, e.g. `zWave->Shutters:down()`, as the action for a panel button or wall switch key

## Final Notes

To better understand object-oriented features of `Lua` and the related syntax, I recommend reviewing the bank account example in the [Programming in Lua](https://www.lua.org/pil/16.html) manual.

An alternative to the presented implementation could be grouping shutters using bit fields stored in a single `Number` variable, using bitwise operations like AND, OR, etc. This code would likely be less readable but simpler from the perspective of the `Lua` interpreter in the CLU.

You can also try calling methods directly on hidden module objects. For example:

```lua
ROL1234:execute(0, 0)
```

The first parameter of the `execute` method is the operation code (`MoveUp`, `MoveDown`, etc.). The second parameter is the actual argument for the operation (e.g., `time`). There is no documentation explaining what each code means — you have to discover them through _reverse-engineering_.  
Module objects can be encapsulated/decorated in your own objects or put in an array and iterated over.
The problem is that after a CLU rediscovery, object identifiers may change.

## About Grenton

Grenton is a smart home system developed in Poland. It is highly customizable thanks to `Lua` scripts that can provided by the user them self. It comprises set of modules and external panels that can be connected together with the communication bus. System configuration is build on and deployed with proprietary IDE based on Eclipse. CLU is a central unit keeping track of the current state of the system and controlling other modules. To some extend modules can work independently from CLU in a distributed logic mode without connection to the central unit. 

## Resources

- https://www.lua.org/pil/contents.html
- https://www.lua.org/pil/13.4.1.html
- https://www.lua.org/pil/16.html
- https://www.lua.org/manual/5.4/contents.html
- https://knowledgebase.grenton.com/
