# stroke

A pair of functions for converting between steno strokes and boolean vectors.

## Summary

    booleanVector ← [letters [numbers]] Parse     rawSteno
    rawSteno      ← [letters [numbers]] Serialize booleanVector

## Usage

Use `Parse` to convert raw steno representing a stroke into a boolean vector
with a `0` (unpressed) or `1` (pressed) for each key in the layout:

          Parse 'STROEBG'
    0 1 1 0 0 0 0 1 0 1 0 1 0 0 0 0 1 0 1 0 0 0 0

Invalid strokes give `DOMAIN ERROR`:

          Parse 'STROKE'
    DOMAIN ERROR

By default the standard English layout is used.
To use a specific layout,
provide a vector of left-, middle-, and right-bank keys as the left argument:

          letters←'#STKPWHR' 'AO*EU' 'FRPBLGTSDZ'
          letters Parse 'STROEBG'
    0 1 1 0 0 0 0 1 0 1 0 1 0 0 0 0 1 0 1 0 0 0 0

You could optionally also specify numbers as a vector with the same structure as the letters,
with spaces indicating the absence of numbers:

          numbers←' 12 3 4 ' '50   ' '6 7 8 9   ' ⍝ optional
          letters↑⍤,⍥⊂numbers
     #STKPWHR  AO*EU  FRPBLGTSDZ
      12 3 4   50     6 7 8 9
          letters numbers Parse '123450'
    1 1 1 0 1 0 1 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0

In the other direction, use `Serialize` to convert a boolean vector into raw steno.
It takes the same left arguments as `Parse`.

          Serialize 0 1 1 0 0 0 0 1 0 1 0 1 0 0 0 0 1 0 1 0 0 0 0
    STROEBG
          letters Serialize 0 1 1 0 0 0 0 1 0 1 0 1 0 0 0 0 1 0 1 0 0 0 0
    STROEBG
          letters Serialize 1 1 1 0 1 0 1 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0
    #STPHAO
          letters numbers Serialize 1 1 1 0 1 0 1 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0
    123450

Unlike raw steno, the boolean vectors can be combined easily.
For example, to stack strokes, simply OR the boolean vectors:

          Stack←Serialize∨⍥Parse
          'APB' Stack 'PHAL'
    PHAPBL
          ⊃Stack/'ST' 'HRAOU' '-S'
    STHRAOUS
          'TH' 'THA'∘.Stack'-S' '-FS'
     TH-S  TH-FS
     THAS  THAFS

## Examples

Parsing `main.json`:

          (keys vals)←↓⍉dict←0 1 1 0/1↓⎕JSON⍠'M'⊃⎕NGET'main.json'
          strokes←⊃⍪/outlines←{↑Parse¨'/'(≠⊆⊢)⍵}¨keys

Finding the frequency of each key by percentage:

          (∊letters)↑⍤,⍥⊂(⌊0.5+100×⊢÷+/)+⌿strokes
    # S T K P W H R A O *  E U F R P B L G T S D Z
    0 3 5 5 5 3 5 6 8 6 2 12 8 1 3 5 6 3 3 3 3 2 0

Finding entries with the fewest keys for the most characters:

          dict[⍒(≢¨vals)÷+/⍣2¨outlines;]
     T*TD   terminal deoxynucleotidyl transferase
     TKH*F  acute decompensated heart failure
     P-B    peanut butter
     S-D    second-degree
     R*F    reticular formation
     ...

Finding the shortest unused right-hand chords:

          all    ← ⍉(23⍴2)⊤1-⍨⍳2*10
          used   ← 0@(⍳13)⍤1⊢strokes
          unused ← all~⍥↓used
          Serialize¨{⍵[⍋+/¨⍵]}unused
     -SDZ  -GTZ  -LSD  -BSD  -BTZ  -PSZ  -PTZ  -PGZ  ...

Finding visually symmetrical strokes:

          mask         ← Parse'#-TDZ'
          firstStrokes ← ↑⊣⌿¨outlines
          extraKeys    ← mask/firstStrokes
          coreKeys     ← (~mask)/firstStrokes
          left         ← coreKeys[;⍳9]
          right        ← coreKeys[;10+⍳9]
          mirror       ← right[;1 2 4 3 6 5 8 7 9]
          oneStroke    ← 1=≢¨outlines
          noExtraKeys  ← ~∨/extraKeys
          symmetrical  ← left≡∘⌽⍤1⊢mirror
          dict⌿⍨symmetrical∧noExtraKeys∧oneStroke
     ...
     STAO*EULS   stylus
     STAOEULS    styles
     STAULS      stalls
     STP*PLS     symptoms
     STPAOEUPLS  sometimes
     ...

Finding outlines where strokes can be stacked without violating steno order,
but excluding cases where the last stroke is an inflected ending:

          endings     ← Parse¨'-',¨'GSD'
          Stackable   ← {∧/</¯1↓0 1⊖(⊢/,⊣/)⍤⍸⍤1⊢⍵}
          Uninflected ← {~endings∊⍨⊂⊢⌿⍵}
          dict⌿⍨(Uninflected∧Stackable∧1<≢)¨outlines
     ...
     AEUR/-BL  arable
     AEUR/-LS  airless
     AF/-BL    affable
     AF/-BLT   affability
     AFRP/-L   ample
     ...
