# The giu library



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
