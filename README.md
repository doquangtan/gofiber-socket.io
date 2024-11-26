# gofiber-socket.io

- gofiber-socket.io is library an implementation of [Socket.IO](http://socket.io) in Golang, which is a realtime application framework.
- It using with web-framework [Go Fiber](https://gofiber.io)
- This library support socket.io-client version 3, 4 and only support websocket transport

## Contents

- [Install](#install)
- [Example](#example)

## Install

Install the package with:

```bash
go get github.com/doquangtan/gofiber-socket.io
```

Import it with:

```go
import "github.com/doquangtan/gofiber-socket.io"
```

and use `socketio` as the package name inside the code.

## Example

Please check more examples into folder in project for details. [Examples](https://github.com/doquangtan/gofiber-socket.io/tree/main/example)

```go
package main

import (
	socketio "github.com/doquangtan/gofiber-socket.io"
	"github.com/gofiber/fiber/v2"
)

func socketIoRoute(app fiber.Router) {
	io := socketio.New()

	io.OnAuthorization(func(params map[string]string) bool {
		// auth, ok := params["Authorization"]
		// if !ok {
		// 	return false
		// }
		return true
	})

	io.OnConnection(func(socket *socketio.Socket) {
		println("connect", socket.Nps, socket.Id)
		socket.Join("demo")
		io.To("demo").Emit("hello", socket.Id+" join us room...", "server message")

		socket.On("test", func(event *socketio.EventPayload) {
			socket.Emit("test", event.Data...)
		})

		socket.On("join-room", func(event *socketio.EventPayload) {
			if len(event.Data) > 0 && event.Data[0] != nil {
				socket.Join(event.Data[0].(string))
			}
		})

		socket.On("leave-room", func(event *socketio.EventPayload) {
			socket.Leave("demo")
			io.To("demo").Emit("hello", socket.Id+" leave us room...", "server message")
		})

		socket.On("room-emit", func(event *socketio.EventPayload) {
			socket.To("demo").Emit("hello", socket.Id, event.Data)
		})

		socket.On("disconnecting", func(event *socketio.EventPayload) {
			println("disconnecting", socket.Nps, socket.Id)
		})

		socket.On("disconnect", func(event *socketio.EventPayload) {
			println("disconnect", socket.Nps, socket.Id)
		})
	})

	io.Of("/admin").OnConnection(func(socket *socketio.Socket) {
		println("connect", socket.Nps, socket.Id)
	})

	app.Use("/", io.Middleware)
	app.Route("/socket.io", io.Server)
}

func main() {
	app := fiber.New(fiber.Config{})

	app.Route("/", socketIoRoute)

	app.Get("/test", func(c *fiber.Ctx) error {
		io := c.Locals("io").(*socketio.Io)

		io.Emit("event", map[string]interface{}{
			"Ok": 1,
		})

		io.Of("/admin").Emit("event", map[string]interface{}{
			"Ok": 1,
		})

		return c.SendStatus(200)
	})

	app.Listen(":3000")
}

```
