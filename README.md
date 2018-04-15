# Resources
[Resources](https://github.com/RoStrap/Resources/blob/master/Resources.module.lua) is the core resource-manager and library-loader for [RoStrap](https://www.roblox.com/library/725884332/RoStrap). It is designed to simplify the loading of libraries and unify the networking of resources between the client and server.

## Set-up
[Resources](https://github.com/RoStrap/Resources/blob/master/Resources.module.lua) is automatically installed when you setup using the plugin, which can installed after clicking the logo [below](https://www.roblox.com/library/725884332/RoStrap):

[![](https://avatars1.githubusercontent.com/u/22812966?v=4&s=100)](https://www.roblox.com/library/725884332/RoStrap)

After installing, you should have a [Folder](http://wiki.roblox.com/index.php?title=API:Class/Folder) called `Repository` in either [ServerStorage](http://wiki.roblox.com/index.php?title=API:Class/ServerStorage) or [ServerScriptService](http://wiki.roblox.com/index.php?title=API:Class/ServerScriptService). This `Repository` is where all of your Libraries will reside. Only Folders and ModuleScripts (and their children) should go in this `Repository`.

## Initialization
To start using the module, simply `require` it.
```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Resources = require(ReplicatedStorage:WaitForChild("Resources"))
```

## For the impatient
Here is a demonstration of the API:
```lua
-- requires a library called `Maid`
local Maid = Resources:LoadLibrary("Maid") 

-- Gets a RemoteEvent called "Chatted" within Resources.RemoteEvents
-- On the server, it will generate Folder "RemoteEvents" or RemoteEvent "Chatted" if missing
-- On the client, it will yield for both
local ChatEvent = Resources:GetRemoteEvent("Chatted")


-- Retrieves a non-replicated table hashed at key "Shared" (which doesn't need to be a string)
local Shared = Resources:GetLocalTable("Shared")
```

## Terminology
#### Library
A Library is constituted of a [ModuleScript](http://wiki.roblox.com/index.php?title=API:Class/ModuleScript) **and its descendants**. In the following image, there are only **two** Libraries; `Keys` and `Rbx_CustomFont`. These two will be accessible through the `LoadLibrary` function and the descendants of `Rbx_CustomFont` will not be (even though `Rbx_CustomFont` will internally `require` them through `require(script.Roboto)`).

![](https://user-images.githubusercontent.com/15217173/38775144-25b833f0-4038-11e8-9545-952f1634148b.png)

#### Local
Anything with "Local" in the name refers to a function that does not replicate across the client-server boundary.

## Functionality and API
#### LoadLibrary
When the server first requires `Resources`, it will move the Libraries within `Repository` to [ReplicatedStorage](http://wiki.roblox.com/index.php?title=API:Class/ReplicatedStorage) (under `Resources.Libraries`), where both the client and server can require them via the function `LoadLibrary`. Notice how folder heiarchy is ignored.

![](https://image.prntscr.com/image/ZonjgCDFQLabru0xbMBUNQ.png)

`LoadLibrary` is a require-by-string function which caches the results, so a Library only has `require` called on it once. To `require` the `Maid` Library (assuming it is installed), we could do the following:

```lua
local Maid = Resources:LoadLibrary("Maid")
-- Requires Library with name "Maid"
-- @param string LibraryName The name of the library you wish to require
-- @returns the result of require(Library)
```

If so desired, **all** methods of `Resources` have hybrid syntax, meaning both method `:` and member `.` syntaxes are supported:

```lua
local require = Resources.LoadLibrary
local Maid = require("Maid")
```

To make a Library **server-only**, give them "Server" in the name or make them a descendant of a [Folder](http://wiki.roblox.com/index.php?title=API:Class/Folder) with "Server" in the name (not case sensitive). This will make the [plugin](https://www.roblox.com/library/725884332/RoStrap) assign the Library with the tag `ServerLibraries`. Libraries moved into [ReplicatedStorage](http://wiki.roblox.com/index.php?title=API:Class/ReplicatedStorage) are tagged with `ReplicatedLibraries`.

Note: Internally, `LoadLibrary` caches and returns `require(Resources:GetLibrary(LibraryName))`. `GetLibrary` is the only function that can retrieve objects from both `ServerStorage` and `ReplicatedStorage`. `GetLocalLibrary` is technically valid, but unnecessary.

## Procedurally Generated Get Functions
Functions can be procedurally generated by `Resources`, in the form of `GetCLASSNAME` with string parameter `Name`. These functions return an instance under `ReplicatedStorage.Resources.CLASSNAMES`. **On the server, missing instances will be generated via [Instance.new](http://wiki.roblox.com/index.php?title=Instance_(Data_Structure)). On the client, the function will yield for the missing instances via [WaitForChild](http://wiki.roblox.com/index.php?title=API:Class/Instance/WaitForChild).**

```lua
local Chatted = Resources:GetRemoteEvent("Chatted")
-- Searches for ReplicatedStorage.Resources.RemoteEvents.Chatted
-- If the Folder `RemoteEvents` or the RemoteEvent `Chatted` do not exist then:
--    on the server, they will be generated
--    on the client, the function will yield until they have been replicated

local ClientLoaded = Resources:GetRemoteFunction("ClientLoaded")
```

![](https://user-images.githubusercontent.com/15217173/38775951-d6bfbeee-404b-11e8-8396-9666a0b20b98.png)

Any instance type is compatible with `Resources`:

```lua
-- Not sure why you would need to, but this retrieves a TextLabel
-- called "Superman" inside ReplicatedStorage.Resources.TextLabels
local Superman = Resources:GetTextLabel("Superman")
```

In fact, `Resources` can also manage instance types that aren't creatable by `Instance.new`. They must however, be preinstalled into `Replicated.Resources`. This allows you to do things like the following:

```lua
local Falchion = Resources:GetSword("Falchion")
-- As long as there exists a Resources.Swords.Falchion, it will be retrieved

-- The function generator will remove "^Get" from "GetSword"
-- It will then add "s" to "Sword" and thus expect the Folder to be named "Swords".
-- Library (and Accessory, and any type ending in 'y') will be in Folder "Librar" .. ies"
```

![](https://user-images.githubusercontent.com/15217173/38775984-64af0b2e-404c-11e8-9279-0adace656665.png)

#### GetLocal Functions
If you want to access local storage (not replicated across the client-server model), you can add `Local` before `CLASSNAME` to access it. On the server, `LOCALSTORAGE` is located in [ServerStorage](http://wiki.roblox.com/index.php?title=API:Class/ServerStorage). On the client, `LOCALSTORAGE` is located in [LocalPlayer](http://wiki.roblox.com/index.php?title=API:Class/Players/LocalPlayer). Everything Resources stores goes into folders named `Resources`.

|LocalStorage|**Location**|
|:-----:|:----:|
|Server|ServerStorage.Resources|
|Client|Players.LocalPlayer.Resources|

Any computer (either client or server) can instantiate instances on local storage.

```lua
local Attacking = Resources:GetLocalBindableEvent("Attacking")
-- Finds LOCALSTORAGE.Resources.BindableEvents.Attacking and creates if missing

-- where LOCALSTORAGE is LocalPlayer on the client and ServerStorage on the Server,
-- where "Resources" and "BindableEvents" are Folder Objects,
-- and "Attacking" is a BindableEvent Object

-- Each instance not present will always be generated (on whichever computer it runs, it will NOT be replicated)
```

Here is what the above code would do if ran by the server versus what it would do if ran by a client:
![](https://user-images.githubusercontent.com/15217173/38776022-37e97934-404d-11e8-89c0-cb8b6ab8b511.png)

Note: In Play-Solo Mode, all local objects will go under `ServerStorage`, as there is no difference between the client and server. If you use identical Local-function calls on the client and server, this could cause conflicts in Play-Solo.

#### GetFolder and GetLocalFolder
These functions will **not** return Folders within `Resources.Folders`, but rather within `Resources` or `LOCALSTORAGE`. `GetFolder` returns `ReplicatedStorage.Resources.FOLDER_NAME` and `GetLocalFolder` returns `LOCALSTORAGE.Resources.FOLDER_NAME` where `FOLDER_NAME` is the parameter passed.

If you wanted an array of all Libraries (in Folder `ReplicatedStorage.Resources.Libraries`), you could do the following:
```lua
-- Will WaitForChild on the client, and Instance.new on the server if non-existent
local Libraries = Resources:GetFolder("Libraries"):GetChildren()
for i = 1, #Libraries do
  print(Libraries[i]:GetFullName())
end

local TweenIsInstalled = Libraries:FindFirstChild("Tween")

if TweenIsInstalled then
  print("Tween is installed in ReplicatedStorage.Resources.Libraries!")
else
  print("Tween is not installed in ReplicatedStorage.Resources.Libraries! Could it be server-only?")
end

-- To get LocalLibraries (ServerStorage.Resources.Libraries) on the server:
local LocalLibraries = Resources:GetLocalFolder("Libraries"):GetChildren()
for i = 1, #LocalLibraries do
  print(LocalLibraries[i]:GetFullName())
end

-- To get ReplicatedStorage.Resources.BindableEvents, internally used by GetBindableEvent()
local BindableEventsFolder = Resources:GetFolder("BindableEvents")
```

![](https://image.prntscr.com/image/ZonjgCDFQLabru0xbMBUNQ.png)

Note: `GetLocalFolder` will always create folders if they do not already exist because it is a local function.

## GetLocalTable
The `GetLocalTable` function returns a (non-replicated) table hashed within `Resources` under the key passed in as a parameter. This is a convienent way to avoid Libraries that are simply `return {}`
```lua
local Shared = Resources:GetLocalTable("Shared") -- returns a table

-- In another script
local Shared = Resources:GetLocalTable("Shared") -- same table (if on the same computer)
local Shared2 = Resources:GetLocalTable("Shared2") -- different table

-- The keys don't have to be strings
local PlayerData = Resources:GetLocalTable(Players.Validark)
```

This function is also used internally, and can be used to access the internal values of `Resources`. For example:

```lua
-- Cached results from `LoadLibrary` by string
local LoadLibraries = Resources:GetLocalTable("LoadedLibraries")
-- e.g. {Maid = [require(ReplicatedStorage.Resources.Libraries.Maid)]}
```

All other hash tables internally used by `Resources` have keys identical to the [Folder](http://wiki.roblox.com/index.php?title=API:Class/Folder) names of the folders within `Resources` or `LOCALSTORAGE`.

```lua
-- Hash table of all Libraries accessible to this machine (server-only and replicated)
local LibraryObjects = Resources:GetLocalTable("Libraries")
-- e.g. {Maid = [ReplicatedStorage.Resources.Libraries.Maid]}


-- Procedurally generated tables:
local RemoteEvents = Resources:GetLocalTable("RemoteEvents")
-- This is a hash table of all RemoteEvents ever accessed through GetRemoteEvent() on this machine,
-- as well as all RemoteEvents that existed when Resources.GetRemoteEvent was first indexed
-- e.g. {Chatted = [ReplicatedStorage.Resources.RemoteEvents.Chatted]}


-- Local tables are prefixed by "Local"
local BindableEvents = Resources:GetLocalTable("LocalBindableEvents")
-- Hash table of all BindableEvents ever accessed through Resources:GetLocalBindableEvents(),
-- as well as any that get existed when Resources.GetLocalBindableEvents was first indexed
-- e.g. {Attacked = [LOCALSTORAGE.Resources.BindableEvents.Attacked]}
```

## Ideas
#### Map Changer
Try using Resources to manage other things like Maps for a game! Make a Folder named "Maps" inside `ServerStorage.Resources` and it can then be accessed!
```lua
-- Server-side
local Resources = require(ReplicatedStorage:WaitForChild("Resources"))
local Hometown = Resources:GetLocalMap("Hometown v2")
Hometown.Parent = workspace
```
#### Gun system
```lua
-- Server-side
local GunThePlayerJustBought = Resources:GetLocalGun("Ak47")
-- Finds `ServerStorage.Resources.Guns.Ak47`
-- Make sure the above exists, otherwise, Resources will error trying to do Instance.new("Gun")

GunThePlayerJustBought:Clone().Parent = PlayerBackpackThatBoughtIt
```

#### UI system
```lua
-- Client-side
local ShopUI = Resources:GetUserInterface("Shop")
-- Yields for `ReplicatedStorage.Resources.UserInterfaces.Shop`
```

## Contact
If you have any questions, concerns, or feature requests, feel free to [message Validark on roblox](https://www.roblox.com/messages/compose?recipientId=2966752). There is also a dedicated issues tab at the top of this page, should you feel inclined to use that. Please send all hatemail and/or unsavory requests to [devSparkle](https://www.roblox.com/messages/compose?recipientId=1631699).
