+++ 
date = 2023-12-26
title = "WTP - bunch of `pipewire-*` packages"
description = "ALSA, PulseAudio, JACK, PipeWire - oh my sound!"
tags = ["linux"]
series = ["what the package"]
+++

In today's run of "dnf update" a bunch of packages were udpated. But I picked `pipewire-*` packages because all I know about it is that it's the new subsystem for Linux audio. Before PipeWire it used to be ALSA and PulseAudio. And I always stayed away from it because it seemed more complicated than required. So, today I decided to learn more about PipeWire project as a part of writing this blog.

List of packages `pipewire-*` updated in today's run of `dnf update`:
* `pipewire`
* `pipewire-alsa`
* `pipewire-gstreamer`
* `pipewire-libs`
* `pipewire-pulseaudio`
* `pipewire-utils`

I was under an impression that PipeWire replaced ALSA and PulseAUdio. But this list has all three of them in it. I hope not to end up in a rabbit hole I can't return from.

### 40,000 feet overview

The description of `dnf info pipewire` says it's a multimedia server for Linux and other UNIX like OSes. On the other hand, `pipewire-utils` provides a bunch of CLI utilities to interact with the PipeWire media server, and `pipewire-libs` provides runtime libraries for applications that wish to interact with a PipeWire media server.

The package `pipewire-alsa` provides an ALSA plugin for the PipeWire media server, `pipewire-gstreamer` contains GStreamer elements to interact with the PipeWire media server, and `pipewire-pulseaudio` provides PulseAudio implementation based on PipeWire.

### 20,000 feet view

Since sound subsystem is something that's always required on a desktop OS, I was curious to see what PipeWire processes were already running:

```shell
$ ps aux | grep  pipewire
dshah       2803  0.0  0.0 326984 13408 ?        D<sl 11:15   0:00 /usr/bin/pipewire
dshah       3513  0.0  0.0 316108  9372 ?        S<sl 11:15   0:00 /usr/bin/pipewire-pulse
```

`/usr/bin/pipewire` binary is provided by the `pipewire` RPM while `pipewire-pulse` by the `pipewire-pulseaudio` RPM.

Looking into the `cgroup` file for these processes we see two corresponding services:

```shell
$ cat /proc/{2803,3513}/cgroup
0::/user.slice/user-1000.slice/user@1000.service/session.slice/pipewire.service
0::/user.slice/user-1000.slice/user@1000.service/session.slice/pipewire-pulse.service
```

From a previous experience, I remember doing `systemctl --user restart pipewire` to restart pipewire service when required. So I decided to check what units corresponding to pipewire are running on the system using:

```shell
$ systemctl --user list-units | grep pipewire
  pipewire-pulse.service                        loaded active running PipeWire PulseAudio
  pipewire.service                              loaded active running PipeWire Multimedia Service
  pipewire-pulse.socket                         loaded active running PipeWire PulseAudio
  pipewire.socket                               loaded active running PipeWire Multimedia System Sockets
```

That's two active services and sockets each that belong to pipewire.

### Rabbit hole

While I was hoping not to end up in a rabbit hole, eventually I did end up in one. It started with me doing `systemctl --user restart pipewire.service` for no reason but to make my life miserable. Since then, I couldn't bring the audio back on my system even though both the pipewire and pulseaudio servers were reporting running! However, when I stepped away from the system for a few hours (simply locking the screen to go do something else outside of the compute), the audio was back. But I wasn't in a position/mood to try and find why it got automagically fixed, and I simply shutdown the laptop.

Next day, I found this amazing video on YouTube which I very highly recommend listening to.
{{< youtube HxEXMHcwtlI >}}

I ended up learning bits I was never aware of. The audio subsystem of Linux has always made me cringe at the thought of dealing with it. It's not like I'm a pro at any of Linux's subsystems, but audio has a special place where I just don't want to go.

What I learned from the video is that ALSA, PulseAudio, Jack, PipeWire aren't mutually exclusive like I earlier thought. 

* ALSA is the kernel module that interacts with the audio hardware on our system. It replaced OSS in the kernel version 2.6. So it's always going to be there in a Linux system. 
* PulseAudio is a userspace sound server that allows multiple software access the interfaces provided by ALSA. That's how we can listen to audio playing from different software (e.g. VLC and YouTube on a browser tab) at the same time.
* JACK has lesser latency than PulseAudio which is important to folks working professionally. For day-to-day users like me, the latency of PulseAudio doesn't really matter.
* PipeWire is yet another multimedia server and, if I understood correctly, will eventually help overcome the limitation of only single software being able to access the camera at the moment, e.g., multiple software can't access a webcam simultaneously at the moment.

So my renewed understanding is that:
1. One can't remove alsa package(s) from the system.
2. PipeWire is the new userspace multimedia server that's meant to replace PulseAudio and, to an extent, JACK in future.

Out of curiosity I tried to remove packages related to PulseAudio and JACK from my system. But bluetooth related packages would get deleted along with PulseAudio, and many virtualization related packages would get removed if I were to delete `jack-audio-connetion-kit` package. Hence, I refrained from removing either.

### That's it

I initially planned to have a section on "Learning from `rpm -ql` outputs" in this post. But that caused me to procrastinate posting this by almost 10 days! Maybe I'll cover that in a different post when PipeWire pops up in my `dnf update` output once again and, more importantly, if I have the motivation for it. :wink: