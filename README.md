*A simple scene implemented with Three.js several years ago during Computer Graphics course in university following J. Dirksen book "Learning Three.js". Uses old version of Three.js and other libs, I don't know if it is compatible with modern versions.*

It is the castle from [Island Castle](https://github.com/AlexP11223/Three.js_IslandCastle) but with textures and some guards: archers on the walls and knights patrolling around. It is also possible to add new knights, they will walk through the gate (not forgetting to open it) and will start patrolling too.

![](https://i.imgur.com/IOp6pNa.png)

![](https://i.imgur.com/WA14kGQ.png)

![](https://i.imgur.com/ZLCJ63T.png)

Requires HTTP server to serve the files, cannot be run by just opening local HTML file (without changing security settings in most web browsers). At the time I was using [Mongose](https://cesanta.com/binary.html) web server suggested in the book: it is free for personal use and very easy to use (on Windows just put .exe in the directory with files, run the .exe and it will open a web page).

Can be run here: [https://alexp11223.github.io/Three.js_GuardedCastle/index.html](https://alexp11223.github.io/Three.js_GuardedCastle/index.html)

# Implementation details

## World
