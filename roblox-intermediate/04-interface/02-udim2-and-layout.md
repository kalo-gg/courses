# UDim2 and layout

*Size and place UI as a fraction of its parent plus pixels, choose the point Position places, and let layout objects arrange the rest.*

Roblox Intermediate · Interface · 11 min

---

A shop panel is 500 by 300 pixels, placed at (390, 180). On your monitor it sits in the middle. On a wider monitor it sits left of centre, and on a phone held sideways it runs off the bottom edge.

Every number was in pixels. Roblox UI can also be sized as a fraction of whatever it sits in, and this lesson is about mixing the two.

## Scale and Offset

```luau
-- UDim2.new(xScale, xOffset, yScale, yOffset)
panel.Size = UDim2.new(0.5, 0, 0, 300) -- half the parent's width, 300 px tall
panel.Size = UDim2.new(0.5, -20, 1, 0) -- half the width less 20 px, full height
panel.Size = UDim2.fromScale(0.4, 0.5) -- 40% of the width, 50% of the height
panel.Size = UDim2.fromOffset(200, 50) -- 200 by 50 px on every screen
```

Each axis is a scale, a fraction of the parent's size, plus an offset in pixels added after scaling. The arguments go scale, offset, scale, offset, X before Y, so `UDim2.new(0.4, 0.4)` is 40% wide and 0.4 px tall.

`Position` is a UDim2 too, and by default it places the object's top-left corner. `AnchorPoint` changes which point it places. It is a `Vector2` measured as a fraction of the object's own size, so `(0.5, 0.5)` is its centre and `(1, 0)` its top-right corner. The object also grows from that point when its size changes.

**Centred on every screen**

```luau
local panel = Instance.new("Frame")
panel.AnchorPoint = Vector2.new(0.5, 0.5)
panel.Position = UDim2.fromScale(0.5, 0.5)
panel.Size = UDim2.fromScale(0.4, 0.5)
panel.Parent = shop
```

The panel's centre sits at the centre of its parent, whatever the screen size. Without the `AnchorPoint` its top-left corner would sit there, and the panel would hang down and to the right. `AbsolutePosition` and `AbsoluteSize` report where it ended up, in pixels.

## Layouts and padding

Placing ten shop items one `Position` at a time breaks as soon as you add an eleventh. A `UIListLayout` parented to a frame arranges its sibling GuiObjects in a column or a row instead: it ignores their `Position` and keeps each one's `Size`. A `UIPadding` insets the frame's contents from its edges.

**A padded column of items**

```luau
local list = Instance.new("UIListLayout")
list.FillDirection = Enum.FillDirection.Vertical
list.Padding = UDim.new(0, 8) -- 8 px between items
list.SortOrder = Enum.SortOrder.LayoutOrder
list.Parent = panel

local padding = Instance.new("UIPadding")
padding.PaddingTop = UDim.new(0, 12)
padding.PaddingLeft = UDim.new(0, 12)
padding.PaddingRight = UDim.new(0, 12)
padding.Parent = panel

for i, name in { "Sword", "Shield", "Potion" } do
	local item = Instance.new("TextLabel")
	item.Name = name
	item.Text = name
	item.LayoutOrder = i
	item.Size = UDim2.new(1, 0, 0, 40)
	item.Parent = panel
end
```

With `SortOrder` set to `LayoutOrder`, items go in `LayoutOrder` order; set to `Name`, they go alphabetically. `UDim.new(scale, offset)` is one axis of a UDim2.

| Object | Parent it to | What it does |
| --- | --- | --- |
| `UIGridLayout` | The container | Lays its siblings out in rows of equal cells, `CellSize` each, starting a new row when the next cell would not fit. |
| `UIAspectRatioConstraint` | The object to constrain | Keeps its width-to-height ratio at `AspectRatio`, even when its size is a scale of its parent. |
| `UICorner` | The object | Rounds its corners by `CornerRadius`. |
| `UIStroke` | A text object or a frame | Draws an outline round the text or the border, with its own `Color`, `Thickness` and `Transparency`. |

Like UIListLayout and UIPadding, each works by being a child. Set its properties and it does the rest.

> **Layout in the sandbox**
>
> The sandbox works out `AbsolutePosition` and `AbsoluteSize` from `Size`, `Position` and `AnchorPoint`, in a fixed 1280 by 720 window. It has no layout engine: a UIListLayout or UIPadding is kept as data and moves nothing. So the exercise grades a panel you place yourself.

**Key points**

- A UDim2 is a scale plus an offset on each axis: a fraction of the parent's size, then pixels added.
- `UDim2.new` takes xScale, xOffset, yScale, yOffset; `fromScale` and `fromOffset` are the one-kind shortcuts.
- `AnchorPoint` picks the point of the object that `Position` places: `(0.5, 0.5)` at `fromScale(0.5, 0.5)` centres it.
- A `UIListLayout` arranges its siblings and ignores their `Position`; a `UIPadding` insets a container's contents.
- `AbsolutePosition` and `AbsoluteSize` say where an object actually ended up, in pixels.
