# A Meta-circular Little Scheme

This is a meta-circular interpreter of a subset of Scheme.
It implements the same language as
[little-scheme-in-python](https://github.com/nukata/little-scheme-in-python)
(`scm.py`) and runs on it.
It also runs on other Schemes such as
[little-scheme-in-go](https://github.com/nukata/little-scheme-in-go) and
[guile](https://www.gnu.org/software/guile/).
It will run on any R5RS Schemes.

It is inspired by [Zick Standard Lisp](https://github.com/zick/ZickStandardLisp).

## How to use

Run `scm.scm` on another Scheme.

```
$ scm.py scm.scm
(+ 5 6)
=> 11
(cons 'a (cons 'b 'c))
=> (a b . c)
(list
1
2
3
)
=> (1 2 3)
+
=> ($Intrinsic . #<(x):((+ (fst x) (snd x))):#-0x7fffffffefa062e5>)
(globals)
=> (globals = < * - + symbol? eof-object? read newline display apply call/cc list
 not null? pair? eqv? eq? cons cdr car)
```

Press EOF (e.g. Control-D) to exit the session.


## Examples

There are four files under the `examples` folder, which
are copied from 
[little-scheme-in-python/examples](https://github.com/nukata/little-scheme-in-python/tree/v1.1.0/examples):

- [`fib90.scm`](examples/fib90.scm)
  calculates Fibonacci for 90 tail-recursively.

- [`nqueens.scm`](examples/nqueens.scm)
  runs an N-Queens solver for 6.

- [`dynamic-wind-example.scm`](examples/dynamic-wind-example.scm)
  demonstrates the example of `dynamic-wind` in R5RS.

- [`yin-yang-puzzle.scm`](examples/yin-yang-puzzle.scm)
  runs the yin-yang puzzle with `call/cc`.

```
$ scm.py scm.scm < examples/fib90.scm 
2880067194370816120
$ time scm.py scm.scm < examples/nqueens.scm
((5 3 1 6 4 2) (4 1 5 2 6 3) (3 6 2 5 1 4) (2 4 6 1 3 5))

real	2m8.860s
user	2m8.746s
sys	0m0.070s
$ scm.py scm.scm < examples/dynamic-wind-example.scm 
(connect talk1 disconnect connect talk2 disconnect)
$ scm.py scm.scm < examples/yin-yang-puzzle.scm

*
**
***
****
*****
******
*******
********
*********
**********
```

Press the interrupt key (e.g. Control-C) to stop the yin-yang puzzle.

As you see the above, `python` is rather slow to run '`scm.py scm.scm`'.
You can use an alternative implementation [PyPy](https://pypy.org) instead.

```
$ time pypy /usr/local/bin/scm.py scm.scm < examples/nqueens.scm 
((5 3 1 6 4 2) (4 1 5 2 6 3) (3 6 2 5 1 4) (2 4 6 1 3 5))

real	0m3.417s
user	0m3.364s
sys	0m0.047s
$ time pypy3 /usr/local/bin/scm.py scm.scm < examples/nqueens.scm 
((5 3 1 6 4 2) (4 1 5 2 6 3) (3 6 2 5 1 4) (2 4 6 1 3 5))

real	0m3.939s
user	0m3.880s
sys	0m0.053s
$ 
```


## Tower of meta-circular interpreters

Being meta-circular, this interpreter is able to run itself recursively.

 1. Copy the interpeter file `scm.scm` to `scm-scm.scm`.

 2. Comment out the last line `(read-eval-print-loop)` of `scm-scm.scm`.

```Scheme
;; (read-eval-print-loop)
```

 3. Append two new lines `(global-eval '(begin` and `))` to `scm-scm.scm`.

```Scheme
;; (read-eval-print-loop)
(global-eval '(begin
))
```

 4. Insert the whole contents of `scm.scm` between the new lines.

```Scheme
;; (read-eval-print-loop)
(global-eval '(begin
;; A meta-circular little Scheme v1.0 H31.03.18 by SUZUKI Hisao
...
(read-eval-print-loop)
))
```

 5. Run `scm-scm.scm` on another Scheme.

For your convenience, I have built it as [`tower/scm-scm.scm`](tower/scm-scm.scm).

```
$ scm.py tower/scm-scm.scm
(+ 5 6)
=> 11
+
=> ($Intrinsic $Closure (x) ((+ (fst x) (snd x))) #<(op):((if (eq? op (quote car)
) CAR (if (eq? op (quote cdr)) CDR (if (eq? (car op) (quote cdr)) (set! CDR (cdr 
op)) (display (list (quote Unknown-op) op CAR CDR)))))):#0x10d474e6>)
```

Note that the _intrinsic_ function `+` is now implemented by a _closure_
of `scm.scm`, the underlying Scheme here.

You can repeat the above process any times.
Try [`tower/scm-scm-scm.scm`](tower/scm-scm-scm.scm) and you will find it runs
prohibitively _slowly_ as might be expected.

```
$ time guile example/fib90.scm
2880067194370816120

real	0m0.030s
user	0m0.016s
sys	0m0.013s
$ time guile scm.scm < example/fib90.scm
2880067194370816120

real	0m0.032s
user	0m0.019s
sys	0m0.013s
$ time guile tower/scm-scm.scm < examples/fib90.scm
2880067194370816120

real	0m0.328s
user	0m0.351s
sys	0m0.025s
$ time guile tower/scm-scm-scm.scm < examples/fib90.scm
2880067194370816120

real	1m13.853s
user	1m25.390s
sys	0m3.957s
$ 
```

I hope this serves as a good benchmark test for Scheme.
