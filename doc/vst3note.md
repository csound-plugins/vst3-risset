# vst3note

## Abstract

Sends a single note with a specified duration to a VST3 plugin

## Description

**vst3note** sends a single note with a specified duration to a VST3
plugin. The note is translated to a MIDI Note On channel message with a
matching MIDI Note Off channel message.

The VST3 protocol supports fractional pitches, which are sent as cents of
detuning from the integer MIDI key number. In this way, any pitch can be sent
to a plugin, which may or may not be equipped to render it.

!!! note

    Be sure the instrument containing **vst3note** is not finished before the
    duration of the note, otherwise you'll have a 'hung' note.

## Syntax

```csound
i_note_id vst3note i_handle, i_midi_channel, i_midi_key, i_midi_velocity, i_duration
```
    
## Arguments

* **i_handle**: the handle that identifies the plugin, obtained from
[vst3init](vst3init.md).

* **i_midi_channel**: The zero-based MIDI channel in the interval [0, 15] of the
message.

* **i_midi_key**: The real-valued MIDI key number in the interval [0, 127] of
the note. It may have a fractional value that will be translated to a VST3
detuning parameter. Middle "C" is key number 60.

* **i_midi_velocity**: The MIDI velocity of the note in the interval [0, 127].
Mezzo-forte is velocity number 80.

* **i_duration**: The real-valued duration of the note in beats (by default, 1 beat in
Csound is 1 second).


## Output

*i_note_id* -- An identifier, unique for this instance of this plugin, of this
note, possibly useful for per-note control.


## Execution Time

* Init

## Examples


