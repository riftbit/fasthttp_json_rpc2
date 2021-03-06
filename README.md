# jrpc2server (early beta)
[Website](https://www.riftbit.com) | [Contributing](https://www.riftbit.com/How-to-Contribute)

[![license](https://img.shields.io/github/license/riftbit/jrpc2server.svg)](LICENSE)
[![GoDoc](http://img.shields.io/badge/go-documentation-blue.svg?style=flat-square)](https://godoc.org/github.com/riftbit/jrpc2server)
[![Coverage Status](https://coveralls.io/repos/github/riftbit/jrpc2server/badge.svg?branch=master)](https://coveralls.io/github/riftbit/jrpc2server?branch=master)
[![Build Status](https://travis-ci.org/riftbit/jrpc2server.svg?branch=master)](https://travis-ci.org/riftbit/jrpc2server)
[![Go Report Card](https://goreportcard.com/badge/github.com/riftbit/jrpc2server)](https://goreportcard.com/report/github.com/riftbit/jrpc2server)

Golang package for easy creation JSON-RPC 2.0 API services.

## System requirements 
- github.com/valyala/fasthttp
- github.com/riftbit/jrpc2errors
- tested on golang 1.10.1+

## Examples

### Benchmark results

Tested on work PC (4 cores, 16gb memory, windows 10)

Used memory on all benchmark time: 7.2 MB

Used CPU on all benchmark time: 12%

Load soft used on same PC: SuperBenchmark (sb)

![Benchmark Results](rps_results.png?raw=true "Benchmark Results")

### Without routing

```golang
package main

import (
	"github.com/valyala/fasthttp"
	"github.com/riftbit/jrpc2server"
	"log"
	"runtime"
	"runtime/debug"
)

//Log area
type DemoAPI struct{}

//Add Method to add Log
func (h *DemoAPI) Test(ctx *fasthttp.RequestCtx, args *struct{ID string}, reply *struct{LogID string}) error {
	//log.Println(args)
	reply.LogID = args.ID
	return nil
}

func main() {
	api := jrpc2server.NewServer()
	err := api.RegisterService(new(DemoAPI), "demo")

	if err != nil {
		log.Fatalln(err)
	}

	fasthttp.ListenAndServe(":8081", api.APIHandler)
}
```


### With minimal routing

```golang
package main

import (
	"github.com/valyala/fasthttp"
	"github.com/riftbit/jrpc2server"
	"log"
	"runtime"
	"runtime/debug"
)

//Log area
type DemoAPI struct{}

//Add Method to add Log
func (h *DemoAPI) Test(ctx *fasthttp.RequestCtx, args *struct{ID string}, reply *struct{LogID string}) error {
	//log.Println(args)
	reply.LogID = args.ID
	return nil
}

func main() {
	api := jrpc2server.NewServer()
	err := api.RegisterService(new(DemoAPI), "demo")

	if err != nil {
		log.Fatalln(err)
	}

	reqHandler := func(ctx *fasthttp.RequestCtx) {
		switch string(ctx.Path()) {
		case "/api":
			api.APIHandler(ctx)
		default:
			ctx.Error("Unsupported path", fasthttp.StatusNotFound)
		}
	}

	fasthttp.ListenAndServe(":8081", reqHandler)
}
```

### With advanced routing and middlewares (not tested, will be updated)

```golang
package main

import (
	"github.com/valyala/fasthttp"
	"github.com/riftbit/jrpc2server"
	"github.com/thehowl/fasthttprouter"
	"log"
	"runtime"
	"runtime/debug"
)

//Log area
type DemoAPI struct{}

//Add Method to add Log
func (h *DemoAPI) Test(ctx *fasthttp.RequestCtx, args *struct{ID string}, reply *struct{LogID string}) error {
	//log.Println(args)
	reply.LogID = args.ID
	return nil
}


func main() {
	api := jrpc2server.NewServer()
	err := api.RegisterService(new(DemoAPI), "demo")
	if err != nil {
		log.Fatalln(err)
	}

	router := fasthttprouter.New()
    router.POST("/", mainHandler())

	server := fasthttp.Server{
		Name:    "AWESOME SERVER BY RIFTBIT.COM",
		Handler: router.Handler,
	}

	Logger.Println("Started fasthttp server on port", config.System.ListenOn)
	Logger.Fatal(server.ListenAndServe(config.System.ListenOn))
}


func mainHandler() fasthttp.RequestHandler {
	return fasthttp.RequestHandler(func(ctx *fasthttp.RequestCtx) {
		stats := statsData{EventTime: time.Now()}

        //Here you can run any handlers you need
		API.APIHandler(ctx)

		ctx.Response.Header.Set("Server", ServerName)
		ctx.Response.Header.Set("X-Powered-By", PoweredBy+build)

		stats.UsedTime = time.Since(stats.EventTime)
		stats.ClientIP = ctx.RemoteIP().String()

		ctx.Request.Header.VisitAll(func(key, value []byte) {
			if string(key) == "X-Forwarded-For" {
				stats.ClientIP = string(value)
			}
		})

		stats.StatusCode = ctx.Response.StatusCode()
		stats.Host = ctx.Request.Host()
		stats.UserAgent = ctx.Request.Header.UserAgent()

		stats.BodyReq = ctx.Request.Body()
		stats.BodyResp = ctx.Response.Body()

        /*
        do something with stats :)
        *./

		return
	})
}
```
