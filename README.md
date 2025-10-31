# MIDI Format Version 3.0 Specifications

I invented a new midi format: MIDI Format Version 3.0! 

Compared to the meaningless Format Version 2.0 that does something that Steinberg and Image-Line have already accomplished using MIDI 1.0 in combination with their own private protocol more than 20 years ago, Format Version 3.0 has actual use cases that no one has ever implemented a direct replacement for. 

Format Version 3.0 is not made for editing; it's made for real time playback. It makes midi parsing and loading 100x faster for all midi players that implemented support for Format Version 3.0! Midi players that have not yet implemented support for Format Version 3.0 can also quickly add support for this format in just a few extra lines of code. 

What made MIDI 3.0 so fast and so easy to implement? 

First, all midi events in Format Version 3.0 always uses absolute timing instead of relative timing. Second, all midi events in Format Version 3.0 has an additional field that holds the absolute microsecond for this event. Third, all midi tracks are merged into one with each individual event having an extra field indicating which track it's coming from. Finally, all note on events have an extra field labeling the index of its matching note off, all note off events also have an extra field labeling the index of its matching note on. 

I investigated the source code of many popular open-sourced midi players and found that the first thing that the midi player does after loading a midi file is to convert relative tick to absolute tick. Then, many of them would merge all midi tracks into a single big event pool sorted by time. Many midi players that have a visualizer like to connect note on and note off so they can render graphics efficiently. 

This is the routine that 90% of the midi players do every single time they load a new midi file for playback. 

So, I wondered what if we do all of that work ahead of time? And that gave me the idea of creating MIDI 3.0. 

MIDI 3.0 uses the exact same MThd header format as MIDI 1.0, only with the format version field set to 3. Followed by a single MTrk section containing ALL midi events instead of splitting into multiple MTrk sections like Format Version 1.0. All events have 3 extra fields, unsigned long long abs_microsecond, unsigned short track_id, and size_t pair_index. pair_index is set to ~0 for all non-note events. 

MIDI 3.0 is backward compatible with MIDI 1.0, and a lossless 1.0 to 3.0 and 3.0 to 1.0 conversion is theoretically doable. However, MIDI 3.0 is NOT compatible with MIDI 2.0. 

MIDI 3.0 can only be played by midi players that support this format, you cannot open Format Version 3.0 midi file in a midi player that only knows Format Version1.0. 

# MIDI Format Version 4.0 Specifications

For midi editors, I have another new midi format made specifically for this purpose: MIDI Format Version 4.0! 

MIDI 4.0 shares the same header format as MIDI 1.0, only with the format version field set to 4. It still preserves the multiple separated MTrk layout like MIDI 1.0, because this track separation, although not friendly to midi players, is very useful for midi editors. The real major difference is here: 

Note events are no longer identified as separated note on and note off events. Instead, they are combined into one single note on event with a specific duration. 
And the same as MIDI 3.0, all events use absolute tick timing, but there's no absolute microsecond field in MIDI 4.0. Microsecond is what midi players would care about, not midi editors; editors only care about tick. 

There's now a new type of event in Format Version 4.0 called "continuous event". Instead of having one instantaneous event on every single tick setting the value of a CC controller or song tempo, it's now replaced by one single continuous event that defines the start time, duration, event type, channel, start value, end value, and easing function. The old instantaneous CC and tempo event are still supported in 4.0 for backward compatibility. 

Be aware that there WILL be permanent data loss when converting 4.0 to 1.0 or 3.0! All continuous midi event will be sliced into multiple instantaneous events after conversion because MIDI 3.0 and MIDI 1.0 doesn't support continuous events. 

The reason why 3.0 did not use combined note on and note off is because although this is super useful for midi editors, it will over complicate things for midi players. This also holds true for why not implement continuous midi event support in 3.0, adding continuous event support in 3.0 will only make midi player's code more complicated with little actual help in audio quality or execution speed. 

The relationship between Format Version 4.0 and Format Version 3.0 is the same as .java VS .class. One is made to be readable; the other is made for fast execution. 

Wait, isn't Format Version 4.0 doing something that DAWs already solve using their private .als or .flp format? Yes, but only partially. What if you want to transfer data from one DAW to another? 

Without a standardized MIDI 4.0, you're forced to lose all your beautiful automation envelopes and suffer for the extradentary long export and import time. 

However, if both of the DAWs you're using happened to implement support for MIDI Format Version 4.0, then you can quickly export your project in literal MILLE SECONDS and import it into the other DAW with all your automation envelops perfectly preserved! 

The biggest problem I encountered when trying to incorporate MIDI 2.0 into my workflow was the fact that 99% of the software I use doesn't support 2.0 at all. 

Even if my DAW supports 2.0, my midi players don't. 

Even if my midi player supports 2.0, my synths don't. 

The beauty of MIDI 3.0 and MIDI 4.0 is that they are all fully compatible with the existing MIDI 1.0 format. 

The synth can stay completely unchanged, it only needs to support MIDI Format 0 and that's it. 

Only the midi player needs a very minor few lines of extra code to support Format Version 3.0, and then they can easily convert it to Format 0 on the fly by simply omitting the track_id field and send it to the synth. 

The same holds true for midi editor as well. With only a tiny bit of very minor code change, they can add support for Format 4, and all their current editing and playback code can stay completely untouched. 

# Implementation Examples

Work in progress... 
