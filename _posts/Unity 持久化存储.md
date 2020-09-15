# Unity 持久化存储

在游戏中，数据的持久化分为两种，一是存储到服务器中，二是存储到玩家的手机存储中，存到服务器中的数据一致不会丢失，而存到玩家本地的，玩家可以自己删除，在unity中，本地持久化通过**PlayerPrefs**类来管理。

1. unity3D中的数据持久化是以键值的形式存储的，可以看作是一个字典。
2. Unity3D中值是通过键名来读取的，当值不存在时，返回默认值。
3. 目前Unity3D中只支持int、string、float三种数据类型的读取。

```c#
 //保存数据
 PlayerPrefs.SetString("Name", mName);
 PlayerPrefs.SetInt("Age", mAge);
 PlayerPrefs.SetFloat("Grade", mGrade)

//读取数据
mName = PlayerPrefs.GetString("Name", "DefaultValue");
mAge = PlayerPrefs.GetInt("Age", 0);
mGrade = PlayerPrefs.GetFloat("Grade", 0F);
```

## 游戏中的使用

```lua
function PreferenceMgr:ctor()
    self:load()
end

function PreferenceMgr:load()
    local data = LuaClass.PlayerPrefs.GetString("data", "")
    local hash = LuaClass.PlayerPrefs.GetString("hash", "")
    if hash == CS.BombMan.Security.ComputeHash(data) then
        self._data = load("return " .. data .. "")()
        return
    end
    if data == "" then
        self._data = {}
        return
    end
    error("Invalid save game hash")
    self._data = {}
end

function PreferenceMgr:onApplicationQuit()
    self:trySave()
end

function PreferenceMgr:onApplicationPause()
    self:trySave()
end

function PreferenceMgr:trySave()
    self:save()
end

function PreferenceMgr:save()
    local text = serialize(self._data)
    local hash = CS.BombMan.Security.ComputeHash(text)
    LuaClass.PlayerPrefs.SetString("data", text)
    LuaClass.PlayerPrefs.SetString("hash", hash)
end

function PreferenceMgr:set(key, value, isAppendPlayerId)
    if isAppendPlayerId then
        key = key .. "_" .. App.playerMgr.playerData.playerId
    end
    self._data[key] = value
end

function PreferenceMgr:get(key, def, isAppendPlayerId)
    if isAppendPlayerId then
        key = key .. "_" .. App.playerMgr.playerData.playerId
    end
    return self._data[key] or def
end

function PreferenceMgr:saveToDisk()
    LuaClass.PlayerPrefs.Save()
end

function PreferenceMgr:hasKey(key)
    return self._data[key] ~= nil
end
```

## 游戏中的持久化

```lua
---@class PersistenceMgr
---@field language string
---@field openId string
local PersistenceMgr = class("PersistenceMgr")

function PersistenceMgr:ctor()
    self:defineGetSet("language", LuaClass.PersistentString("currentLanguage", "Cn"))
    self:defineGetSet("openId", LuaClass.PersistentString("currentOpenId", ""))

    print("currentLanguage = ", self.language)
end

--设置get set
function PersistenceMgr:defineGetSet(key, value)
    self["_" .. key] = value
    --LuaClass.GetSet.defineProperty(self, key)
    LuaClass.GetSet.defineProperty(self, key, {
        get = function()
            return value:get()
        end,
        set = function(v)
            value:set(v)
        end
    })
end

--eg：
--App.persistenceMgr.language 就是get
--App.persistenceMgr.language = "Cn" 就是set
```



get and set 文件

