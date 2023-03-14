- vm

    - modern systems are `byte-addressable`
    - all valid addresses make up the `virtual address space`
    - the 2<sup>64</sup>-bit virtual address space size is determined by the 64-bit `word length`
    - data type spaces might vary since machines have different word length

- int

    - repr

        - be aware of `Little Endian` or `Big Endian` in network transport or disassembly
        - truncate / left shift logically, extend / right shift arithmetically
        - the highest bit of a signed is the `negative weight`

    - manip

        - add or mul might produce more bits, truncate them after op
        - overflow / underflow to the other half range
        - use `(x < 0 ? x + (1 << k) - 1 : x) >> k` in division by  2<sup>k</sup> to round to 0
        - whenever `signed op unsigned`, `signed` -> `unsigned`
        - `x%y = x - y(x/y)`; `if (x == Tmin) x == -x;`

- double

    - repr

        - `1b sign + 11b exp + 52b frac`

            | type         | exp                 | value                                               |
            | ------------ | ------------------- | --------------------------------------------------- |
            | denormalized | all 0               | 0.frac * 2<sup>-1022</sup> (including +0 & -0)      |
            | normalized   | not all 0 nor all 1 | (-1)<sup>sign</sup> * 1.frac * 2<sup>exp-1023</sup> |
            | special      | all 1               | frac == 0 ? inf : NaN (illegal like √-1 or ∞*0)     |

        - IEEE 754 has denser distribution as closer to 0, while the `denormalized <=> normalizedwith` transition is smooth

    - manip

        - new frac in add or mul, shift & truncate bits for correct repr
        - be careful of the precision & overflow in ops
        - overflow if value exceeds the range; truncate if the frac is shorter
        - float/double -> int, round to 0; int -> float, lossless