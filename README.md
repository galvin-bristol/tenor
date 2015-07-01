# Tenor

The concept of music is incredibly accommodating; anything from industrial noise to baroque classical can be appreciated with enough interest. This makes the problem statement simple - all we need is a random rhythm and melody generator and we could pass off its output as music.

But popular and contemporary music has tuned the ears of the average listener to certain preset patterns in melody, harmony and rhythm. As a result, most of us have developed a collective sense of what makes good music and what doesn't, while in fact, there is no such thing as good or bad music (objectively, of course). Now this makes the problem statement a lot more tangible - generating music is easy, generating music that *sounds good to the human ear* requires more effort.

The project is an attempt at constructing primitive music theory and using it as a base to build procedures that can generate melodies, in-tune to the ears of the average listener. Most of the heavy lifting is done by [Clojure](http://clojure.org/) and [Overtone](http://overtone.github.io/); we just build simple abstractions to represent concepts and patterns in music theory and use them to generate musical pieces.

***

## Contents

- [Music to the human ears](#music-to-the-human-ears)
- [Rhythm](#rhythm)
  - [Time Signature](#time-signature)
  - [Popular songs in non-4/4 signature](#popular-songs-in-non-44-signature)
  - [Representing measures](#representing-measures)
  - [Meters](#meters)
  - [Generating a measure in 11/4](#generating-a-measure-in-114)
  - [Sparseness of a measure](#sparseness-of-a-measure)
  - [TL;DR](#tldr)
- [Melody](#melody)
  - [Notes](#notes)
  - [Octaves](#octaves)
  - [Semitones and Whole tones](#semitones-and-whole-tones)
  - [Scales](#scales)
  - [Intervals - unison, steps and leaps](#intervals-unison-steps-and-leaps)
  - [Melodic motion - conjunct and disjunct](#melodic-motion-conjunct-and-disjunct)

***

### Music to the human ears

Most pieces in contemporary music share certain common characteristics in rhythm, harmony and melody. There's a high chance that one or more of these characteristics occur in a song that we happen to discover, and since we're attuned to these patterns already, the melody then becomes instantly identifiable without sounding like random noise.

Some of these idiosyncrasies could be:

- The famed [4/4 time signature](https://en.wikipedia.org/wiki/Time_signature#Most_frequent_time_signatures).
- The [call-and-response](https://en.wikipedia.org/wiki/Call_and_response_%28music%29#Popular_music) pattern.
- Phil Specter's [Wall of Sound](https://en.wikipedia.org/wiki/Wall_of_Sound).
- The [C-major scale](https://en.wikipedia.org/wiki/C_major).
- Variations of the C-D-G-C chord pattern or other common chord patterns.

Here's a [very interesting analysis](http://www.hooktheory.com/blog/i-analyzed-the-chords-of-1300-popular-songs-for-patterns-this-is-what-i-found/) on common patterns found in over 1300 popular songs. One way of generating listenable music would be to create procedures that conform to some of these (though not exactly these) norms.

***

### Rhythm

Rhythm is, perhaps, the most accessible and instantly decipherable component of a song because it lays out the fundamental structure over which melody and harmony interplay. A single rhythmic theme (say, the 4/4) also repeats fairly often in a song, so each individual measure in a musical piece need not be procedurally generated.

##### Time Signature

A [time signature](https://en.wikipedia.org/wiki/Time_signature) governs how many beats (or foot-taps) there are in a measure, and the duration of each beat. A signature in 4/4 simply means that we repeatedly tap 4 times in quarter-note (1/4 - its length depends on the length of the full note) intervals. Here, each tap is a beat and 4 taps make a measure (the repeating basic pattern of the song). Here's how we'd tap our feet to *Comfortably Numb* by Pink Floyd (a popular 4/4 song), for instance:

```
Hello Hel - lo Hello - (Rest) Is there - anybody 
Tap       - Tap      - Tap             - Tap

In there? - (Rest) - (Rest) Just - nod
Tap       - Tap    - Tap         - Tap
```

Let's take the first beat ('Hello Hel'), and decompose the quarter note (1/4) into sixteenth notes (1/16), which means we're gonna split the beat further into four other beats. Now, if we tap to *these* beats (we'd have to tap really fast), the quarter beat will look like this.

```
Hel - lo  - (rest)  - Hel
Tap - Tap - Tap     - Tap
```

This tells us that in a measure decomposed into sixteenth beats (assuming we do not cross over into 1/32 and further), notes and rests are, to an extent, randomly interspersed. To reaffirm this, we can look at *Smoke on the Water* by Deep Purple (another popular 4/4 song). Let's take the first measure (remember, 4 foot taps, each of quarter-note length):

```
Smoooo - ke on the - Waaate - rrrrrrr
Tap    - Tap       - Tap    - Tap
```

First beat, sixteenth notes:

```
Smo - oo  - oo  - oo
Tap - Tap - Tap - Tap
```

If 'Hello Hel' was `1-1-0-1` (note-note-rest-note), then 'Smoooo..' is `1-1-1-1` because there are no rests. We can observe the `1-1-1-1` decomposition for the first beat in other songs like *Eleanor Rigby* by The Beatles (just the dragged out 'I' syllable).

##### Popular songs in non-4/4 signature

*Happy Birthday* is in 3/4 time. Let's do the tap routine for the first four measures:

```
(Rest) - (Rest) - Happy
Tap    - Tap    - Tap

birth  - day    - to
Tap    - Tap    - Tap

you    - (Rest) - Happy
Tap    - Tap    - Tap

birth  - day    - to
Tap    - Tap    - Tap
```

*Money* by Pink Floyd is in 7/4 time.

```
Money - (Rest) - (Rest) - (Rest) - (Rest) - (Rest) - (Rest)
Tap   - Tap    - Tap    - Tap    - Tap    - Tap    - Tap

Get A - way    - (Rest) - (Rest) - (Rest) - (Rest) - (Rest)
Tap   - Tap    - Tap    - Tap    - Tap    - Tap    - Tap

Get a - good   - job    - with   - more   - pay'n  - you're
Tap   - Tap    - Tap    - Tap    - Tap    - Tap    - Tap

Ooo   - ohkaa  - aay    - (Rest) - (Rest) - (Rest) - (Rest)
Tap   - Tap    - Tap    - Tap    - Tap    - Tap    - Tap
```

*Four Sticks* by Led Zeppelin uses 5/4 in parts, *Paranoid Android* by Radiohead uses 7/8 in parts, the *Mission Impossible theme* by Lalo Schifrin uses 5/4.

##### Representing measures

Sixteenth beats give us enough granularity to represent positions in the measure where notes are placed. A 4/4 measure, decomposed into sixteenth beats, would look like this in positional notation:

```
(1-2-3-4)  -  (5-6-7-8)  -  (9-10-11-12) -  (13-14-15-16)
1/4 Beat 1 -  1/4 Beat 2 -  1/4 Beat 3   -  1/4 Beat 4
```

Any of these numbers from 1 to 16 could be notes or rests. For the sake of representation, let's ignore the rests (there can only be notes and rests, so if a number is missing, we know it's a rest). Let's go back to the *Comfortably Numb* example, whose first beat we decomposed as `1-1-0-1`. In our new notation of a measure, this would be `1-2-4` (3 is a rest, so it's ignored). Decomposing the other beats of the measure in the old beat-wise and new positional notations, we get:

```
Beat-wise                    |    Positional
-----------------------------|---------------------------------
                             |
1   -  1    -  0  -  1       |    1   -   2   -   4
Hel -  lo   -  _  -  Hel     |    Hel -   lo  -   Hel
                             |
1   -  0    -  1  -  1       |    5   -   7   -   8
lo  -  _    - Hel -  lo      |    lo  -   Hel -   lo
                             |
0   -  0    -  1  -  1       |    11  -  12
_   -  _    -  Is -  there   |    Is  -  there
                             |
1   -  1    -  1  -  1       |    13  - 14   - 15 - 16
any -  body -  in -  there?  |    any - body - in - there?
```

Ignoring the rests, this becomes `(1-2-4) - (5-7-8) - (11-12) - (13-14-15-16)` in our new positional notation. Let's put these together into a list:

```
[1 2 4 5 7 8 11 12 13 14 15 16]
```

We just represented the first rhythmic measure of *Comfortably Numb* in an ordered list. Similarly for *Smoke on the Water*, this is:

```
[1 2 3 4 7 8 9 10 11 12 13 14 15 16]
```

As we can see, only the 5th and 6th sixteenth-beats (immediately after 'Smooooke' and before 'on the') are rests, so the first measure of *Smoke on the Water* turns out to be densely populated with notes.

##### Meters

So far, we only looked at a measure as having beats of the same type (a `4/4` measure had 4 quarter-beats), but this is not an overarching rule for constructing measures.

A [meter](https://en.wikipedia.org/wiki/Meter_%28music%29) tells us how to *count* or *accentuate* beats in a measure. For instance, a measure in 11/4 time signature could be split into `2-2-3-2-2` or `4-3-2-2`, a measure in 4/4 time signature could be split into `2-2`, `3-1`, or so on. These are still comprised of quarter-beats, but the meter simply groups beats together to emphasize stress on certain groups of beats and de-emphasize stress on others. A measure in 11/4 can be counted in any of the following ways:

```
 one-two - one-two - one-two-three - one-two - one-two
 one-two-three-four - one-two-three - one-two - one-two
 one-two-three-four - one-two-three-four - one-two-three
```
 
The grouping could simply mean that the *one* beats are more emphasized (preferably using notes), than the others (could contain notes or rests).

For example, the song *Flower Punk* by Frank Zappa, a song with interchanging 5/4 and 7/4 measures, can be counted using the following meter:

```
one-two - one-two-three
one-two - one-two - one-two-three
one-two - one-two-three
one-two - one-two - one-two-three
```

We can count the *Mission Impossible* theme using the following meter:

```
one-two-three - one-two-three - one-two - one-two
```

##### Generating a measure in 11/4

So far, we've looked at decomposing a measure into a meter and further into a list of sixteenth-beats. The meter helps in singling out sixteenth-beat positions within the measure which *must* contain notes for emphasis, and each of the other sixteenth-beat positions could either contain a note or a rest.

The [generate-meter](https://github.com/pranavrc/tenor/blob/master/src/tenor/constructs.clj#L21) function takes a beat count for a measure, and generates a random meter. Here are example meters for the 11/4 and 4/4 time signatures:

```
user=> (generate-meter 11)
(1 4 3 3)
```

This is the `one - one-two-three-four - one-two-three - one-two-three` counting pattern where all the ones *must* contain notes and not rests.

```
user=> (generate-meter 4)
(2 2)
```

The `one-two - one-two` meter with ones containing notes.

Now that we have the meter, we can further decompose these into sixteenth-beats using the [segment-measure](https://github.com/pranavrc/tenor/blob/master/src/tenor/constructs.clj#L59) function. A 11/4 measure would contain `11*16/4 = 44` sixteenth-beats.

```
user=> (def meter-11 (generate-meter 11))
#'user/meter-11
user=> meter-11
(1 4 3 3)
user=> (segment-measure meter-11 :note-value 4)
(1 3 4 5 7 8 9 10 11 13 14 16 19 20 21 23 28 30 32 33 37 39 42 44)
user=> (segment-measure meter-11 :note-value 4)
(1 2 4 5 6 8 9 11 12 15 17 19 21 22 23 25 26 28 29 31 32 33 36 37 38 39 40 42 43)
user=> (segment-measure meter-11 :note-value 4)
(1 2 3 4 5 7 8 10 11 12 13 16 17 18 19 21 23 24 26 30 32 33 34 35 36 38 39 40 41 43 44)
```

Note that for our `1-4-3-3` (`4-16-12-12` in sixteenth-beats) meter (the *ones* would correspond with 1, 5, 21 and 33 sixteenth-beat positions respectively), every generated measure contains a note at the 1, 5, 21 and 33 note positions. A rest can never fall on these positions.

##### Sparseness of a measure

We introduce a new factor called *sparseness* which determines how dense a measure is (Remember? The first measure of *Smoke on the Water* had more notes than *Comfortably Numb*. Ergo, it was less sparse). The higher the *sparseness* of a measure, the more rests it contains. The default sparseness is 1, which means a sixteenth-beat has equal chances of being either a note or a rest.

```
user=> (segment-measure meter-11 :note-value 4 :sparseness 2)
(1 2 4 5 8 10 13 15 21 23 27 29 30 33 35 41 42 44)
```

Let's turn up the sparseness to an absurdly high number, say 100:

```
user=> (segment-measure meter-11 :note-value 4 :sparseness 100)
(1 5 21 33)
```

Voila, we have the *ones* from the meter! The other sixteenth-beats were never filled, they all ended up as rests.

##### TL;DR

- We construct a random *meter* for a single measure of a specific time signature using the `generate-meter` function. Say, `(1 4 3 3)` for a 11/4 measure (11 quarter-beats, counted `one - one-two-three-four - one-two-three - one-two-three`).
```
user=> (def meter-11 (generate-meter 11))
#'user/meter-11
user=> meter-11
(1 4 3 3)
```

- We segment the measure further into sixteenth-beats, using the `segment-measure` function. A 11/4 measure will have `11 * 16/4 = 44` sixteenth beats, and since the meter specifies positions that *must* contain notes, our measure will contain notes at `1`, `1 + (16/4)*1 = 5`, `5 + (16/4)*4 = 21` and `21 + (16/4) * 3 = 33` sixteenth-beat positions. The rest of the beats are populated randomly with notes or rests.
```
user=> (segment-measure meter-11 :note-value 4)
(1 3 4 5 7 8 9 10 11 13 14 16 19 20 21 23 28 30 32 33 37 39 42 44)
```

- The ratio of notes to rests in the measure can be determined by using the *sparseness* parameter (whose default value is 1).
```
user=> (segment-measure meter-11 :note-value 4 :sparseness 2)
(1 2 4 5 8 10 13 15 21 23 27 29 30 33 35 41 42 44)
user=> (segment-measure meter-11 :note-value 4 :sparseness 100)
(1 5 21 33)
```

***

### Melody

Melody is, possibly, the identity of a song. It can be considered the prime contributing factor to the *mood* or *feel* of a song. Though a melody cannot exist without a rhythm, both rhythm and harmony can be considered as support structures that enrich the melody. There are probably a huge number of patterns and methods in which listenable melodies can be generated procedurally.

##### Notes

[Notes](https://en.wikipedia.org/wiki/Musical_note) are sounds of a certain pitch or frequency. The A4 note is, for instance, a sound of 440 Hz. The A5 note is a sound of 880 Hz. The musical note starts from C0 at 16.35 Hz. Unlike the English alphabet, musical notes in western musical notation start from C and end with G, as follows:

```
C  C#/Db  D  D#/Eb  E  F  F#/Gb  G  G#/Ab  A  A#/Bb  B
```

`#` indicates a *Sharp* and `b` indicates a *Flat*. Sharps, or flats, are indicated by the black keys on the piano. Ever wondered why there is no black key between `B#/Cb` and `E#/Fb`? There *is* no `B#/Cb` or `E#/Fb`. If you sharp a B, you get a C and if you sharp an E you get an F. This is merely a convention that arose from the constraint that there could only be 12 notes in an *octave*. In some forms of musical notation, E# and B# do exist.

The frequency ratio between a note and its previous note is the [twelfth root of two](https://en.wikipedia.org/wiki/Twelfth_root_of_two), or `1.0594`. For instance, `C#0 (17.32 Hz) / C0 (16.35 Hz) = 1.0594`.

##### Octaves

An [octave](https://en.wikipedia.org/wiki/Octave) is the distance between a note of frequency X and the note of frequency 2X, containing twelve notes. The range of frequencies between A4 and A5 constitute one octave.

For instance, here's the octave from A4 to A5 and the corresponding frequencies:

```
A4   A#4/Bb4  B4      C5      C#5/Db5   D5      D#5/Eb5   E5      F5      F#5/Gb5   G5      G#5/Ab5
440  466.16   498.88  523.25  554.37    587.33  622.25    659.25  698.45  739.99    783.99  830.61
```

##### Semitones and Whole tones

- The interval between one note and the next (A4 and A#4) is a [semitone](https://en.wikipedia.org/wiki/Semitone) or a half-step. They're indicated by `H`.
- The interval between a note and the note two semitones after it (A4 and B4) is a [whole tone](https://en.wikipedia.org/wiki/Major_second) or a whole-step. They're indicated by `W`.

##### Scales

[Scales](https://en.wikipedia.org/wiki/Scale_%28music%29) are groups of notes, ascending in frequency. Depending on how a scale is built, it could exhibit a certain mood, characteristic or emotion. The most used types of scales are the major scales and the minor scales, but there are a lot of other types.

The first note of a scale (the note after which the scale) is named, is called the *tonic* of the scale.

[Major scales](https://en.wikipedia.org/wiki/Major_scale) are constructed using the `W-W-H-W-W-W-H` pattern (W-Whole step, H-Half step). For example, the A4 major scale is as follows:

```
A4 - B4 - C#5 - D5 - E5 - F#5 - G#5 - A5
   W    W     H    W    W     W     H
```

They tend to express *positive* emotions like majesty, victory, curiosity, love, joy and so on, in general.

[Minor scales](http://en.wikipedia.org/wiki/Minor_scale) or *natural minor* scales are constructed using the `W-H-W-W-H-W-W` pattern. For example, the A4 minor is as follows:

```
A4 - B4 - C5 - D5 - E5 - F5 - G5 - A5
   W    H    W    W    W    W    W
```

They tend to express *negative* emotions like betrayal, melancholy, tragedy, ominousness and so on, in general.

Here's a document for further reading on [Characteristics of Musical Keys](http://biteyourownelbow.com/keychar.htm).

##### Intervals - unison, steps and leaps

An [interval](https://en.wikipedia.org/wiki/Interval_%28music%29) is the difference between two notes or pitches. The smallest interval is the semitone. Here are the intervals for the A4 minor scale (T - Tonic):

```
A4 - B4 - C5 - D5 - E5 - F5 - G5 - A5
T  - 1  - 2  - 3  - 4  - 5  - 6  - 7
```

C5, for instance, is 2 intervals away from A4 in the A4 minor scale. G5 is 6 intervals away.

Moving by zero intervals in the scale (staying on the same note) is called an [*unison*](https://en.wikipedia.org/wiki/Unison).
Moving by a single interval in the scale is called a [*step*](https://en.wikipedia.org/wiki/Steps_and_skips).
Moving by two or more intervals in the scale is called a [*leap*](https://en.wikipedia.org/wiki/Steps_and_skips).

##### Melodic motion - conjunct and disjunct

[Melodic motion](https://en.wikipedia.org/wiki/Melodic_motion) characterizes the tendency of a melody to *jump around*. The most common types of melodic motion are *conjunct motion* and *disjunct motion*, though there are lot of other ways in which a melody can be structured.

A melody that exhibits *conjunct motion* consists of a lot of *steps* (single intervals or successive notes) and *unisons* (same notes), and very little *leaps*.
A melody that exhibits *disjunct motion* consists of a lot of *leaps* (multiple intervals) and very little *steps* and *unisons*.

Most popular music uses *conjunct motion*. The melodies tend to be closely structured around a scale's notes by traversing the scale in steps, whilst avoiding leaps.

Let's pick apart the first few vocal measures of *My Generation* by *The Who*. These are roughly the notes of each of the syllables:

```
Peo - ple - try - to - put - us - down - talk - in' - 'bout - my - gen - er - a - tion
G   - G   - F   - F# - C   - C  - Bb   - D    - D   - E     - E  - F   - F# - E - D
```

This is a tune in the F major scale `F - G - A - A#/Bb - C - D - E - F` with an F# added in. Here's how the melodic motion is (U - Unison, S - Step, L - Leap):

```
Peo - ple - try - to - put - us - down - talk - in' - 'bout - my - gen - er - a - tion
G   - G   - F   - F# - C   - C  - Bb   - D    - D   - E     - E  - F   - F# - E - D
    U     S     S    L     U    S      L      U     S       S    S     S    S   S
```

That's just *two* leaps while we have *three* unisons and *nine* steps. This is clearly conjunct melodic motion.

Melodic motion is the most important quality of a melody that determines its quality and *listenability*. Simulating melodic motion is going to be the purpose of every procedure we write that generates music.

*...work in progress...*
