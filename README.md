# PUEO-DAQ R0.4

* firmware-pueo-turf v0r5p7
* software-pueo-turf 0.4.0
* firmware-pueo-surf6 v0r2p0
* software-pueo-surf6 0.7.0
* firmware-pueo-turfio v0r4p0

# Features

## TURF

* INI files for all daemons can be placed in ``/media`` for persistent loading.
  These INI files are copied to ``/usr/local/share`` - if they are not found
  the defaults located in the daemon directories are used instead.

* ``eCommandReset`` housekeeping command implemented to force a TURFIO
  command processor reset. Occasionally after a reboot a TURFIO will
  be stuck in a state unable to respond to register accesses
  (bridge timeout). ``eCommandReset`` will toggle the system reset.
  The TURF also does this at reboot but for some reason sometimes this
  still doesn't work.

* The complete trigger path from the SURF -> TURF should train correctly
  after fixing CIN inversion.

* Extended event parameters (``PX`` command) support added to add
  fragment delays.

### Triggers with R0.3

1. turfManualStartup
2. surfStartup --manual --enable (this SHOULD disable the data masks for the desired SURFs - script needs to be edited)
3. dev.trig.latency = 200 (not important right now but whatever - this is how long a trigger has to come in - this is probably waaay short)
4. dev.event.reset() (whatever, just be safe)
5. dev.event.mask = 0b0000 (this is a 4 bit mask to not wait on events from a given TURFIO)
6. create event server (es = EventTester.EventServer())
7. es.open() (open the event path and fill the acks)
8. dev.trig.runcmd(dev.trig.RUNCMD_RESET)  ( starts everyone running )
9. dev.trig.soft_trig()

The 'enable' in surfStartup can be skipped if you go through the
TURFIOs and set ``tio[i].dalign[j].enable=1`` for all of the SURFs you
want enabled (on TURFIO#i in slot j obviously).

if you don't do the surfStartup you'll still get data it'll just be all ramps from the TURFIOs
ONE of the TURFIOs has to be unmasked in dev.event.mask because right now the TURF uses that to generate fake headers because I don't have a real turf header generator
for any TURFIO that's masked off, the data that's output there is just garbage - it's literally random reads from the DDR memory so don't expect it to be zero

It might be fussy, watch data coming in with ``dev.event.statistics()``.
Reboot and try again as necessary, sigh.

## TURFIO

* TURF watchdog implemented. When the TURF reboots, the TURFIOs will
  reload firmware to avoid corruption due to loss of clock. This will
  also trigger the SURF watchdogs if the SURFs are running.

* caligns/daligns have IDELAYs expanded to 63 taps. You need to use
  the R0.2+ ``pueo-python`` functions.

* Data enable plumbed properly. The SURF in slot 0 has debugging
  ILAs.

* ``eAurora`` command added for lane initialization debugging.
  Aurora reset command added as a bit in ``eEnable``.

## SURF

* Multi-Tile Synchronization now performed at startup end. By default
  the SURFs will not perform this - you need to set their end state
  to 19 or higher.

* The data is now no longer complete trash because by default it selects
  the data path, not the calibration path. The cal amp is also only on
  when the calibration path is selected. Note that I literally have no
  idea if the calibration path will be total garbage or not.

* ``pyfwupd`` commanding upload path now works. You can both send files
   and execute scripts once the commanding path is established and
   running. This can be done in a broadcast mode so every SURF can be
   updated simultaneously.

* SURF state machines now advance even farther - the current final state
  is 254 (``STARTUP_COMPLETE``), however by default it only runs to 15
  (``WAIT_SYNC``). The additional states (16/17/18) are MTS related
  (``MTS_STARTUP``, ``RUN_MTS``, and ``MTS_SHUTDOWN``). ``MTS_SHUTDOWN``
  turns off significant power resources (about 0.7 W).

* SURF watchdog implemented. When the SURF has programmed its clocks
  (advanced past state ``WAIT_CLOCK``), if it detects the loss of
  ``RACKCLK`` from the TURFIO, its watchdog triggers. It then
  restarts, forcing reload of the firmware (the equivalent of
  ``bmForceReprogram``). Note that the SURF does **not** reboot
  when its watchdog is triggered - it can reload its firmware
  without rebooting, unlike the TURF.



