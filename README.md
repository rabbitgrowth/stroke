# stroke

A pair of functions for converting between steno strokes and boolean vectors.

## Summary

    stroke ← [letters [numbers]] Stroke steno
    steno  ← [letters [numbers]] Steno  stroke

## Usage

The steno layout is specified as a vector of left-, middle-, and right-bank keys.
You can optionally specify numbers as a vector with the same structure where spaces indicate the absence of numbers.

          letters←'#STKPWHR' 'AO*EU' 'FRPBLGTSDZ'
          numbers←' 12 3 4 ' '50   ' '6 7 8 9   ' ⍝ optional

Use `Stroke` to convert raw steno into a boolean vector
where `0` indicates an unpressed key and `1` indicates a pressed key, arranged in steno order.
The left argument is the steno layout, which can be

- omitted, in which case the default English layout is used;
- given as `letters`, in which case that will be used as the layout;
- given as `letters numbers`, in which case numbers will be parsed as well.

<!-- dummy comment to force the code block to render properly -->

          Stroke 'STROEBG'
    0 1 1 0 0 0 0 1 0 1 0 1 0 0 0 0 1 0 1 0 0 0 0
          letters Stroke 'STROEBG'
    0 1 1 0 0 0 0 1 0 1 0 1 0 0 0 0 1 0 1 0 0 0 0
          letters numbers Stroke '134*EU9'
    1 1 0 0 1 0 1 0 0 0 1 1 1 0 0 0 0 0 0 1 0 0 0

Invalid strokes give `DOMAIN ERROR`:

          letters Stroke 'STROKE'
    DOMAIN ERROR

In the other direction, use `Steno` to convert a boolean vector into raw steno.
It takes the same left arguments as `Stroke`.

          Steno 0 1 1 0 0 0 0 1 0 1 0 1 0 0 0 0 1 0 1 0 0 0 0
    STROEBG
          letters Steno 0 1 1 0 0 0 0 1 0 1 0 1 0 0 0 0 1 0 1 0 0 0 0
    STROEBG
          letters Steno 1 1 0 0 1 0 1 0 0 0 1 1 1 0 0 0 0 0 0 1 0 0 0
    #SPH*EUT
          letters numbers Steno 1 1 0 0 1 0 1 0 0 0 1 1 1 0 0 0 0 0 0 1 0 0 0
    134*EU9

Unlike raw steno, the boolean vectors can be combined easily.
For example, to stack strokes, simply OR the boolean vectors:

          Stack←Steno∨⍥Stroke
          'APB'Stack'PHAL'
    PHAPBL
          ⊃Stack/'ST' 'HRAOU' '-S'
    STHRAOUS
          'TH' 'THA'∘.Stack'-BG' '-BGD'
     TH-BG  TH-BGD
     THABG  THABGD

## Examples

Parsing `main.json`:

          (keys vals)←↓⍉dict←0 1 1 0/1↓⎕JSON⍠'M'⊃⎕NGET'main.json'
          strokes←⊃⍪/outlines←{↑Stroke¨'/'(≠⊆⊢)⍵}¨keys

Finding the frequency of each key by percentage:

          (∊letters)↑⍤,⍥⊂(⌊0.5+100×⊢÷+/)+⌿strokes
    # S T K P W H R A O *  E U F R P B L G T S D Z
    0 3 5 5 5 3 5 6 8 6 2 12 8 1 3 5 6 3 3 3 3 2 0

Finding entries with the fewest keys for the most characters:

          dict[5↑⍒(≢¨vals)÷+/⍣2¨outlines;]
     T*TD   terminal deoxynucleotidyl transferase
     TKH*F  acute decompensated heart failure
     P-B    peanut butter
     S-D    second-degree
     R*F    reticular formation

Finding the shortest unused right-hand chords:

          all    ← ⍉(23⍴2)⊤1-⍨⍳2*10
          used   ← 0@(⍳13)⍤1⊢strokes
          unused ← all~⍥↓used
          Steno¨{⍵[10↑⍋+/¨⍵]}unused
     -SDZ  -GTZ  -LSD  -BSD  -BTZ  -PSZ  -PTZ  -PGZ  -FTZ  -FGZ

Finding visually symmetrical strokes with more than ten keys:

          mask         ← Stroke'#-TDZ'
          firstStrokes ← ↑⊣⌿¨outlines
          extraKeys    ← mask/firstStrokes
          coreKeys     ← (~mask)/firstStrokes
          left         ← coreKeys[;⍳9]
          right        ← coreKeys[;10+⍳9]
          mirror       ← right[;1 2 4 3 6 5 8 7 9]
          oneStroke    ← 1=≢¨outlines
          manyKeys     ← 10≤+/firstStrokes
          noExtraKeys  ← ~∨/extraKeys
          symmetrical  ← left≡∘⌽⍤1⊢mirror
          dict⌿⍨symmetrical∧noExtraKeys∧manyKeys∧oneStroke
     KPWAOEUPBG     combining
     STPAOEUPLS     sometimes
     TKPWAUPBLG     gauge
     TKPWHR*FRPBLG  grasshopper
     TKPWR*RPBLG    Grand Juror
     TKPWR-RPBLG    grand juror

Finding outlines that can be stacked without violating steno order,
but excluding cases where the last stroke is an inflected ending:

          endings     ← Stroke¨'-',¨'GSD'
          Stackable   ← {∧/</¯1↓0 1⊖(⊢/,⊣/)⍤⍸⍤1⊢⍵}
          Uninflected ← {~endings∊⍨⊂⊢⌿⍵}
          dict⌿⍨(Uninflected∧Stackable∧1<≢)¨outlines
     AEUR/-BL  arable
     AEUR/-LS  airless
     AF/-BL    affable
     AF/-BLT   affability
     AFRP/-L   ample
     ...
