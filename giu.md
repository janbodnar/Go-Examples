# The giu library

The giu is a rapid cross-platform GUI framework for Go based on Dear ImGui. Dear ImGui is a  
bloat-free graphical user interface library for C++.  

Dear ImGui uses the immediate mode GUI paradigm to define interfaces. Unlike in traditional  
toolkits (such as Qt, Gtk or Swing) where we create separate UI objects and manage their state,  
we directly tell the GUI library what to draw in each frame.  

Immediate mode GUIs are often used in game development.  

The giu is a Go wrapper over the Dear ImGui C interface. It uses a convenient builder  
pattern to construct widgets.   

## A column of labels

```go
package main

import (
	"github.com/AllenDang/giu"
)

func loop() {
	giu.SingleWindow().Layout(
		giu.Label("an old falcon"),
		giu.Label("an old falcon"),
		giu.Label("an old falcon"),
		giu.Label("an old falcon"),
		giu.Label("an old falcon"),
		giu.Label("an old falcon"),
		giu.Label("an old falcon"),
	)
}

func main() {
	wnd := giu.NewMasterWindow("Labels", 450, 400, 0)
	wnd.Run(loop)
}
```


## A column of rows with spacing

```go
package main

import (
	g "github.com/AllenDang/giu"
)

func loop() {
	g.SingleWindow().Layout(

		g.Row(
			g.Label("an old falcon"),
			g.Label("an old falcon"),
			g.Label("an old falcon"),
		),

		g.Row(
			g.Label("an old falcon"),
			g.Label("an old falcon"),
			g.Label("an old falcon"),
		),

		g.Spacing(),
		g.Separator(),
		g.Spacing(),

		g.Row(
			g.Label("an old falcon"),
			g.Spacing(),
			g.Label("an old falcon"),
			g.Spacing(),
			g.Label("an old falcon"),
		),

		g.Row(
			g.Label("an old falcon"),
			g.Spacing(),
			g.Label("an old falcon"),
			g.Spacing(),
			g.Label("an old falcon"),
		),

		g.Spacing(),
		g.Separator(),
		g.Spacing(),

		g.Row(
			g.Label("an old falcon"),
			g.Label("an old falcon"),
			g.Label("an old falcon"),
		),

		g.Row(
			g.Label("an old falcon"),
			g.Label("an old falcon"),
			g.Label("an old falcon"),
		),
	)
}

func main() {
	wnd := g.NewMasterWindow("Labels", 450, 400, 0)
	wnd.Run(loop)
}
```

## Alignment 

```go
package main

import (
	"github.com/AllenDang/giu"
)

func loop() {
	giu.SingleWindow().Layout(

		giu.Align(giu.AlignCenter).To(
			giu.Label("an old falcon"),
			giu.Label("an old falcon"),
			giu.Label("an old falcon"),
			giu.Label("an old falcon"),
		),

		giu.Align(giu.AlignRight).To(
			giu.Label("an old falcon"),
			giu.Label("an old falcon"),
			giu.Label("an old falcon"),
			giu.Label("an old falcon"),
		),

		giu.Align(giu.AlignLeft).To(
			giu.Label("an old falcon"),
			giu.Label("an old falcon"),
			giu.Label("an old falcon"),
			giu.Label("an old falcon"),
		),
	)
}

func main() {
	wnd := giu.NewMasterWindow("Alignment demo", 640, 480, 0)
	wnd.Run(loop)
}
```
