# Containerization is the new Virtualization - A short historical Overview

In the '90 Sun Microsystems coined the slogan [Write Once, Run Anywhere (WORA)](https://en.wikipedia.or/Write_once,_run_anywhere) to illustrate the crossplatform capabilities of it's - back by then new - Java platform. It never really took off at the frontend side (clients), but on the backend side (servers) it was a real revolution. Why? Because it abstracted away the real hardware of your system and instead gave you a kind of generic machine for your application. Almost all the common platform depended problems (operation system differences, versions etc.pp.) we were used to in the software development world where gone.

The next revolutionary step was the advent of virtualization. Instead of just abstracting away the hardware for a single application it abstracted it away for a complete system including the operation system itself. Just to name a few benefits:

* better utilization of hardware (infrastructure optimization)
* portability (bundle once, run anywhere)
* recovery from critical crashes within minutes
* "frozen" setups aka "immutable servers"
* sharing of complete setups is just a matter of sharing a single file
* simplified automation including system setup and various test scenarios
* separation of different applications/services into isolated VMs
* security (due to separation/isolation)
* reduced complexity (due to separation)
* ... and and and ...

Nevertheless virtualization has a few drawbacks. Beside the upfront costs of image generation (which takes quite some time) and image transfers (also time consuming), it's quite a heavyweight approach when it comes to runtime costs in terms of startup time, memory and cpu consumption.

Google, one of the largest players in the IT market, never really got into virtualization itself due to its heavyweight nature, but used containerization since years. With containerization all instances on a machine share a single kernel and the separation is done on process level. The benefits are:

* much more lightweight memory and cpu
* greatly reduced startup time (down from minutes to max. seconds)
* better utilization of hardware due to it's lightweight nature
* ... there are more due to the excellent opensource tooling which is surfacing now ...

Of course there are also some drawbacks, for example security. Process level separation will never be as secure as complete isolated VMs. It depends completely on your use case(s) which one (virtualization or containerization) is the better solution for you. But you can nevertheless mix and match as you are pleased: run a few VMs side by side with some containers, or run some containers in one or more of the VMs. There are endless possibilities ...

Actually containerization is mainly driven by opensource projects (how great is this?). The standard solution is [Docker](www.docker.com).

[In the next part](docker-up-and-running.md) I will focus on what docker is exactly, how to setup it and how to use it.
