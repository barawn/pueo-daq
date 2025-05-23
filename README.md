# PUEO-DAQ R0.2 Features

## TURF

* INI files for all daemons can be placed in ``/media`` for persistent loading.
  These INI files are copied to ``/usr/local/share`` - if they are not found
  the defaults located in the daemon directories are used instead.

* Event readout process works, but no global trigger exists yet,
  therefore events can only be read out from a single TURFIO by masking
  off the other TURFIOs (``dev.event.mask``) and forcing a trigger
  at the TURFIO (``tio.surfturf.write(0xC, 1)``).

* ``eCommandReset`` housekeeping command implemented to force a TURFIO
  command processor reset. Occasionally after a reboot a TURFIO will
  be stuck in a state unable to respond to register accesses
  (bridge timeout). ``eCommandReset`` will toggle the system reset.
  The TURF also does this at reboot but for some reason sometimes this
  still doesn't work.

## TURFIO

* TURF watchdog implemented. When the TURF reboots, the TURFIOs will
  reload firmware to avoid corruption due to loss of clock. This will
  also trigger the SURF watchdogs if the SURFs are running.

* caligns/daligns have IDELAYs expanded to 63 taps. You need to use
  the R0.2+ ``pueo-python`` functions.

## SURF

* ``pyfwupd`` commanding upload path now works. You can both send files
   and execute scripts once the commanding path is established and
   running. This can be done in a broadcast mode so every SURF can be
   updated simultaneously.

* SURF state machines now advance farther - the current final state
  is 15 (``WAIT_SYNC``). Error checking on this needs to be improved
  at the moment - if the software restarts once the start state has
  advanced, it will currently attempt to train on data without a training
  pattern and simply spin in a restart cycle. Resetting the DAQ system
  is best done just by rebooting the TURF at the moment.

* SURF watchdog implemented. When the SURF has programmed its clocks
  (advanced past state ``WAIT_CLOCK``), if it detects the loss of
  ``RACKCLK`` from the TURFIO, its watchdog triggers. It then
  restarts, forcing reload of the firmware (the equivalent of
  ``bmForceReprogram``). Note that the SURF does **not** reboot
  when its watchdog is triggered - it can reload its firmware
  without rebooting, unlike the TURF.

* MTS is not implemented at the moment so the individual channels
  will not be in sync.

