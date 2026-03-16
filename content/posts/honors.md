+++
date = '2026-03-16T10:56:48+01:00'
draft = false 
title = 'Draft: Honoring the IRC Protocol and C-ing it through'
toc = true
+++
At the start of my master degree I decided that I wanted to do something new. The Computer Science and Engineering bachelor was fun but felt a bit unpractical. Although I learned more about how computers work and how to make great programs, at the end of the day it still felt like magic and like Gandalf I also wanted to be able to write my spells from the ground up. So I did the rational thing, and choose Embedded Systems. But the more courses I followed, the greater the gap between the hardware and software felt. So I decided to do a somewhat irrational thing... a Honors project.

## The project
To become a proper wizard, I simply felt like I mostly needed more practical experience. No more curriculum, no more constraints, just pure experience. To become a better embedded systems engineer, I decided to get as close to the problem as possible. So I decided to really, really, learn how to program an embeded system in C. Some of the courses I followed already got pretty close to my problem and even increased my curiosity further. The closest of which was Parallelization, compilers and platforms by Roel Jordans. So I asked for his supervision, which he luckily agreed with. Now I just needed a concrete-ish plan. So eventually, I came up with the following learning objectives to **become a better embedded engineer**:
- Become a better C programmer
- Produce a complete embedded product so I can also apply this to industry (design, build, eval, doc, and all)
- Stay organized (stay Agile, but not covered here?)
- ~~Destroy Isildur's Bane~~

