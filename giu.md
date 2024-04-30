# The giu library

The giu is a rapid cross-platform GUI framework for Go based on Dear ImGui. Dear ImGui is a  
bloat-free graphical user interface library for C++.  

Dear ImGui uses the immediate mode GUI paradigm to define interfaces. Unlike in traditional  
toolkits (such as Qt, Gtk or Swing) where we create separate UI objects and manage their state,  
we directly tell the GUI library what to draw in each frame.  

Immediate mode GUIs are often used in game development.  

The giu is a Go wrapper over the Dear ImGui C interface. It uses a convenient builder  
pattern to construct widgets.  

## Label 

```go
package main

import (
    g "github.com/AllenDang/giu"
)

func loop() {

    g.SingleWindow().Layout(
        g.Label("An old falcon in the sky"),
    )
}

func main() {
    wnd := g.NewMasterWindow("Application", 400, 200,
        g.MasterWindowFlagsFloating)
    wnd.Run(loop)
}
```

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

## Quit button

```go
package main

import (
    "os"

    g "github.com/AllenDang/giu"
)

func loop() {
    g.SingleWindow().Layout(
        g.Button("Quit").Size(80, 30).OnClick(func() {
            os.Exit(0)
        }),
    )
}

func main() {
    wnd := g.NewMasterWindow("Button", 400, 200, 0)
    wnd.Run(loop)
}
```

## Checkbox

```go
package main

import (
    g "github.com/AllenDang/giu"
)

var cbSelected bool = true

func loop() {

    g.SingleWindow().Layout(
        g.Checkbox("Show Title", &cbSelected).OnChange(
            func() {
                if cbSelected {
                    g.Context.GetPlatform().SetTitle("CheckBox")
                } else {
                    g.Context.GetPlatform().SetTitle("")
                }
            }),
    )
}

func main() {
    wnd := g.NewMasterWindow("CheckBox", 400, 200,
        g.MasterWindowFlagsFloating)
    wnd.Run(loop)
}
```


## Canvas

```go
package main

import (
    "image"
    "image/color"

    g "github.com/AllenDang/giu"
)

func loop() {
    g.SingleWindow().Layout(
        g.Custom(func() {

            canvas := g.GetCanvas()
            col := color.RGBA{0, 140, 140, 255}

            canvas.AddLine(image.Pt(25, 25), image.Pt(100, 100), col, 1)
            canvas.AddRect(image.Pt(160, 25), image.Pt(260, 115),
                col, 5, g.DrawFlagsRoundCornersAll, 1)
            canvas.AddRectFilled(image.Pt(330, 25),
                image.Pt(430, 115), col, 5, 0)

            canvas.AddCircleFilled(image.Pt(150, 250), 60, col)
            canvas.AddTriangleFilled(image.Pt(330, 300),
                image.Pt(450, 200), image.Pt(500, 300), col)
        }),
    )
}

func main() {
    wnd := g.NewMasterWindow("Canvas", 450, 300,
        g.MasterWindowFlagsNotResizable)
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

## Key binding

```go
package main

import (
    "os"

    g "github.com/AllenDang/giu"
)

func loop() {

    g.SingleWindow().Layout(
        g.Label("Ctrl + Q to exit"),
    )
}

func main() {
    wnd := g.NewMasterWindow("Window", 400, 200, g.MasterWindowFlagsFloating).RegisterKeyboardShortcuts(
        g.WindowShortcut{
            Key:      g.KeyQ,
            Modifier: g.ModControl,
            Callback: func() { os.Exit(0) }},
    )
    wnd.Run(loop)
}
```

## Styled button

```go
package main

import (
    "fmt"
    "image/color"

    g "github.com/AllenDang/giu"
)

var mouseButton g.MouseButton

func loop() {
    g.SingleWindow().Layout(
        g.Style().SetColor(g.StyleColorText,
            color.RGBA{0x5, 0x92, 0xc5, 255}).SetFontSize(17).To(
            g.Button("Click").Size(80, 30)),
        g.Event().OnClick(mouseButton, func() {
            fmt.Println("button clicked")
        }),
    )

}

func main() {
    wnd := g.NewMasterWindow("Button", 400, 200, 0)
    wnd.Run(loop)
}
```
