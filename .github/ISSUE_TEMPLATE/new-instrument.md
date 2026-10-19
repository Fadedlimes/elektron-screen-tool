---
name: A machine that is not in the table
about: Send the evidence that makes an Elektron instrument supportable
title: "New instrument: "
labels: new-instrument
---

An instrument joins the table only once a real screen has been captured from it
and read, so what is needed here is a **capture**, not an id. Two steps, and the
first one is a single command.

## 1. Trace it

```
screendump --trace --midi-in <port> --midi-out <port>
```

That sends the instrument's own stock screenshot command once, using the id
bytes that already work for a Digitakt. Nothing is written to the instrument and
no firmware is touched. It prints whatever comes back, silence included, and
leaves two files:

- `screendump-trace-<time>.txt` — the report
- `screendump-trace-<time>.syx` — the frame it received, if any

If it reports no reply, try other bytes and trace again — `--probe-id 0c 00`,
for example. Both answers are useful: a machine that ignores this command is
worth knowing about, and the report records exactly what was tried.

## 2. Fill this in

- **Instrument:**
- **OS version:** (as the instrument itself reports it)
- **MIDI port name:** (the exact line from `screendump --list-midi`)
- **Did the picture match the screen?** yes / no / no reply / no picture
- **Anything unusual:** USB hub, MIDI interface, several machines plugged in

**Attach both trace files.**

## Why the port name is asked for too

It is what the window's device dropdown uses to notice that a machine is plugged
in, and every model spells it differently. A table entry that captures a screen
but cannot be auto-detected still works — it just has to be chosen by hand.

## What happens then

The capture gets read against the instrument, and if the picture matches, the
model goes into the table with those bytes and that port name. The next release
finds it automatically.
