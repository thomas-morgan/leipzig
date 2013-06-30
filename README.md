[Leipzig](https://github.com/ctford/leipzig)
=========

[![Build Status](https://travis-ci.org/ctford/leipzig.png)](https://travis-ci.org/ctford/leipzig)

A composition library for [Overtone](https://github.com/overtone/overtone) by [@ctford](https://github.com/ctford).

Examples
--------
See [Row, row, row your boat](src/leipzig/example/row_row_row_your_boat.clj) or [whelmed](https://github.com/ctford/whelmed).

Using it
--------
Include it as a dependency in your project.clj:

    [leipzig "0.6.0"]

API
---

[API documentation](http://ctford.github.io/leipzig/), generated by [Codox](https://github.com/weavejester/codox).

Building a melody
-----------------

Leipzig models music as a sequence of notes, each of which is a map:

    {:time 2000
     :pitch 67
     :duration 1000
     :part :melody}

You can create a melody with the `phrase` function. Here's a simple melody:

    (def melody
      (->>
        (phrase [2 2 1 1 2]
                [2 0 2 3 4])
        (where :part (is :melody))))

The first argument to `phrase` is a sequence of durations. The second is a sequence of pitches. `phrase` builds a sequence of notes which we can transform with sequence functions, either from Leipzig or ones from Clojure's core libraries. In this case, we've used `where` to set the `:part` key of each note to `:melody`.

To play a melody, first define an arrangement. `play-note` is a multimethod that dispatches on the `:part` key of each note, so you can easily define an instrument responsible for playing notes of each part. Then, put the sequence of notes into a particular key and tempo and pass them along to `play`:

    (defmethod play-note :melody [{midi :pitch}] (sampled-piano midi))

    (->>
      melody
      (where :time (bpm 90))
      (where :duration (bpm 90))
      (where :pitch (comp C major))
      play)

Actually, `phrase` accepts more than just pitches. `nil`s are interpreted as rests, vectors and lists as clusters and maps as chords. Here's a more advanced example that plays a 1/4/5 chord progression on the offbeat. We'll supply a default arrangement so that the accompaniment is played on the piano too:

    (defmethod play-note :default [{midi :pitch}] (sampled-piano midi))

    (def accompaniment
      (->>
        (phrase (repeat 1)
                [nil triad nil triad nil (-> triad (root 3)) nil (-> triad (root 4))])
        (where :pitch lower)))

You can then put multiple series of notes together:

    (->>
      melody
      (with accompaniment)
      (where :time (bpm 90))
      (where :duration (bpm 90))
      (where :pitch (comp C major))
      play)

Design
------

Leipzig is designed to play nicely with Clojure's standard sequence functions. Therefore, Leipzig's functions for transforming notes all take the sequence as a final argument so that they can be threaded with the `->>` macro:

    (->>
      (phrase (repeat 1) (cycle [0 2 4]))
      (take 24)
      (filter #(-> % :time even?)))

These sequence functions all exhibit "closure" i.e. their result is the same shape as their input. That allows them to be used and combined very flexibly. `where` for example, can be used to set the part, raise the pitch or put the notes into a particular tempo: 

    (->> notes (where :part (is :bass)))
    (->> notes (where :pitch inc))
    (->> notes (where :time (bpm 90)))

Leipzig aims to be a library rather than a framework or environment. It uses simple Clojure datastructures and strives to be as open as possible. A new timing scheme, tuning or tempo can be mixed with Leipzig's other functions just as easily as the ones that come with the library.

Tests
-----

To run the unit tests without having to start Overtone's Supercollider server:

    lein midje leipzig.test.*
