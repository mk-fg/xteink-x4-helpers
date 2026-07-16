Helper code bits for Xteink X4 eReader device
=============================================

[Xteink X4] is a cheap thin pocket-sized offline eink reader device,
great advantage of which is effectively open hardware/firmware -
runs on common ESP32 MCU, and its screen hardware is also all open or
reverse-engineered - so plenty of open-source firmwares are available for it,
unlocking all uses that its hardware is theoretically capable of.

This repository is a collection of small hacks that help with my use-cases of the device.

[Xteink X4]: https://www.xteink.com/products/xteink-x4

Table of Contents:

- [Intended use-cases here](#hdr-intended_use-cases_here)

- [Firmware patches](#hdr-firmware_patches)

- [Tools](#hdr-tools)

- [Links](#hdr-links)

Repository URLs:

- <https://github.com/mk-fg/xteink-x4-helpers>
- <https://codeberg.org/mk-fg/xteink-x4-helpers>
- <https://fraggod.net/code/git/xteink-x4-helpers>


<a name=hdr-intended_use-cases_here></a>
# Intended use-cases here

With its "standing image" eink screen, for me it's a great device for a few tasks:

- Use any kind of reminder, checklist, reference image, etc as main/sleep screen,
  same as a paper note that you can pull out of your pocket and stare at anytime
  with zero button-fiddling - always just there on screen like it's a piece paper.

- Send QR or barcode image to the device, to go pickup order or parcel which is
  handed out quickly via such codes. Similar to putting a note as a cover screen,
  but for "need to present this code/image somewhere" purpose.

- Carry a bunch of common notes, lists, references, maps, recipes, etc with me,
  with an easy way to pull those up on-screen when needed, have those stay there
  while doing whatever stuff.

    Again kinda same as paper note - no concerns for sleep mode, battery drain,
    or any kind of lighting issues - reflective eink display is great for all that.

- Reading internet or small-web marginalia - random interesting blog posts,
  think pieces, stories, explainers, wikipedia or even news articles, in a much
  more convenient form-factor than PC display, laptop or phone.

    Using [SingleFile] browser addon to store html and then convert/send it to a
    date-stamped dir on the device in one go, to read there anytime later.

- And reading books too of course, anywhere/anytime, with a great screen and
  form-factor for it.

I almost never bother carrying a phone with me, as it's a complete crap usability-wise
for many reasons (size, weight, battery, controls, bad UI/UX, worse apps, very fiddly
and fragile, etc etc), esp. for all the tasks above, which is what I need most often from
a portable-screen device, and this pocket tool is great for all that instead.

Patches/tools/stuff here is to aid/help or enable all these use-cases in some way.

Thought to make one repository for those, as there seem to be an increasing number of them.

[SingleFile]: https://www.getsinglefile.com/


<a name=hdr-firmware_patches></a>
# Firmware patches

I use [Papyrix firmware] at the moment, and [papyrix-reader-patches] dir has
usually non-upstreamable tweaks to make stuff I use the thing for more convenient,
by removing/disabling something else that I don't care about.

Each patch there should have a header at the top, describing what it does,
but they tend to be trivial enough to just read.

Can be applied via usual `patch -p1 < papyrix-reader-patches/...` command.\
Often useful to do `--dry-run` first to check whether they still apply to latest upstream.

`firmware.bin` file needs to be rebuilt and uploaded afterwards, as per upstream
instructions, but the gist there is to run `make build` to produce
`.pio/build/default/firmware.bin` and run something like `./papyrix-flasher flash ...`
with device powered-on and USB-connected. Change-rebuilds are fast after initial one.

[Papyrix firmware]: https://github.com/bigbag/papyrix-reader/
[papyrix-reader-patches]: papyrix-reader-patches


<a name=hdr-tools></a>
# Tools

To be updated after collecting various scripts in here.


<a name=hdr-links></a>
# Links

- [Xteink X4] - device product, also widely available on aliexpress and wherever.
- [Papyrix firmware] - fw that I use, initially picked for fb2 format support.
- [cbz2xtc] - tool to bundling any images into a readable "book" file for this device.

[cbz2xtc]: https://github.com/srokl/cbz2xtc
