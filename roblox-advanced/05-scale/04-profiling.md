# Profiling

*Find where the frame goes with the MicroProfiler and your own labels, and read the Developer Console's other tools, before changing anything.*

Roblox Advanced · Scale · 10 min

---

Players report a hitch every few seconds. You are sure it is the NPC loop, so you spend an afternoon making it twice as fast, and the hitch stays: the cause was a leaderboard refresh that runs every five seconds and fills one frame.

A guess optimises the code you were already looking at. A profiler shows where the frame went, so **measure first**, change one thing, and measure again.

## The MicroProfiler

The **MicroProfiler** shows how long each frame took and what ran inside it. Press Ctrl+F6 (⌘F6) in Studio or the desktop client. A bar graph along the top has one bar per frame, taller for a longer frame; below it, a timeline shows the frame's tasks, wider for longer ones.

To see a section of your own code as its own bar, label it. `debug.profilebegin(label)` opens a label, and `debug.profileend()` closes the most recent one.

**Labels nest**

```luau
local RunService = game:GetService("RunService")

RunService.Heartbeat:Connect(function()
	debug.profilebegin("Wave")
	for _, enemy in enemies do
		debug.profilebegin("Pathing")
		updatePath(enemy)
		debug.profileend() -- closes "Pathing"
	end
	debug.profileend() -- closes "Wave"
end)
```

On the timeline, "Pathing" appears beneath "Wave" as part of it. A parent label can never be shorter than its children added up.

**Two ways to close on every path**

```luau
-- Open the label after the early return
local function spawnWave(wave)
	if wave.count == 0 then
		return
	end
	debug.profilebegin("SpawnWave")
	for i = 1, wave.count do
		spawnEnemy(wave.kind)
	end
	debug.profileend()
end

-- Or label the call: every return is inside it
debug.profilebegin("SpawnWave")
spawnWave(wave)
debug.profileend()
```

Labelling the call site keeps the function untouched, and the label closes however the function returns. A function that can error still skips `profileend`.

## The Developer Console

| Tool | Where | It answers |
| --- | --- | --- |
| MicroProfiler | Ctrl+F6 in Studio or the client; the Developer Console records server frames | Which task made this frame long? |
| Script Profiler | Developer Console, client or server | Which functions used the most CPU time over a session? |
| Memory | Developer Console | How much memory each category uses, with Luau heap snapshots to compare over time |
| Network | Developer Console | Every web call, DataStore requests included: how many, how many failed, and how long they took |

Open the Developer Console with F9 in a test or a live game. Server data needs edit permission on the game.

Compare like with like: the same scene on the same device, before and after one change. The MicroProfiler guide calls the mobile client the best place to profile, because most players are on phones and phones struggle first. Luau Advanced's lesson on measuring covers fair comparisons, and its last module covers making Luau code itself cheaper.

> **In the sandbox**
>
> The sandbox's clock is virtual, so there are no timings to read. It records each `profilebegin` and `profileend`, so the exercise checks that your labels are named and balanced, not how long they took.

**Key points**

- Measure first: a profile shows where the frame went, and a guess only shows where you were looking.
- The MicroProfiler (Ctrl+F6) shows each frame's tasks. `debug.profilebegin` and `debug.profileend` add your own labels, which nest.
- `profileend` closes the most recent label, so every path out of a section must reach it. Labelling the call site is the easy way.
- The Developer Console (F9) adds the Script Profiler, which records client or server scripts, and the Memory and Network tools.
