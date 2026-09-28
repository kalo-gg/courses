# Debugging in Studio

*Read an error to its line, pause a script with breakpoints and Watch, and pick the test mode that shows the bug.*

Roblox Fundamentals · Shipping a place · 11 min

---

You start a test, walk onto the lava, and nothing happens. The Output has one red line, and it already names the script, the line, and what was nil. Most bugs end there.

When the line is not enough, Studio's debugger pauses the script mid-run so you can look at every variable, and its test modes run the place with no player, one, or several.

## Reading an error

**ServerScriptService.Main**

```luau
local stats = player:FindFirstChild("Leaderstats") -- nil: the folder is "leaderstats"
stats.Coins.Value += 1 -- line 14

-- Output:
-- ServerScriptService.Main:14: attempt to index nil with 'Coins'
```

The script's full name, the line, then the message. `with 'Coins'` names the key being looked up; the nil is whatever came before it. Click the line in the Output to jump to it.

A **breakpoint** pauses a script at a line. Click the gutter beside the line number (a red circle appears) and start a playtest. When the script reaches it, hover any variable to see its value, or open the **Watch** window for everything in scope plus expressions you type. Step Over (F10), Step Into (F11) and Step Out (Shift+F11) move a line at a time.

Right-click the gutter for a **conditional** breakpoint, which pauses only when an expression is true, or a **logpoint**, which prints and does not pause.

```luau
lava.Touched:Connect(function(otherPart)
	-- A logpoint on this line with the message
	--   "hit:", otherPart:GetFullName()
	-- prints what this print would:
	print("hit:", otherPart:GetFullName())
end)
```

A logpoint's message is written like `print`'s arguments. It is not part of the script's source, so there is no stray `print` to clean up afterwards.

## Test, Run and several players

| Mode | What happens | Use it for |
| --- | --- | --- |
| **Test** (F5) | Your avatar joins. A server and your client both run; the Client/Server toggle picks which one you inspect. | Anything that needs a player: joining, touching, dying |
| **Test Here** | The same, with your avatar in front of the camera | One spot on a big map |
| **Run** (F8) | The world simulates with no avatar, so no player joins | Scripts that only drive the world: spinners, blinking parts |
| **Server & Clients** | A server and 1 to 8 clients, each in its own window | Several players at once: teams, races, shared doors |

During a test, the Output tags each message green for the server or blue for the client.

In a live game, F9 opens the Developer Console. Its Log has a Client/Server switch, and only the owner and group members who can edit the place see the Server view. A player reporting a bug sees their client's errors, never your server's.

> **Recognising older code**
>
> Old tutorials and free models still run, but many of their shapes are out of date. `wait()`, `spawn()` and `delay()` are now `task.wait`, `task.spawn` and `task.delay`. `:connect()` is `:Connect()`. `Instance.new("Part", workspace)` becomes setting `Parent` last. Advice about `FilteringEnabled` is obsolete: it is always on.

**Key points**

- An error reads *script:line: message*. In `index nil with 'X'`, X is the key; the nil is the value before it.
- Breakpoints pause at a line; hover or the Watch window shows values; F10, F11 and Shift+F11 step.
- Conditional breakpoints pause only when an expression is true; logpoints print without pausing.
- Test (F5) adds your avatar, Run (F8) adds no player, and Server & Clients runs up to eight clients.
- Server output is green and client output blue; in a live game only editors see the server log in F9.
