<div align="center">
<h1><b>Class</b></h1>
<sup><b>Supercharge your classes</b></sup><br>
<sup><a href="https://create.roblox.com/marketplace/asset/13953067746" target="_blank">Model</a></sup>
<sup><a href="https://devforum.roblox.com/t/class-supercharge-your-classes/2446100" target="_blank">DevForum</a></sup>
</div>

<div align="center"> <h4> Preface </h4> </div> <hr>

Because of how bored I am lately and taking a short break to release my own game, I've decided, "You know what, what if I challenge myself to put in private and protected variables in LuaU classes?", and I did.

With a bit of playing around with `debug.info`, I managed to make Class, the module being presented today!

<div align="center"> <h4> What is Class? </h4> </div> <hr>

Class is a module that allows you to make your classes go super.

By how super you may ask? Class takes advantage of `debug.info`; by using the said method, we can slowly climb up from function to function until we get to a specific function that is under the class. With this, we are able to determine our current "level" within the code, thus, we can finally evaluate and interpret internal, private and protected properties!

<div align="center"> <h4> Okay so, what can Class do? </h4> </div> <hr>

Class is basically a generic class but one major alteration from the vanilla ones: we are able to make special properties. On top of that, we can initialize everything by creating `__init` method. It's best practiced to initialize everything under `class:__init()` than using the optional `defaultProps` parameter when creating a new class, as it's (supposedly) less prone to errors and other gimmicks!

The table shown below will showcase you the special properties, their prefixes and descriptions:
| Property type | Prefix | Example | Description |
| --- | --- | --- | --- |
| Constants | PROP_NAME | `self.MESSAGE` | Makes the property a constant; cannot be changed after it has been initialized under the `class:__init()` method or via the `defaultProps` option
| Internal properties | \_\_propname__ | `self.__message__` | Makes the property internal; only the source class can access it |
| Private properties | _ | `self._message` | Makes the property private; only the source class and inherited classes can access it |
| Protected properties | __ | `self.__message` | Makes the property protected; other sources outside the source class or inherited classes can read the property but cannot overwrite the said property; the source class and inherited classes can change the property |

<sup>Note, when userdatas; such as CFrames, Vector3s, Instances and such, are given as keys, they won't be assigned from any of these special property cases</sup>

Class also gives you the option to lock the property; preventing the property in detail from being changed. To lock a property, you have to call `class:__lockProperty(propName)` and to unlock it you have to call `class:__unlockProperty(propName)` instead. <sub>Note, Constants are basically locked properties,  but you cannot unlock them; doing so will rase an error.</sub>

Another addition of Class is to strictify properties, for example, if we want property `X` strictly only to be numbers, we have to do some checks and stuff; Class will introduce the `class:__strictifyProperty__(propName, predicate)` method, where `predicate` is a function that returns a `boolean` value to validate the set process.

<div align="center"> <h4> API and Examples </h4> </div> <hr>

- ## `Class(defaultProps: {}?)`
  - Creates a new `Class`
***
- ## `class.new(...any?): Class`
  - Constructs a `Class` object; any properties within the `defaultProps` option containing any of the given special character prefixes will be assigned accordingly.
- ## `class.inherits(otherClass: Class)`
  - Allows the given task to access private and protected properties of `otherClass` when both are instantiated as `Class` objects
- ## `class.extends()`
  - A simplified version of:
  ```lua
	local subClass = setmetatable({}, superClass)
	subClass.__index = subClass
  ```
- ## `class:__init(...any?)`
  - Called during `class.new()` is ran, all initialization must be done here; but it's for personal preferences; like above, any properties assigned inside the function, with any of the given special character prefixes, will be assigned accordingly.
- ## `class:__lockProperty(propName: string)`
  - Locks the given property; prevents the property in detail from being changed. <sub>Constants are locked properties by default</sub>
- ## `class:__unlockProperty(propName: string)`
  - Unlocks the given; allows the property in detail from being changed <sub>Cannot unlock Constants by default</sub>
- ## `class:__overloadTargetFunction__(target: string, expects: {string | {string}}, func: (...any) -> (...any))`
  - Allows function overloading to `target`; the `target` function should be an empty function, doing nothing, as the `func` parameter will be used instead after the conditions that the `expects` parameter is satisfied. `expects` must contain strings of the desired datatype, ie: `Vector3`, `string`, `number`, etc. On top of that, `target` is case-sensitive! Below is an example:
  ```lua
	function myClass:__init()
		self:__overloadTargetFunction("someFunction", {"number", "number"}, function(a, b)
			return a + b
		end)
		self:__overloadTargetFunction("someFunction", {"string", "string"}, function(a, b)
			return a .. b
		end)
		
		print(self:someFunction(5, 3)) -- 8
		print(self:someFunction("Hello ", "World!")) -- Hello World!
	end

	function myClass:someFunction()
	end
  ```
