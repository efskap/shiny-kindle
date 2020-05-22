Basically I just added support for X11 root windows of depth 8 and added conversion of 32-bit rgba buffers to 8-bit grayscale ones in the preUpload step.

So the MIT-SHM stuff operates on a greyscale buffer, while clients work on a normal `*image.RGBA`

## Why

So we can run Go GUI apps on our beautiful e-ink devices without having to cross compile C!

And because GTK is ugly on the kindle anyway.

## How

So to use [aarzilli/nucular](https://github.com/aarzilli/nucular) on your Kindle Paperwhite (though it should work on any model?)

Add this to go.mod
```gomod
replace golang.org/x/exp => github.com/efskap/shiny-kindle latest
```

### Example app

Build with `GOARCH=arm GOARM=5 go build` (at least for the PW1)

```go
package main

import (
	"fmt"
	"github.com/aarzilli/nucular"
	"github.com/aarzilli/nucular/style"
	"image"
	"time"
)

var count int
var wnd nucular.MasterWindow

func main() {
	wnd = nucular.NewMasterWindowSize(0, "L:A_N:application_ID:test", image.Pt(768, 1024-24), updatefn)
	wnd.SetStyle(style.FromTheme(style.WhiteTheme, 2))
	go clear(wnd)
	wnd.Main()
}

func updatefn(w *nucular.Window) {
	w.Row(50).Dynamic(2)
	if w.ButtonText("Exit") {
		wnd.Close()
	}
	if w.ButtonText(fmt.Sprintf("increment: %d", count)) {
		count++
	}
}

// hack to flash the buffer dark to clear ghosting
func clear(w nucular.MasterWindow) {
	originalTheme := *w.Style()
	time.Sleep(1000 * time.Millisecond)
	w.SetStyle(style.FromTheme(style.DarkTheme, 0))
	w.Changed()
	time.Sleep(200 * time.Millisecond)
	w.SetStyle(&originalTheme)
	w.Changed()
}
```