```lua
--[[
getset.lua
A library for adding getters and setters to Lua tables.
Copyright (c) 2011 Josh Tynjala
Licensed under the MIT license.
]]--

local function throwReadOnlyError(table, key)
	error("Cannot assign to read-only property '" .. key .. "' of " .. tostring(table) .. ".");
end

local function throwNotExtensibleError(table, key)
	error("Cannot add property '" .. key .. "' because " .. tostring(table) .. " is not extensible.")
end

local function throwSealedError(table, key)
	error("Cannot redefine property '" .. key .. "' because " .. tostring(table) .. " is sealed.")
end

local function getset__index(table, key)
	local gs = table.__getset
	local old__index = gs.old__index

	-- try to find a descriptor first
	local descriptor = gs.descriptors[key]
	if descriptor then
		if descriptor.get then
			return descriptor.get()
		else
			local result = old__index(table, "_" .. key)
			if result ~= nil and result.get then
				return result:get()
			end
		end
	end
	
	-- if an old metatable exists, use that
	if old__index then
		return old__index(table, key)
	end
	
	return nil
end

local function getset__newindex(table, key, value)
	local gs = table.__getset

	-- check for a property first
	local descriptor = gs.descriptors[key]
	if descriptor then
		if descriptor.set then
			descriptor.set(value)
			return
		else
			local result = gs.old__index(table, "_" .. key)
			if result ~= nil then
				if type(result) == "table" and result.set  then
					return result:set(value)
				else
					local old__newindex = gs.old__newindex
					if old__newindex then
						old__newindex(table, "_" .. key, value)
						return
					end
				end
			else
				throwReadOnlyError(table, key)
			end
		end

	end

	-- use the __newindex from the previous metatable next
	-- if it exists, then isExtensible will be ignored
	local old__newindex = gs.old__newindex
	if old__newindex then
		old__newindex(table, key, value)
		return
	end
	
	-- finally, fall back to rawset()
	if gs.isExtensible then
		rawset(table, key, value)
	else
		throwNotExtensibleError(table, key)
	end
end

-- initializes the table with __getset field
local function initgetset(table)
	if table.__getset then
		return
	end
	
	local mt = getmetatable(table)
	local old__index
	local old__newindex
	if mt then
		if type(mt.__index) == "function" then
			old__index = mt.__index
		else
			local oldCls = mt.__index
			old__index = function(o, k)
				local r = rawget(o, k)
				if r ~= nil then
					return r
				end
				return oldCls[k]
			end
		end

		if not mt.__newindex then
			old__newindex = function(o, k, v)
				return rawset(o, k, v)
			end
		else
			old__newindex = mt.__newindex
		end

	else
		mt = {}
		setmetatable(table, mt)
	end
	mt.__index = getset__index
	mt.__newindex = getset__newindex
	rawset(table, "__getset",
	{
		old__index = old__index,
		old__newindex = old__newindex,
		descriptors = {},
		isExtensible = true,
		isOldMetatableExtensible = true,
		isSealed = false
	})
	return table
end

---@class GetSet
local getset = {}

local empty = {}

--- Defines a new property or modifies an existing property on a table. A getter
-- and a setter may be defined in the descriptor, but both are optional.
-- If a metatable already existed, and it had something similar to getters and
-- setters defined using __index and __newindex, then those functions can be 
-- accessed directly through table.__getset.old__index() and
-- table.__getset.old__newindex(). This is useful if you want to override with
-- defineProperty(), but still manipulate the original functions.
-- @param table			The table on which to define or modify the property
-- @param key			The name of the property to be defined or modified
-- @param descriptor	The descriptor containing the getter and setter functions for the property being defined or modified
-- @return 				The table and the old raw value of the field
function getset.defineProperty(table, key, descriptor)
	initgetset(table)
	
	local gs = table.__getset
	
	local oldDescriptor = gs.descriptors[key]
	local oldValue = table[key]
	
	if gs.isSealed and (oldDescriptor or oldValue) then
		throwSealedError(table, key)
	elseif not gs.isExtensible and not oldDescriptor and not oldValue then
		throwNotExtensibleError(table, key)
	end
	
	gs.descriptors[key] = descriptor or empty
	
	if descriptor then
		-- we need to set the raw value to nil so that the metatable works
		rawset(table, key, nil)
	end
	
	-- but we'll return the old raw value, just in case it is needed
	return table, oldValue
end

--- Prevents new properties from being added to a table. Existing properties may
-- be modified and configured.
-- @param table		The table that should be made non-extensible
-- @return			The table
function getset.preventExtensions(table)
	initgetset(table)
	
	local gs = table.__getset
	gs.isExtensible = false
	return table
end

--- Determines if a table is extensible. If a table isn't initialized with
-- getset, this function returns true, since regular tables are always
-- extensible. If a previous __newindex metatable method was defined before
-- this table was initialized with getset, then isExtensible will be ignored
-- completely.
-- @param table		The table to be checked
-- @return			true if extensible, false if non-extensible
function getset.isExtensible(table)
	local gs = table.__getset
	if not gs then
		return true
	end
	return gs.isExtensible
end

--- Prevents new properties from being added to a table, and existing properties 
-- may be modified, but not configured.
-- @param table		The table that should be sealed
-- @return			The table
function getset.seal(table)
	initgetset(table)
	local gs = table.__getset
	gs.isExtensible = false
	gs.isSealed = true
	return table
end

--= Determines if a table is sealed. If a table isn't initialized with getset,
-- this function returns false, since regular tables are never sealed.
-- completely.
-- @param table		The table to be checked
-- @return			true if sealed, false if not sealed
function getset.isSealed(table)
	local gs = table.__getset
	if not gs then
		return false
	end
	return gs.isSealed
end
		
return getset
```

