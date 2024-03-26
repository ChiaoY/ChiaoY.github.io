---
layout: post
title:  "[Solved][Jekyll] Failed to run Jekyll site locally on Windows (port error)"
date:   2024-03-26
categories: Jekyll
tags: Jekyll
---

I found this issue when testing the jekyll website on Windows envronment.

## Main Error message
```bash
C:/Ruby32-x64/lib/ruby/3.2.0/socket.rb:205:in `bind': Permission denied - bind(2) for 127.0.0.1:4000 (Errno::EACCES)
```

### Full error message
```bash
PS C:\0_Workspace\ChiaoY\ChiaoY.github.io\docs> bundle exec jekyll serve
Configuration file: C:/0_Workspace/ChiaoY/ChiaoY.github.io/docs/_config.yml
To use retry middleware with Faraday v2.0+, install `faraday-retry` gem
            Source: C:/0_Workspace/ChiaoY/ChiaoY.github.io/docs
       Destination: C:/0_Workspace/ChiaoY/ChiaoY.github.io/docs/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 0.972 seconds.
 Auto-regeneration: enabled for 'C:/0_Workspace/ChiaoY/ChiaoY.github.io/docs'
jekyll 3.9.5 | Error:  Permission denied - bind(2) for 127.0.0.1:4000
C:/Ruby32-x64/lib/ruby/3.2.0/socket.rb:205:in `bind': Permission denied - bind(2) for 127.0.0.1:4000 (Errno::EACCES)
        from C:/Ruby32-x64/lib/ruby/3.2.0/socket.rb:205:in `listen'
        from C:/Ruby32-x64/lib/ruby/3.2.0/socket.rb:768:in `block in tcp_server_sockets'
        from C:/Ruby32-x64/lib/ruby/3.2.0/socket.rb:231:in `each'
        from C:/Ruby32-x64/lib/ruby/3.2.0/socket.rb:231:in `foreach'
        from C:/Ruby32-x64/lib/ruby/3.2.0/socket.rb:766:in `tcp_server_sockets'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/webrick-1.8.1/lib/webrick/utils.rb:60:in `create_listeners'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/webrick-1.8.1/lib/webrick/server.rb:130:in `listen'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/webrick-1.8.1/lib/webrick/server.rb:111:in `initialize'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/webrick-1.8.1/lib/webrick/httpserver.rb:47:in `initialize'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/jekyll-3.9.5/lib/jekyll/commands/serve.rb:229:in `new'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/jekyll-3.9.5/lib/jekyll/commands/serve.rb:229:in `start_up_webrick'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/jekyll-3.9.5/lib/jekyll/commands/serve.rb:104:in `process'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/jekyll-3.9.5/lib/jekyll/commands/serve.rb:93:in `block in start'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/jekyll-3.9.5/lib/jekyll/commands/serve.rb:93:in `each'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/jekyll-3.9.5/lib/jekyll/commands/serve.rb:93:in `start'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/jekyll-3.9.5/lib/jekyll/commands/serve.rb:75:in `block (2 levels) in init_with_program'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/mercenary-0.3.6/lib/mercenary/command.rb:220:in `block in execute'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/mercenary-0.3.6/lib/mercenary/command.rb:220:in `each'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/mercenary-0.3.6/lib/mercenary/command.rb:220:in `execute'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/mercenary-0.3.6/lib/mercenary/program.rb:42:in `go'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/mercenary-0.3.6/lib/mercenary.rb:19:in `program'
        from C:/Ruby32-x64/lib/ruby/gems/3.2.0/gems/jekyll-3.9.5/exe/jekyll:15:in `<top (required)>'
        from C:/Ruby32-x64/bin/jekyll:25:in `load'
        from C:/Ruby32-x64/bin/jekyll:25:in `<main>'
```

## Solution

This is becuse the default `port 4000` used by jekyll is occupied by other services.
To check if there's any service using `port 4000`, we can use the following command.

```bash
PS C:\0_Workspace\ChiaoY\ChiaoY.github.io\docs> netstat -ano | find """4000"""
  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:4000           0.0.0.0:0              LISTENING       6860
```

Note: the `find` cmd is similar to `grep` in linux. However, if I don't quote the 
keyword three times, the `find` command would complain the parameter format is not correct.

So we confirmed that the process with `PID 6860` is using the `port 4000`.

We have two solution:

1. Kill `process 6860`
2. Use different port for Jekyll

### Solution 1: Kill the process 6860

1. Open the Task Manager with hotkey `Ctrl + Alt + Del`.
2. Select Task Manager.
3. In the Task Manager, right click the column header and enable the PID.
4. Clik the PID column header to sort it.
5. Found out the process or software with `PID 6860`.
6. If the process is not important, kill it.

### Solution 2: Use different port for Jekyll

Add `--port <port>` to assign the different port for jekyll, take `port 4444` for example

```
PS C:\0_Workspace\ChiaoY\ChiaoY.github.io\docs> bundle exec jekyll serve --port 4444
Configuration file: C:/0_Workspace/ChiaoY/ChiaoY.github.io/docs/_config.yml
To use retry middleware with Faraday v2.0+, install `faraday-retry` gem
            Source: C:/0_Workspace/ChiaoY/ChiaoY.github.io/docs
       Destination: C:/0_Workspace/ChiaoY/ChiaoY.github.io/docs/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 0.744 seconds.
 Auto-regeneration: enabled for 'C:/0_Workspace/ChiaoY/ChiaoY.github.io/docs'
    Server address: http://127.0.0.1:4444/
  Server running... press ctrl-c to stop.
```