- ## `class:__registerSpecialHandler__(handler: (...any) -> (...any)): (() -> ())`
 - Allows C and anonymous functions to be used, this function is sugar-coated by methods such as `__wrapSignal`, `__wrapCoroutine` and `__wrapTask`.
- ## `class:__strictifyProperty__(propName: string, predicate: (value: any) -> boolean)`
  - Makes the property's value setting strict by calling `predicate` whenever `self.key = value` is done; when `predicate` returns false, it will raise an error.
- ## `class:__wrapSignal(signal: | {Connect: () -> ()}, handler: (...any) -> ())`
  - A workaround when indexing private or internal properties inside roblox signals such as `workspace.ChildAdded` and `RunService.Heartbeat` as both are rather debugged as C functions. <sub>Do **NOT** forget to wrap your signals with this method as there might be cases of script exhaustion or constant errors</sub>
- ## `class:__wrapCoroutine(co: coroutine, handler: (...any) -> ())`
  - Similar to `class:__wrapSignal()` but for `coroutine` instead
- ## `class:__wrapTask(task: task, handler: (...any) -> ())`
  - Similar to `class:__wrapSignal()` but for `task` instead

<h2><b>Examples</b></h2>

### **Example Class**
```lua
local classObject = Class({
	CONSTANT_MESSAGE = "Bye",
	publicMessage = "Hi",
	_privateMessage = "Secret",
	__protectedMessage = "Hello?"
})

function classObject:__init()
	print(self.CONSTANT_MESSAGE, self.publicMessage, self._privateMessage, self.__protectedMessage)
	-- Bye, Hi, Secret, Hello?
end

function classObject:setProtected(msg)
	self.__protectedMessage = msg
end

function classObject:changeConstant(msg)
	self.CONSTANT_MESSAGE = msg -- errors
end

local class = classObject.new()

print(class.__protectedMessage) -- "Hello?"
--class.__protectedMessage = "Hi!" -- errors
class:setProtected("Hi!") -- success
print(class.__protectedMessage)

class.publicMessage = {}
--print(class._privateMessage) -- errors

print(class.CONSTANT_MESSAGE)
class:changeConstant("Hiiii")
print(class.CONSTANT_MESSAGE)
```

### **Strict Properties**
```lua
local class = Class()
function class:__init()
    self.X = 2
    self:__strictifyProperty__('X', function(value)
        return type(value) == "number" -- we only expect numbers
    end)
    self.X = 10 -- ok
    self.X = "test" -- uh oh error!
end
-- main code
```

### **Inheritance**
```lua
local classObject = Class()
function classObject:__init()
	self._message = "I can only be accessed by myself and my successors"
	print(self._message)
end

local class = classObject.new()

local successor = Class()
successor.inherits(classObject)
function successor:__init()
	print(class._message)
end

local notSuccessor = Class()
function notSuccessor:__init()
	print(class._message)
end

successor.new() -- success
notSuccessor.new() -- fails
```

### **Benchmarking**
```lua
local BENCHMARK_COUNT = 50000

local classObject = Class()
function classObject:__init()
	self.strict = 1
	self:__strictifyProperty__('strict', function(v) return type(v) == "number" end)
end

function classObject:testPublic()
	local s = os.clock()
	for i = 1, BENCHMARK_COUNT do
		self.value = 1
	end
	print('public', os.clock()-s)
end

function classObject:testPrivate()
	local s = os.clock()
	for i = 1, BENCHMARK_COUNT do
		self._value = 1
	end
	print('private', os.clock()-s)
end

function classObject:testProtected()
	local s = os.clock()
	for i = 1, BENCHMARK_COUNT do
		self.__value = 1
	end
	print('protected', os.clock()-s)
end

function classObject:testInternal()
	local s = os.clock()
	for i = 1, BENCHMARK_COUNT do
		self.__value__ = 1
	end
	print('internal', os.clock()-s)
end

function classObject:testLock()
	local s = os.clock()
	for i = 1, BENCHMARK_COUNT do
		self:__unlockProperty('value')
		self.value = 1
		self:__lockProperty('value')
	end
	print('locking', os.clock()-s)
end

function classObject:testStrict()
	local s = os.clock()
	for i = 1, BENCHMARK_COUNT do
		self.strict = 1
	end
	print('strict', os.clock()-s)
end

function classObject:testActivity(n)
	local t = 0
	while true do
		self._actiivty = math.sin(tick())
		t += task.wait()
		if t >= (n or 5) then break end
	end
end

local class = classObject.new()
class:testPublic()
class:testPrivate()
class:testProtected()
class:testInternal()
class:testLock()
class:testStrict()
class:testActivity()
```
<div align="center"> <h4> What now? </h4> </div> <hr>

Everything is now up to you once you are using it! You have complete control over how you'll manage your newly constructed class with Class as it brings new functionalities you want complete supervision on!

And always, have fun!;)