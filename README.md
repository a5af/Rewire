# Rewire (v0.3.0)

**Rewire** is a Roblox library that enables **Hot Reloading** of modules while your game is running in Studio play mode.

## 🔥 What is Hot Reloading?

Hot Reloading means you can **edit code during runtime** and see the results immediately without stopping and restarting the session. This is especially powerful for UI, systems tuning, or gameplay prototyping.

> Example:  
> [Watch Hot Reloading a Roact UI in action](https://user-images.githubusercontent.com/6133296/161100007-9e6616f1-01ca-4d1d-9812-270fbc238433.mp4)

---

## 🚀 How to Use

### 1. Create a HotReloader

```lua
local Rewire = require(PATH_TO_REWIRE)
local reloader = Rewire.HotReloader.new()
```

### 2. Listen to a ModuleScript

```lua
local myInstance = nil

reloader:listen(MODULE_TO_WATCH,
	function(module, context)
		print("🔁 Reloading module:", module)

		local ok, result = pcall(function()
			return require(module)
		end)

		if not ok then
			warn("❌ Failed to reload module:", result)
			return
		end

		if myInstance then
			myInstance:Destroy()
		end

		myInstance = result() -- for example, this could return a UI
	end,
	function(_, _)
		print("🧹 Cleaning up before reload")
		if myInstance then
			myInstance:Destroy()
			myInstance = nil
		end
	end
)
```

> **Tip:** Always use `pcall(require(...))` to guard against syntax or runtime errors in the reloaded module.

---

## ⚙️ Rojo & Server Sync

To enable hot reloading in a **Rojo + Play Solo** environment, you **must launch both a server and a client**, and Rojo must be syncing with the **server process**, not the client.

### ✅ Setup Checklist:

- Use `rojo serve` to sync your `default.project.json` to the server.
- In Studio, start **Play > Start (Server + 1 Player)**.
- Reconnect Rojo to the **server instance**.
- Rewire will now hot-reload on the **client side** when files change.

> **Note:** Rewire does **not reload modules at runtime on live servers**. Outside Studio, it just fires the callback once.

---

## 🧠 `Context` Parameter (New in v0.3.0)

As of v0.3.0, Rewire passes a `context` object to both the reload and cleanup callbacks. You can use this to tell whether the module is reloading or running for the first time.

```lua
type Context = {
	originalModule: ModuleScript, -- the original watched module
	isReloading: boolean,         -- true if this is a reload, false for first run
}
```

---

## 🛠 How It Works

Rewire watches `ModuleScript` instances for changes. Because Roblox caches required modules, Rewire **clones** the module when reloading to bypass that cache.

### 📛 Clone Tagging

All cloned modules are tagged using `CollectionService` with a special identifier:

```lua
Rewire.CollectionServiceTag -- e.g. "__rewire_clone"
```

You can use this tag to filter Rewire clones out of your logic (e.g. in `ChildAdded` listeners).

---

## 🧪 Example Use Cases

- Live-editing Roact or Fusion UI while playtesting
- Real-time tweaking of gameplay logic or behaviors
- Avoiding Studio restart cycles for every small code change

---

## 📌 Limitations

- Rewire only works in **Studio** (not in published or live games).
- Only ModuleScripts can be hot-reloaded.
- Requires Rojo syncing to the **server process** for live reloading.

---

## ✅ Final Notes

Use Rewire to dramatically speed up your iteration time while building systems, UI, and game logic. For safety and reliability, always wrap `require(...)` in a `pcall` and handle errors gracefully to avoid crashes during reloads.

Happy hot-reloading!
