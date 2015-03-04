{:title "Longest Increasing Subsequence"
 :layout :post
 :tags  ["4clojure", "clojure"]}

[4clojure](http://www.4clojure.com) is an awesome resource -- I have been having a lot of fun working at these problems.  I am now tackling my first "hard" level problem,
[Longest Increasing Sub-Seq](http://www.4clojure.com/problem/53).  I have been (mostly) good about not searching for solutions or code so far.

>Given a vector of integers, find the longest consecutive sub-sequence of increasing numbers. If two sub-sequences have the same length, use the one that occurs first. An increasing sub-sequence must have a length of 2 or greater to qualify.

### plan of attack
My (hopefully idiomatic) approach to the problem is

1. Create a sequence producer-fn that takes the input and outputs all of the possible sequences therein
1. filter the sequences for length >= 2
1. Filter that list for ascending sequences
1. Sort that list by length, grab the first


### the easy bits
Filtering the list for ascending sequences is straightforward.
```
(apply < '(1 2 3)) ;;; true
(apply < '(1 3 2)) ;;; false
```
And filtering/sorting by count are trivial.

### the tricky part
How to structure this producer function?  I will use a multiple-arity function here.  The first arity, taking a single argument, is simply the entry point.
The single argument will be the starting sequence.  It will immediately call to the other arity.

The second arity will take 2 arguments
1. the sequence
1. a number of elements to take

Recursive calls incrementing the second argument appropriately will give us all permutation of sequences from the original.
1. Start with a take value of 1
1. Each call increment the take value
1. When the the take value exceeds the length of the sequence, reset the take value to 1 and take the tail of the sequence for the next call
1. When the sequence is empty?, we have reached the base case.

To that end, I came up with this for generating the sequence:
```
(fn gen-seq
  ([s] (gen-seq s 1))
  ([s t] (cond
           (empty? s) nil
           (= (count s) t) (cons s (lazy-seq (gen-seq (rest s) 1)))
           :else (cons (take t s) (lazy-seq (gen-seq s (inc t)))))))
```
Which generates the following:
```
(gen-seq '(5 2 3)) ;;; ((5) (5 2) (5 2 3) (2) (2 3) (3))
```

So good so far.  After adding the length and ascending filter we have...
```
(fn [x]
   ( ->> ((fn gen-seq
           ([s] (gen-seq s 1))
           ([s t] (cond
                    (empty? s) nil
                    (= (count s) t) (cons s (lazy-seq (gen-seq (rest s) 1)))
                    :else (cons (take t s) (lazy-seq (gen-seq s (inc t))))))) x)
         (filter (fn [e] (> (count e) 1)))
         (filter (fn [e] (apply < e)))))
```

But the first hitch in my plan has appeared.  The sort natural order places all of the
longest elements at the rear of the list, not the front.  Reversing the list causes the unit tests to fail since one of the requirements is grabbing the
first of the longest sequences of equal length, not the last.

Take 2:  we have a list of sequences.  We can group-by instead of sorting on count, find the max key, and take the first element of the corresponding value.
Structurally we add a let binding so we can manipulate the keys and the map after we build it.

```
(fn [x]
   (let [seq-map
         (->> ((fn gen-seq
                 ([s] (gen-seq s 1))
                 ([s t] (cond
                          (empty? s) nil
                          (= (count s) t) (cons s (lazy-seq (gen-seq (rest s) 1)))
                          :else (cons (take t s) (lazy-seq (gen-seq s (inc t))))))) x)
              (filter (fn [e] (> (count e) 1)))
              (filter (fn [e] (apply < e)))
              (group-by count))
         m-key (apply max (keys seq-map))]
     (first (seq-map m-key))))
```

Ah -- so close.  But now we are failing the unit test in which there is no ascending sequence at all.  This causes the max assignment to fail.
But now we simply add a nil check and we are good to go.  The final result:

```
(fn [x]
   (let [seq-map
         (->> ((fn gen-seq
                 ([s] (gen-seq s 1))
                 ([s t] (cond
                          (empty? s) nil
                          (= (count s) t) (cons s (lazy-seq (gen-seq (rest s) 1)))
                          :else (cons (take t s) (lazy-seq (gen-seq s (inc t))))))) x)
              (filter (fn [e] (> (count e) 1)))
              (filter (fn [e] (apply < e)))
              (group-by count))
         m-key (if (seq seq-map)
                 (apply max (keys seq-map))
                 nil)]
     (vec (first (seq-map m-key)))))
```
I think the result is pretty readable.



