---
title: 'A Haunted Story While Developing with FastAPI (feat. VSCode)'
date: '2024-10-04T23:17:03+09:00'
---

## The Experience While Developing with FastAPI

While developing a [new web project](../../project/easyplotlib) using FastAPI, I encountered a mysterious incident. (Spoiler: There were no ghosts.) For this project, I planned to use AWS for production, so I had already set up my development environment on AWS EC2. Although I had done several Python projects before, this was my first time using FastAPI, so I was trying to understand the basics with a simple [demo code](https://fastapi.tiangolo.com/ko/#_5).

![](/fastapi-spooky-story-1.png "Development screen at the time")

The environment I was developing in looked like the screen above. I installed the Remote-SSH extension in Visual Studio Code to connect to the EC2 Instance remotely. Although the screenshot is small and hard to see, you can tell that the AWS EC2 was assigned the Public IP `172.31.3.163`.

To run the server in FastAPI, you execute a command like `fastapi dev my-file.py`, and you get output like the following:

```bash
 ╭────────── FastAPI CLI - Development mode ───────────╮
 │                                                     │
 │  Serving at: http://127.0.0.1:8000                  │
 │                                                     │
 │  API docs: http://127.0.0.1:8000/docs               │
 │                                                     │
 │  Running in development mode, for production use:   │
 │                                                     │
 │  fastapi run                                        │
 │                                                     │
 ╰─────────────────────────────────────────────────────╯
```

The point to note here is that the FastAPI server is open at `http://127.0.0.1:8000`. The EC2 security group policy had already opened port 8000 at the time.

![](/fastapi-spooky-story-2.png "AWS security group setting, port 8000 is open to all IPs.")

So, I thought, since the Public IP is `172.31.3.163` and port 8000 is open, I should be able to access the test server from my browser at `http://172.31.3.163:8000`.

## The Haunted Computer

But something unexpected happened. `http://172.31.3.163:8000` timed out and could not be accessed. Wondering if the VPC settings were wrong, or if the subnet I created for the new project was misconfigured, or if the routing table was broken, I checked everything.

After checking everything and still not finding the problem, I tried accessing `http://127.0.0.1:8000` just in case...

![](/fastapi-spooky-story-3.png "Without any configuration, I can connect to the remote PC from my own computer?")

This is where things got weird. All I set up was SSH, not a VPN, so why was localhost connecting to the EC2 instance? I started Googling with all sorts of thoughts in mind, but there were so many related topics that even coming up with search keywords was difficult. Summing up the situation at the time:

* AWS EC2
* Connected via SSH
* FastAPI Dev Server
* Cannot connect via Public IP
* Can connect via localhost

Because there were so many dependencies, I couldn't even come up with proper search keywords. At this point, learning FastAPI became a lower priority, and I just wanted to know why this was happening right now. To solve this, I hypothesized that the root cause of this strange phenomenon would be in one of AWS, SSH, or FastAPI, and decided to verify each.

## Hypotheses About the Cause

**Is it AWS?**  
I checked all the AWS settings from scratch, but there was nothing wrong. If AWS was the problem, it would be hard to explain why I could connect via localhost but not at all otherwise.

**Is it SSH?**  
SSH seemed more likely to be the problem. After all, it's the direct bridge between Remote and Local. But I couldn't figure out how the connection could "leak" outside SSH, so I thought it might be related to FastAPI.

**Is it FastAPI?**  
I thought there might be some feature in FastAPI itself. My hypothesis was that when running `fastapi dev my-file.py`, FastAPI might detect if it's running in an SSH environment and automatically establish some hidden connection.

**Is it UVicorn?**  
While looking through the FastAPI documentation, I found that FastAPI's server implementation relies on the open-source [UVicorn](https://www.uvicorn.org/). I searched for keywords like Uvicorn SSH, but found no mention of any special feature that only activates in SSH environments.

## An Unexpected Clue

Up to this point, there was no sign of a solution. Then, by chance, I found a clue. While searching, I tried connecting to SSH using Git Bash instead of Visual Studio Code and ran `fastapi dev my-file.py`. This time, I could no longer access the web page via `localhost:8000`. Feeling something was off, I reconnected with Visual Studio Code, and this time it worked. Using this as a clue, I Googled "vscode ssh remote localhost" and at this point, the cause became clear.

## The Cause of the Problem - The Answer

I found the answer in a [blog post](https://caniro.tistory.com/292) by someone who had the same problem in 2021. When you connect via SSH in VSCode, you don't just connect; you use an extension called [Remote-SSH](https://code.visualstudio.com/docs/remote/ssh). This plugin has a built-in feature called Port Forwarding, which binds the remote server's port to your local machine's port. In fact, basic SSH also has this feature, but the key difference is that Remote-SSH automatically detects ports and performs port forwarding for you. Because I use SSH so often, I had heard that such a feature existed. But I had never needed to use it before, so I forgot about it, and above all, I didn't know that VSCode would automatically do port forwarding.

According to [a comment by a Microsoft developer](https://github.com/microsoft/vscode/issues/143958), the extension automatically detects open ports in two ways, depending on settings:

* Detects processes with open ports among currently running processes
* Detects terminal output containing a URL + port format

After learning this, I checked VSCode again and found the following tab.

![](/fastapi-spooky-story-4.png "Remote port 8000 is being forwarded to localhost:8000.")

Actually, this problem was a combination of two issues. Remember, not only was it strange that I could connect via localhost, but I also couldn't connect via Public IP? To allow connections via Public IP in the FastAPI dev server, you actually need to add the following option:

```bash
fastapi dev my-file.py --host 0.0.0.0
```

If you run it with this option (assuming your security settings are correct), you can also access it via `http://172.31.3.163:8000` using the Public IP. ■