An old goody for many new developers is to write a HTTP webserver but I had different plans... During the bachelor there was course called Computer Networks and Security by Tanir Ozcelebi. While writing a paper on QUIC, I remember reading about an old chat protocol. A little something called IRC. As a gen-z zoomer this felt like magic from a long forgotten era which I definitely needed to learn someday. And today was that day (I guess). So for my first main objective I decided to tackle IRC. And to go through the whole pipeline I decided to first develop a client for Linux and then port that onto an embedded system. Specifically, an ESP32 after discussing it viability in industry with my supervisor. Moreso, to add some interesting inputs and actuation, I decided to go with the [ESP32-2432S028R](https://www.tinytronics.nl/nl/development-boards/microcontroller-boards/met-wi-fi/jingcai-esp32-2432s028r-2.8-inch-tft-lcd-display-320*240-pixels-met-resistief-touchscreen-esp32) because added additional interesting inputs (touchscreen) and actuation (LCD display) to my project and I was inspired to potentially further develop my project into a multi medium communicaiton device somewhat like the [ESP32 Marauder project](https://github.com/justcallmekoko/ESP32Marauder) by justcallmekoko.

## Setting up the server
To develop a client for the IRC protocol it was necessary to deploy a IRC server. For IRC servers the most popular options are [UnrealIRCd](https://www.unrealircd.org/) and [InspIRCd](https://www.inspircd.org/). I tried running both but the first one I eventually ended up getting to work is the UnrealIRCd server. Still, I must say this wasn't as easy as I was expecting. Following the tutorial on their website was quite doable expect for changing the config file. However, using this Youtube video I eventually managed to set it up properly.

Given the setup experience wasn't that nice for me. I decided to make a dDocker file that could set i up for future development of my tools. This resulted in [Dockerfile](...) which I ended up writing with support of ChatGPT and the Docker docuemntation to get the sed part working correctly and use the correct terms for the install section so you don't have to rebuild all the required packages when rebulding the server.
<!--
Managed to get UnrealIRCd w
### Choosing an IRC server
 
- UnrealIRCd
- InspIRCd

UnrealIRCd

- Setup:

Honestly quite confusing. The [wiki page](https://www.unrealircd.org/docs/Installing_from_source) was good untill the configuration file part. That was really confusing. Moreover, the tutorials were quite vague. Luckily, I eventually managed to get it up and running :).

- Docker time:

Since I managed to set it up locally for testing. I now wanted to make it accessable for everybody following my GitHub repository. However, I did not manage to setup the online repository. Thus, I decided to make my own Dockerfile using the Docker knowledge i acquired from Embedded Visual Control.
-->
Still, some notable issues with the server are that when logging off, the used nickname will stay unusable for longer than expected. As such, logging off and on requires the user to wait or use a different nickname. 
## Testing the protocol
To develop a client for the IRC protocol, I first tried to familiarize myself with the general process and most important commands. i
### Learning commands
IRC actually has hundreds of commands and parameter pecularities (see [ircdocs.horse](ircdocs.horse)). Luckily, for my simple client, there were only three stages that I felt I needed to concern myself with: Authentication, channel navigation, and communication.

For authentication, I decided to use the following commands to login:
- `PASS <password>`: Submits your password to the server
- `NICK <nickname>`: Submits your nickname to the server
- `USER <username> 0 * :<realname>`: Submits your username and realname to the server

It is important that this stage happens first after connecting to the server. If not, the client will not be serviced and won't even be allowed to authenticat if the time-out is reached. Note that password is not always essential as many servers allow clients to connect without full authentication.

For navigation, I decided to restrict myself to the following three commands:
- `JOIN <channel>`: this allows you to join a channel (kind of like a groupchat with a specific name in the server)
- `LIST`: show you a list of all available channels on the server
- `PART <channel>`: removes you from the specified channel

Then finally, for communication, I only focussed on one command:
- `PRIVMSG <target> <msg>`: Sends a msg to the target ( channel or nickname) you specified.
### Testing commands
Now that I learned some important IRC commands, I needed to test them. The easiest way to experience the IRC protocol without a full client is to connect to an IRC server using `telnet` or `netcat`.

On Arch you can install `telnet` by running:
```bash
sudo pacman -Sy inetutils
```
Then you can connect to the server using:
```bash
telnet <server address> <port>
```
Then you can simply write the commands in the terminal and press enter to send the commands to the server.
However, please note that `telnet` is not practical to use for real IRC communication because it does not support TLS encryption. This allows malicious actors to read and  modify the content of any message that you or the server send if they can manage to position themselves between the client and the server.

Alternatively, you can use netcat. Installing `netcat` on Arch goes as follows:
```bash
sudo pacman -Sy openbsd-netcat
```
**Please note that the `openbsd-netcat` package has been flagged as out of date as of 08-03-2026.**
Connecting to the server using `netcat`:

```bash
nc <server address> <port>
```
You need to watch out since netcat only sends `\n` by default. As such, you need to add `\r` for some servers to function properly because the IRC protocol specifies that all lines should be ended with  a `\r\n` and old servers mainly still need this constraint to hold to parse properly.
## Relearning to program in C
<!-- _Ariel, listen to me OO languages? it's a mess. Programming in C is better than anything they've got over there_
-->
The next big step was to implement this IRC conversation. However, my knowledge of C was quite lacking. Although I had programmed in C before, for example for both Operating Systems and Parallelization Compilers and Platforms, I was still having a hard time because the environment wasn't as constrained as before. Hence, I decided to also follow the C Programming homologation course provided by TU/e while doing this project. But even after all of that I was having trouble with the socket communication. Hence, I returned to the basics, "The C programming language" by Brain W. Kernighan and Dennis M. Rithchie. Additionally, my supervisor also let me borrow "Linux Programming het complete handboek" by John Goerzen which also helped me tremoundously. Then after some tips from my supervisor on the advanced usage of man pages and after shredding my OOP biases by watching a lecture on Return Oriented Programming (can't find it right now). Return Oriented Programming (... was especially helpful), I was finally getting the hang of it.

## IRC sockets
Now that I felt like I was back in the game, I could finally properly tackle the IRC socket communication in C. Eventually, after a good amount of reading and trial and error, I ended up with the following snippet:
### Using C
```c
...
```
While working on the snippet I after found myself in a situation were the server would ignore my commands. After debugging, this generally seemed to happen because of one of the following two reasons:
1. I didn't wait enough between sending commands. This can be resolved by adding a time delay after every `write`
2. Commands were not properly parsed by the server. This could be resolved by explicitly adding a `\r\n` after each command.

Additionally, I often was rejected by the server because the nickname was already register. This happens because after registering a nickname and then leaving the server it takes a while for the server to realize the user is gone if the user doesn't explicitly notify the server.

<!--
## Making my life easier
- Struggling with C
- Learning c through the tutorial
- learning C through a course
- Learning simple electronics
-->
## Parsing responses 
<!-- add hash map joma tech reference -->
- Mapping idea
- Rejected ideation... possible perfect hashmap

## Adding proper interfaces
- Learning to use structs in a embedded environment
- Making use of static to emulate private, unspecified functions

## Setting up a testing pipeline
- cUnit integration with Make
- Writing test cases for the interfaces

## Switching to the ESP32
- Arduino examples
- Switching from Arduino to C
- Updating the build system
- Using the provided examples
- Finding a working GUI driver
- Returning to the examples 
- Fixing the backlight issues

## Making a GUI
- Following the examples
- Fixing backlight issues
- Testing the Networking
- Running into issues
- Optimizing performance

## Fixing the orientation
- Researching the craft
After building the GUI, I wanted to make the project a bit easier to use. Currently, with the my ESP in portrait mode it was quite hard to type in credentials or server addresses. Normally, the recalculations or switching to a different orientation can be quite hard since the screen, touchscreen and graphics drivers all would need to be adjusted.

## Putting it all together
**STILL WORKING ON THIS**
- Processing feedback
- Refactoring (the hash map and o.g. code combo)
- Making a server that is remotely available
- Printing cases
- Testing the product

## Wrap-up
The journey was quite long and although I lost my way a couple of times, all in all it was very fun. I want to thank my supervisor for all the advice and positivity even when I kept swearing about package incompatibility and [the existential dread resulting from not being a Fortran programmer](https://homepages.inf.ed.ac.uk/rni/papers/realprg.html). Finally, I want to thank the reader for bearing with me until the end.
