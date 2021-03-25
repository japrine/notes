## Windows Stuff

[RDP with SOME multimonitors](https://www.hanselman.com/blog/how-to-remote-desktop-fullscreen-rdp-with-just-some-of-your-multiple-monitors)
```
List Monitor ID Numbers:
mstsc /l

Edit RDP file in notepad:
...
span monitors:i:1
use multimon:i:1
selectedmonitors:s:0,1

Add/change selectedmonitors line to list desired monitors (eg is 0 and 1)
```

