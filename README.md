# floop
`floop` is a minimal, lisp-like language implemented in C.
The best way to get a quick feel for floop is to take a peek at the standard prelude, written in floop and defined in [`prelude.flp`](https://github.com/xlxs4/floop/blob/main/prelude.flp).

A way to compile floop would be `cc -std=c99 -Wall floop.c mpc.c -ledit -lm -o floop`.

Instead of macro-powered, quoted expressions, floop features Q-expressions; macroless unevaluated stuff.
Just surround an expression with curly braces:

```lisp
floop> {1 2 3 4}
{1 2 3 4}

floop> {crouch grossman}
{crouch grossman}
```

Staying true to its C ancestry, floop doesn't have boolean types.
Everything that isn't 0 is true:

```lisp
floop> (> 17 11)
1

floop> (< 17 11)
0
```

You can define stuff using `def`:

```lisp
floop> (def {shaman} "thrall")
()

floop> shaman
"thrall"
```

There's lambda functions (*duh*):

```lisp
floop> (\ {a b} {(+ a b)})
(\ {a b} {(+ a b)})

floop> ((\ {a b} {(+ a b)}) 1 2)
3
```

Prelude helps us define functions easier:

```lisp
floop> (def {fun} (\ {f b} {
         def (head f) (\ (tail f) b)
       }))
()

floop> (fun {not x} {- 1 x})
()

floop> (not 0)
1

floop> (not 1)
0
```

You can also `print`:

```lisp
floop> print "hi"
"hi"
()

floop> def {hi} "hi"
()

floop> print hi
"hi"
()

floop> print 23
23
()
```

Or `error`:

```lisp
floop> error "Instructions unclear: joined a cult"
Error: Instructions unclear: joined a cult
```

There's builtin functions:

```lisp
floop> head {+ - * /}
{+}

floop> eval (head {+ - * /})
<builtin>

floop> (eval (head {+ - * /})) 10 20
30
```

Some nifty, easy to piece-up things from prelude:

```lisp
(def {fun} (\ {f b} {
  def (head f) (\ (tail f) b)
}))

(fun {let b} {
  ((\ {_} b) ())
})

(fun {unpack f l} {
  eval (join (list f) l)
})

(fun {pack f & xs} {f xs})

(fun {do & l} {
  if (== l nil)
    {nil}
    {last l}
})

(fun {min & xs} {
  if (== (tail xs) nil) {fst xs}
    {do
      (= {rest} (unpack min (tail xs)))
      (= {item} (fst xs))
      (if (< item rest) {item} {rest})
    }
})

(fun {max & xs} {
  if (== (tail xs) nil) {fst xs}
    {do
      (= {rest} (unpack max (tail xs)))
      (= {item} (fst xs))
      (if (> item rest) {item} {rest})
    }
})

(fun {select & cs} {
  if (== cs nil)
    {error "No Selection Found"}
    {if (fst (fst cs)) {snd (fst cs)} {unpack select (tail cs)}}
})

(fun {case x & cs} {
  if (== cs nil)
    {error "No Case Found"}
    {if (== x (fst (fst cs))) {snd (fst cs)} {
	  unpack case (join (list x) (tail cs))}}
})

(fun {flip f a b} {f b a})

(fun {comp f g x} {f (g x)})

(fun {fst l} { eval (head l) })

(fun {snd l} { eval (head (tail l)) })

(fun {trd l} { eval (head (tail (tail l))) })

(fun {len l} {
  if (== l nil)
    {0}
    {+ 1 (len (tail l))}
})

(fun {nth n l} {
  if (== n 0)
    {fst l}
    {nth (- n 1) (tail l)}
})

(fun {last l} {nth (- (len l) 1) l})

(fun {map f l} {
  if (== l nil)
    {nil}
    {join (list (f (fst l))) (map f (tail l))}
})

(fun {filter f l} {
  if (== l nil)
    {nil}
    {join (if (f (fst l)) {head l} {nil}) (filter f (tail l))}
})

(fun {init l} {
  if (== (tail l) nil)
    {nil}
    {join (head l) (init (tail l))}
})

(fun {reverse l} {
  if (== l nil)
    {nil}
    {join (reverse (tail l)) (head l)}
})

(fun {foldl f z l} {
  if (== l nil)
    {z}
    {foldl f (f z (fst l)) (tail l)}
})

(fun {foldr f z l} {
  if (== l nil)
    {z}
    {f (fst l) (foldr f z (tail l))}
})

(fun {take n l} {
  if (== n 0)
    {nil}
    {join (head l) (take (- n 1) (tail l))}
})

(fun {drop n l} {
  if (== n 0)
    {l}
    {drop (- n 1) (tail l)}
})

(fun {split n l} {list (take n l) (drop n l)})

(fun {take-while f l} {
  if (not (unpack f (head l)))
    {nil}
    {join (head l) (take-while f (tail l))}
})

(fun {drop-while f l} {
  if (not (unpack f (head l)))
    {l}
    {drop-while f (tail l)}
})

(fun {elem x l} {
  if (== l nil)
    {false}
    {if (== x (fst l)) {true} {elem x (tail l)}}
})

(fun {lookup x l} {
  if (== l nil)
    {error "No Element Found"}
    {do
      (= {key} (fst (fst l)))
      (= {val} (snd (fst l)))
      (if (== key x) {val} {lookup x (tail l)})
    }
})

(fun {zip x y} {
  if (or (== x nil) (== y nil))
    {nil}
    {join (list (join (head x) (head y))) (zip (tail x) (tail y))}
})

(fun {unzip l} {
  if (== l nil)
    {{nil nil}}
    {do
      (= {x} (fst l))
      (= {xs} (unzip (tail l)))
      (list (join (head x) (fst xs)) (join (tail x) (snd xs)))
    }
})
```