```csound

<CsoundSynthesizer>
<CsOptions>
-m195
</CsOptions>
<CsInstruments>

sr      = 48000
ksmps   = 100
nchnls  = 2
0dbfs   = 20

connect "JX10_Output", "outleft", "Master_Output", "inleft"
connect "JX10_Output", "outright", "Master_Output", "inright"
connect "Piano_Output", "outleft", "Master_Output", "inleft"
connect "Piano_Output", "outright", "Master_Output", "inright"

alwayson "JX10_Output"
alwayson "Piano_Output"
alwayson "Master_Output"

gi_vst3_handle_jx10 vst3init "mda-vst3.vst3", "mda JX10", 1
vst3info gi_vst3_handle_jx10

gi_vst3_handle_piano vst3init "mda-vst3.vst3", "mda Piano", 1
vst3info gi_vst3_handle_piano

// Array of instrument plugins indexed by instrument number, for sending
// parameter changes.

gi_plugins[] init 5
gi_plugins[3] init gi_vst3_handle_piano
gi_plugins[4] init gi_vst3_handle_jx10

// Score generating instrument.

gi_iterations init 500
gi_duration init 2
gi_time_step init .125
gi_loudness init 70
instr Score_Generator
i_time = p2
i_instrument = p4
i_c = p5
i_y = p6
i_bass = p7
i_range = p8
i_time_step = 1 / 8
i_iteration = 0
while i_iteration < gi_iterations do
    i_iteration = i_iteration + 1
    i_time = p2 + (i_iteration * gi_time_step)
    // Normalized logistic equation:
    i_y1 = i_c * i_y * (1 - i_y) * 4
    i_y = i_y1
    i_pitch = floor(i_bass + (i_y * i_range))
    event_i "i", i_instrument, i_time, gi_duration, i_pitch, gi_loudness
    prints "   %f => i %f %f %f %f %f\n", i_y, i_instrument, i_time, gi_duration, i_pitch, gi_loudness
od
prints "%-24.24s i %9.4f t %9.4f d %9.4f k %9.4f v %9.4f p %9.4f #%3d\n", nstrstr(p1), p1, p2, p3, p4, p5, p7, active(p1)
endin

instr Param_Change
i_target_plugin = p4
i_vst3_plugin init gi_plugins[p4]
k_parameter_id init p5
k_parameter_value init p6
vst3paramset i_vst3_plugin, k_parameter_id, k_parameter_value
prints "%-24.24s i %9.4f t %9.4f d %9.4f target: %3d  id: %3d  value: %9.4f #%3d\n", nstrstr(p1), p1, p2, p3, p4, p5, p6, active(p1)
endin

instr Piano
i_note_id vst3note gi_vst3_handle_piano, 0, p4, p5, p3
prints "%-24.24s i %9.4f t %9.4f d %9.4f k %9.4f v %9.4f p %9.4f #%3d\n", nstrstr(p1), p1, p2, p3, p4, p5, p7, active(p1)
endin

instr JX10
i_note_id vst3note gi_vst3_handle_jx10, 0, p4, p5, p3
prints "%-24.24s i %9.4f t %9.4f d %9.4f k %9.4f v %9.4f p %9.4f #%3d\n", nstrstr(p1), p1, p2, p3, p4, p5, p7, active(p1)
endin

instr Piano_Output
a_out_left, a_out_right vst3audio gi_vst3_handle_piano
outleta "outleft", a_out_left
outleta "outright", a_out_right
prints "%-24.24s i %9.4f t %9.4f d %9.4f k %9.4f v %9.4f p %9.4f #%3d\n", nstrstr(p1), p1, p2, p3, p4, p5, p7, active(p1)
endin

instr JX10_Output
a_out_left, a_out_right vst3audio gi_vst3_handle_jx10
outleta "outleft", a_out_left
outleta "outright", a_out_right
prints "%-24.24s i %9.4f t %9.4f d %9.4f k %9.4f v %9.4f p %9.4f #%3d\n", nstrstr(p1), p1, p2, p3, p4, p5, p7, active(p1)
endin

instr Print_Info
i_target_plugin = p4
i_vst3_plugin init gi_plugins[p4]
vst3info i_vst3_plugin
prints "%-24.24s i %9.4f t %9.4f d %9.4f k %9.4f v %9.4f p %9.4f #%3d\n", nstrstr(p1), p1, p2, p3, p4, p5, p7, active(p1)
endin

instr Save_Preset
i_target_plugin = p4
S_preset_name init p5
i_vst3_plugin init gi_plugins[p4]
vst3presetsave i_vst3_plugin, S_preset_name
prints "%-24.24s i %9.4f t %9.4f d %9.4f target: %3d  preset: %s #%3d\n", nstrstr(p1), p1, p2, p3, i_target_plugin, S_preset_name, active(p1)
endin

instr Load_Preset
i_target_plugin = p4
S_preset_name init p5
i_vst3_plugin init gi_plugins[p4]
vst3presetload i_vst3_plugin, S_preset_name
prints "%-24.24s i %9.4f t %9.4f d %9.4f target: %3d  preset: %s #%3d\n", nstrstr(p1), p1, p2, p3, i_target_plugin, S_preset_name, active(p1)
endin

instr Program_Change
i_target_plugin = p4
i_vst3_plugin init gi_plugins[p4]
p6 = 1886548852
k_parameter_id init p5
k_parameter_value init p6
vst3paramset i_vst3_plugin, k_parameter_id, k_parameter_value
prints "%-24.24s i %9.4f t %9.4f d %9.4f target: %3d  id: %3d  value: %9.4f #%3d\n", nstrstr(p1), p1, p2, p3, p4, p5, p6, active(p1)
endin

instr Master_Output
a_in_left inleta "inleft"
a_in_right inleta "inright"
outs a_in_left, a_in_right
prints "%-24.24s i %9.4f t %9.4f d %9.4f k %9.4f v %9.4f p %9.4f #%3d\n", nstrstr(p1), p1, p2, p3, p4, p5, p7, active(p1)
endin

</CsInstruments>
<CsScore>
f 0 72
i "Score_Generator" 1 1 3 .989 .5 36 60
i "Score_Generator" 2 1 4 .989 .5 78 6
; Stores original filter state...
i "Save_Preset" 1 1 4 "jx10.preset"
; Changes filter state...
i "Param_Change" 10 1 4 6 .1
i "Print_Info" 10.5 1 4
; Restores original filter state.
i "Load_Preset" 12 1 4 "jx10.preset"
i "Program_Change" 15 1 4 0 .5
i "Print_Info" 12.5 1 4
</CsScore>

</CsoundSynthesizer>
```

## See also

* [vst3audio](vst3audio.md)
* [vst3info](vst3info.md)
* [vst3init](vst3init.md)
* [vst3midi](vst3midi.md)
* [vst3note](vst3note.md)
* [vst3paramget](vst3paramget.md)
* [vst3paramset](vst3paramset.md)
* [vst3presetload](vst3presetload.md)
* [vst3presetsave](vst3presetsave.md)
* [vts3tempo](vts3tempo.md)


## Credits

Michael Gogins, http://michaelgogins.tumblr.com, michael dot gogins at gmail dot com
