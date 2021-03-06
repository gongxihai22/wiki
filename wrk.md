---
layout: post
title: wrk
date: 2015-06-30 15:50:00
tags:
- Atom
categories: Text Editor
---


Overview

  wrk supports executing a LuaJIT script during three distinct phases: setup,
  running, and done. Each wrk thread has an independent scripting environment
  and the setup & done phases execute in a separate environment which does
  not participate in the running phase.

  The public Lua API consists of a global table and a number of global
  functions:

  wrk = {
    scheme  = "http",
    host    = "localhost",
    port    = nil,
    method  = "GET",
    path    = "/",
    headers = {},
    body    = nil,
    thread  = <userdata>,
  }

  function wrk.format(method, path, headers, body)

    wrk.format returns a HTTP request string containing the passed parameters
    merged with values from the wrk table.

  function wrk.lookup(host, service)

    wrk.lookup returns a table containing all known addresses for the host
    and service pair. This corresponds to the POSIX getaddrinfo() function.

  function wrk.connect(addr)

    wrk.connect returns true if the address can be connected to, otherwise
    it returns false. The address must be one returned from wrk.lookup().

  The following globals are optional, and if defined must be functions:

    global setup    -- called during thread setup
    global init     -- called when the thread is starting
    global delay    -- called to get the request delay
    global request  -- called to generate the HTTP request
    global response -- called with HTTP response data
    global done     -- called with results of run

Setup

  function setup(thread)

  The setup phase begins after the target IP address has been resolved and all
  threads have been initialized but not yet started.

  setup() is called once for each thread and receives a userdata object
  representing the thread.

    thread.addr             - get or set the thread's server address
    thread:get(name)        - get the value of a global in the thread's env
    thread:set(name, value) - set the value of a global in the thread's env
    thread:stop()           - stop the thread

  Only boolean, nil, number, and string values or tables of the same may be
  transfered via get()/set() and thread:stop() can only be called while the
  thread is running.

Running

  function init(args)
  function delay()
  function request()
  function response(status, headers, body)

  The running phase begins with a single call to init(), followed by
  a call to request() and response() for each request cycle.

  The init() function receives any extra command line arguments for the
  script which must be separated from wrk arguments with "--".

  delay() returns the number of milliseconds to delay sending the next
  request.

  request() returns a string containing the HTTP request. Building a new
  request each time is expensive, when testing a high performance server
  one solution is to pre-generate all requests in init() and do a quick
  lookup in request().

  response() is called with the HTTP response status, headers, and body.
  Parsing the headers and body is expensive, so if the response global is
  nil after the call to init() wrk will ignore the headers and body.

Done

  function done(summary, latency, requests)

  The done() function receives a table containing result data, and two
  statistics objects representing the per-request latency and per-thread
  request rate. Duration and latency are microsecond values and rate is
  measured in requests per second.

  latency.min              -- minimum value seen
  latency.max              -- maximum value seen
  latency.mean             -- average value seen
  latency.stdev            -- standard deviation
  latency:percentile(99.0) -- 99th percentile value
  latency(i)               -- raw value and count

  summary = {
    duration = N,  -- run duration in microseconds
    requests = N,  -- total completed requests
    bytes    = N,  -- total bytes received
    errors   = {
      connect = N, -- total socket connection errors
      read    = N, -- total socket read errors
      write   = N, -- total socket write errors
      status  = N, -- total HTTP status codes > 399
      timeout = N  -- total request timeouts
    }
  }



下面的脚本会统计测试结果，并输出到控制台中。
wrk默认统计qps时，没有判断http status，那些错误的请求处理，比如response是500的，也被统计在qps中。很多时候，我们希望统计的qps中排除掉这些错误的请求。

```lua
done = function(summary, latency, requests)
   
    io.write("--------------------------\n")
    local durations=summary.duration / 1000000    -- 执行时间，单位是秒
    local errors=summary.errors.status            -- http status不是200，300开头的
    local requests=summary.requests               -- 总的请求数
    local valid=requests-errors                   -- 有效请求数=总请求数-error请求数


    io.write("Durations:       "..string.format("%.2f",durations).."s".."\n")
    io.write("Requests:        "..summary.requests.."\n")
    io.write("Avg RT:          "..string.format("%.2f",latency.mean / 1000).."ms".."\n")
    io.write("Max RT:          "..(latency.max / 1000).."ms".."\n")
    io.write("Min RT:          "..(latency.min / 1000).."ms".."\n")
    io.write("Error requests:  "..errors.."\n")
    io.write("Valid requests:  "..valid.."\n")
    io.write("QPS:             "..string.format("%.2f",valid / durations).."\n")
    io.write("--------------------------\n")

end

```


```bash
wrk -t4 -c600 -d30s http://127.0.0.1:8087/hello
```
