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

    - [xteink-send](#hdr-xteink-send)
    - [wifi-tmp](#hdr-wifi-tmp)
    - [xteink-xtc](#hdr-xteink-xtc)
    - [xteink-xtc-feh](#hdr-xteink-xtc-feh)

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
  with zero button-fiddling - always just there on screen like it's a piece of paper.

- Send QR or barcode image to the device, to go pickup order or parcel which is
  handed out quickly via such codes - similar to putting a note as a sleep screen,
  but for "need to present this code/image somewhere" purpose.

- Have a bunch of common notes, lists, references, maps, recipes, etc to carry
  somwhere if needed, with an easy way to pull those up on-screen and stay there,
  while doing whatever stuff.

    Again kinda same as paper note - no concerns for timeouts, backlight, battery
    drain, button fiddling - passive/reflective eink display is great for all that.

- Reading internet or small-web marginalia - random interesting blog posts,
  think pieces, stories, explainers, wikipedia or even news articles, in a much
  more convenient form-factor than PC display, laptop or phone, and eink is
  much nicer for prolonged staring at.

    Using [SingleFile] browser addon to store html and then convert/send it to a
    date-stamped dir on the device in one go, to read there anytime later.

- And reading books too of course, anywhere/anytime, with great screen and
  form-factor for it.

Almost never bother carrying a phone with me, as it's a complete crap usability-wise
for many reasons (size, weight, battery, controls, bad UI/UX, worse apps, very fiddly
and fragile, etc etc), esp. for all the tasks above, which is what I need most often from
a portable-screen device, and this pocket tool is great for all that instead.

Patches/tools/stuff here is to aid/help or enable all these use-cases in some way.\
Thought to make one repository for those, as there seem to be an increasing number of them.

[SingleFile]: https://www.getsinglefile.com/


<a name=hdr-firmware_patches></a>
# Firmware patches

I use [Inx firmware] at the moment, and [inx-patches] dir has usually
non-upstreamable tweaks to make things I care about more convenient,
often by removing/disabling something else I don't need.

Each patch there should have a header at the top, describing what it does,
but they tend to be trivial enough to just read as well.\
Can be applied via usual `patch -tNp1 < inx-patches/...` command.\
Often useful to do `--dry-run` first to check whether they still apply to latest upstream.

`firmware.bin` file needs to be rebuilt and uploaded/flashed afterwards,
as per upstream instructions, but the gist there is to do `pio run` to produce
`.pio/build/default/firmware.bin`, send that .bin over wifi (e.g. via [xteink-send]),
and run update process through "Actions" in the settings menu.\
Alternative is to use USB-C link with [fw-updater WebUI] or CLI [esptool] / [papyrix-flasher] tools.\
Or worst-case can also [put fw file for bootloader to pickup onto SD card].\
Rebuilds after any extra patches/changes are fast after that initial first build.

[Inx firmware]: https://github.com/obijuankenobiii/inx
[inx-patches]: inx-patches
[fw-updater WebUI]: https://xteink.dve.al/
[esptool]: https://github.com/crosspoint-reader/crosspoint-reader#command-line
[put fw file for bootloader to pickup onto SD card]: https://crosspointreader.com/unlock


<a name=hdr-tools></a>
# Tools

Helper scripts for terminal-centered workflow, as I tend to use zsh command-line
there as a primary/origin interface for everything - to convert formats and
simplify any routine file management operations.

<a name=hdr-xteink-send></a>
## [xteink-send]

File transfer tool, to run as `xteink-send web-longread.fb2`, which then does
everything else needed for keeping wifi connection (auto-closing it after 20min of
idleness), waiting for it to establish (if not open already), mkdir `/web/<date>`,
check/process image file(s), and run curl to upload file there, with filename sanitization
(stripping characters that firmware doesn't support, like `:`, `?`, `*`, unicode, etc),
handling request endpoint/encoding/hacks and per-file success/error checking/reporting.

Basically one run-and-done tool for easy file uploads from the command line.

Under the hood uses [wifi-tmp script] (found next to this one) via sudo
to keep wifi link running (and enable usb port for it if [hwctl] is used),
as well as common [fping]/[curl] tools, plus optionally [imagemagick]
(for sleep.bmp auto-rotation) and python [unidecode] module for better
unicode-to-ascii filename transliteration (if it's available to import).

Makes some assumptions wrt where to put files, set at the top of the script
and listed in `-h/--help` output.

[xteink-send]: xteink-send
[wifi-tmp script]: wifi-tmp
[hwctl]: https://github.com/mk-fg/hwctl
[fping]: https://fping.org/
[curl]: https://curl.se/
[imagemagick]: https://imagemagick.org/
[unidecode]: https://pypi.org/project/Unidecode/

<a name=hdr-wifi-tmp></a>
## [wifi-tmp]

Root script to run via sudo exception, to enable and keep wifi running
in a reasonably safe manner, until preset inactivity timeout expires,
used by/for [xteink-send] script above.

Running with `-h/--help` should list profiles from hardcoded wifi-tmp.profiles
file for [wpa_supplicant], and a couple other parameters, like interface name and
usb port that enables it (if [hwctl] is used), set at the top of the script.

WiFi-profile lines in `/etc/wpa_supplicant/wifi-tmp.profiles` file to connect
to AP that couple different firmwares create:

    xx4-papyrix :: 192.168.4.2/24 20m ssid="Papyrix" key_mgmt=NONE
    xx4-inx :: 192.168.4.2/24 20m ssid="Xteink-X4" key_mgmt=NONE

APs created by firmware tend to be unsecured like that, but then also has quite
a short range, so probably shouldn't be a problem in practice, but otherwise
temp-hostapd + stored connection parameters can be used instead - more PITA to
make it work (entering pw via left-right buttons), and passwordless AP is easier
to access from other devices (e.g. any laptop or phone when not on home WiFi network).

[wifi-tmp]: wifi-tmp
[wpa_supplicant]: https://w1.fi/wpa_supplicant/

<a name=hdr-xteink-xtc></a>
## [xteink-xtc]

Python script to convert image(s) to [XTC/XTCH format], supported by Xteink
device firmwares. XTC is 1-bit black-and-white, while XTCH is a nicer 2-bit
white + black + 2 shades of gray format, with both supporting pages of images,
and chapters with table of contents for those.

Format is intended for comics/manga, but is also nice for any kind of externally
pre-formatted text/documents, notes, etc, which can be exported to a set of images
in advance, dithered/tweaked to work best on grayscale eink screen as-needed,
and then bundled together into one file with this tool.

Script uses [PIL/pillow] module for loading images, and optionally [numpy]
for more efficient XTCH color processing, if it's available (can be imported).

Has options for brightness/contrast/sharpness adjustments, as well as
gray bitmap thresholds, to tweak resulting images in .xtc(h) file(s),
with `-p/--preview` option allowing to export one of the images to
BMP for easy inspection.\
See also [xteink-xtc-feh] script for trying such tweaks out with interactive
sliders and immediate/dynamic result preview.

XTCH format produced by the script is same as in [cbz2xtc] tool, with
black-dark-light-white color ordering, that seem to be incorrect according to
[CrazyCoder's gist] with format descrption (also linked above, where order is
black-light-dark-white), but this order seem to be implemented in papyrix/inx fw,
so script uses that, instead of (more difficult) patching of bit order there.
Don't know which of the two orders is supported by other firmwares, didn't check.

[xteink-xtc]: xteink-xtc
[XTC/XTCH format]: https://gist.github.com/CrazyCoder/b125f26d6987c0620058249f59f1327d
[PIL/pillow]: https://pillow.readthedocs.io/
[numpy]: https://numpy.org/
[cbz2xtc]: https://github.com/srokl/cbz2xtc
[CrazyCoder's gist]: https://gist.github.com/CrazyCoder/b125f26d6987c0620058249f59f1327d

<a name=hdr-xteink-xtc-feh></a>
## [xteink-xtc-feh]

Trivial wrapper to twiddle [xteink-xtc] image-adjustment values interactively
(using GUI [zenity] sliders) and see immediate result in [feh image viewer].

Creates zenity slider windows for each relevant parameter, combines their outputs
via [fdlinecombine] tool, and when any slider is moved, waits for half-second of
inactivity (for changes to settle) and runs [xteink-xtc] (with same CLI parameters),
creating .xtc(h) file + preview image, that's immediately displayed/refreshed by [feh]
(should monitor and auto-reload preview file on changes, as it gets updated from sliders).

Limits for zenity sliders, as well as image-viewer command
should be easy to tweak in that script itself.

[xteink-xtc-feh]: xteink-xtc-feh
[zenity]: https://gitlab.gnome.org/GNOME/zenity
[feh image viewer]: https://feh.finalrewind.org/
[fdlinecombine]: https://github.com/vi/fdlinecombine
[feh]: https://feh.finalrewind.org/


<a name=hdr-links></a>
# Links

Misc / general links:

- [Xteink X4] - device product, also widely available on aliexpress and wherever.
- [FreeInk] - ecosystem/docs about these cheap readers, and how to DIY-make one.
- [xteinkom.lowio.xyz] - large sleep-screen gallery, apparently synced from official app.

[FreeInk]: https://freeink.org/
[xteinkom.lowio.xyz]: https://xteinkom.lowio.xyz/

Firmwares - only couple ones I've used, there're many more of them around:

- [Inx] - currently using this one. Fast, stable, has nice UI.

- [Papyrix] - initially picked for fb2 format support. Janky, badly vibe-coded.

    Has nice [papyrix-flasher] command-line firmware updater tool,
    which works for other crosspoint-base firmwares as well (e.g. Inx above).

[Inx]: https://github.com/obijuankenobiii/inx
[Papyrix]: https://github.com/bigbag/papyrix-reader/
[papyrix-flasher]: https://github.com/bigbag/papyrix-flasher/

File format conversion tools:

- [cbz2xtc] - CLI tool for converting .cbz image archives into .xtc/.xtch files for this device.
- [cr2xt] - GUI tool to render common text/book files to .xtc/.xtch xteink bitmap formats.
- [dotepub.com], [epubkit.ink], [x4later.lowio.xyz] - online webpage/epub converter/optimizer tools.

[dotepub.com]: https:///dotepub.com
[epubkit.ink]: https://epubkit.ink/
[cr2xt]: https://github.com/CrazyCoder/cr2xt
[x4later.lowio.xyz]: https://x4later.lowio.xyz/
