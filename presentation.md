# cq

-----

### cq origins

* Jonathan gave a talk about jq
* Chouser used jq to do some Advent of Code puzzles
* We found jq inspiring and frustrating
* So we asked:

#### What if we had the power of jq in Clojure?

Notes:
* The easier-to-answer question was what would it be called.
* The harder, how would it look and behave, took a little longer ...2 months later...

-----

## cq: & (exploding fns!)

```clojure [1|2|3|19|20|37|34-37]
    (cq (+ 1 2))             ;;=> (3)
    (cq (& 3 4))             ;;=> (3 4)
    (cq (+ 1 (& 3 4)))       ;;=> ???
    
    
    
    
                                                                 


    
    
    
    
    
    
    (cq (+ 1 2))             ;;=> (3)
    (cq (& 3 4))             ;;=> (3 4)
    (cq (+ 1 (& 3 4)))       ;;=> (4 5)
    (cq (+ (& 1 2) (& 3 4))) ;;=> ???
    
    
    
    
                                                                 


    
    
    
    
    
    
    (cq (+ 1 2))             ;;=> (3)
    (cq (& 3 4))             ;;=> (3 4)
    (cq (+ 1 (& 3 4)))       ;;=> (4 5)
    (cq (+ (& 1 2) (& 3 4))) ;;=> (4 5 5 6)
```

Notes:
* & returns multiple values, as you can see here. How does + behave?
* What if you give multiple streams to +

---
## takeways
* clojure fns work
* & 'explodes' the context it is within

-----

## cq: |, pick, modify
    
```clojure [1|2|4-5|7-9|11-14]
(def data {:coords [2 3]})
(cq (pick data :coords))           ;;=> ([2 3])

(cq (| data
       (pick . :coords)))          ;;=> ([2 3])

(cq (| data
       (pick . :coords)
       (pick . 0)))                ;;=> (2)

(cq (| data
       (modify (| (pick . :coords)
                  (pick . 0))
               (* . 10))))         ;;=> ({:coords [20 3]})


```
---
## takeways
* `pick` acts like `get` (but "navigates")
* `|` threads multiple values as `.` into following forms
* `modify` works on navigations

-----
## cq: all the things at once

```clojure [1-3|5-6|8-13]
(def people
  [{:first "alan" :last "turing"}
   {:first "markus" :last "kaltenbach"}])

(defn title-case [s]
  (clojure.string/replace s #"^." #(.toUpperCase %)))

(cq (| people
       (modify (-> . each (pick (& :first :last)))
               (title-case .))))

;;=> ([{:first "Alan", :last "Turing"}
;;     {:first "Markus", :last "Kaltenbach"}])
```

---
## takeways
* Clojure macros and user defined functions work too
* use `each` to navigate all items

-----
## cq primitives

| | |
| - | - |
| **Stream**           | `\|`,  `&`, `collect`, `each` |
| **Navigation**       | `pick`, `slice` |
| **Update**           | `modify`, `assign` |
| **Normal functions** | `+`, `-`, `*`, `str`, _etc._ |
| | |

_Only a handful of new operations_

Notes:
* ...so where did these come from?

-----

#### primitives in cq and jq

```clojure [1-12]
;; cq                ; jq

(| a b c)            ; a|b|c
(& a b c)            ; a,b,c
(collect s)          ; [s]
(each v)             ; v[]

(pick c i)           ; c[i]
(slice c j k)        ; c[j:k]

(modify nav update)  ; nav |= update
(assign nav val)     ; nav = val
```

Notes:
* note that square brackets mean different things in different contexts

-----

#### leverage clojure's builtins

```clojure [1-10]
;; cq                       ; jq

(each (range j k))          ; range(j;k)

(first [a b c])             ; [a,b,c] | first
(first (collect a b c))     ; first(a,b,c)

(some true? [a b c])        ; [a,b,c] | any
(some pred [a b c])         ; [a,b,c] | any(pred)
(some pred (collect a b c)) ; any(a,b,c;pred)
```

-----
## obligitory CS slide

After extracting the essense of jq into Clojure, we believe jq boils down to the
combination of two functional programming concepts:
* Lenses

  <small>https://ncatlab.org/nlab/show/lens+(in+computer+science)</small>

* List Monads (Free Monoid Monads)

  <small>https://ncatlab.org/nlab/show/free+monoid</small>

Notes:
* The first gives us the ability to query or update immutable data with the same
expression
* The Second gives us a way to compose operations on deep data structures that
can impact multiple areas of the.

-----
## what we made

* First implementation: monadic functions plus lenses
* jq to cq compiler
* Second implementation: code-walking macro (e.g., `clojure.core.async/go`) plus lenses

-----
## Conclusion

* The combination of lenses and ubiquitous mapcat is a surprisingly simple and potent tool for manipulating complex data structures is jq
* Similar primitives can be written in Clojure and combinded with a set of regular rules for "lifting" all of Clojure's functions into the same world
* This appears to give us the power of jq on Clojure data with Clojure syntax


