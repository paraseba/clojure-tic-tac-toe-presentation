<!SLIDE bullets>
#Clojure Introduction

* Sebastián B. Galkin
* @paraseba

<!SLIDE smbullets>
#TOC

* Some problems with our day to day languages
* Lisp/Clojure
* Syntax
* Data structures
* State/Identity
* Sample code


<!SLIDE smaller>

# Problem 1

    @@@ruby
    class DiceSet
      attr_reader :values

      def initialize; @values = []; end

      def roll(x)
        @values.clear
        x.times { @values << rand(6) + 1 }
      end
    end

    dice = DiceSet.new
    dice.roll(5)
    roll1 = dice.values
    dice.roll(5)
    roll2 = dice.values

    assert_not_equal roll1, roll2

<!SLIDE smaller>
# Problem 2

    @@@java
    public class Account {
       public void deposit(int amount) {
         if (balance + amount > 0)
           balance += amount;
         else
           error!!!!
       }
    }

    // Thread 1
    account1.deposit(100);

    // Thread 2
    account1.withdraw(100);

<!SLIDE smaller>
# Problem 2 (fix?)

    @@@java
    public class Account {
       public synchronized void deposit(int amount) {
         if (balance + amount > 0)
           balance += amount;
         else
           error!!!!
       }
    }

<!SLIDE smaller>
# Problem 2b

    @@@java
    public class Account {
       public void transfer(Account other, int amount) {
         this.deposit(- amount);
         other.deposit(amount);
       }
    }

<!SLIDE smaller>
# Problem 2b

    @@@java
    // Thread 1
    account1.transfer(account2, 100);

    // Thread 2
    account1.balance() + account2.balance()

<!SLIDE smaller>
# Problem 2b (fix?)

    @@@java
    public class Account {
      public void transfer(Account other, int amount) {
        synchronized(this) {
          synchronized(other) {
            this.deposit(- amount);
            other.deposit(amount);
          }
        }
      }
    }

<!SLIDE smaller>
# Problem 2c

    @@@java
    // Thread 1
    account1.transfer(account2, 100);

    // Thread 2
    account2.transfer(account1, 100);


<!SLIDE title-slide>
# Mutable objects
## The one and only problem

<!SLIDE smbullets>
# Complexity

* Hard to reason about programs
* Hard to test programs
* Hard to parallelize programs

<!SLIDE title-slide>
# Immutability
## The solution

<!SLIDE smbullets>
# Clojure

* Functional language
* Runs on the JVM (and CLR)
* Compiled
* 3 years old
* Great community
* Active development
* Several new concepts and ideas
* Several old and great concepts and ideas

<!SLIDE smbullets>
# It is a Lisp

* Language family
* Born in 1958 (Fortran is older)
* Influenced by Church's Lambda Calculus
* Code = Data (homoiconic)
* Lots of parentheses
* Maybe the most expressive language ever invented


<!SLIDE smbullets>
# Native types

    1, 3, 5.5, "hello", 3/4, POJO
    Lists (1 2 3 "hello")
    Vectors [1 2 3 "hello"]
    Maps {1 "hello", :two "bye"}
    Sets #{1 2 3 "hello"}

<!SLIDE >

#A function call is just a list

    (println "Hello World!")

    (function-name arguments...)


<!SLIDE small>
# Function definition

    (fn [x y]
      (+ (* x x) (* y y)))

    (defn square-sum
      "The sum of the squares"
      [x y]
      (+ (* x x)
         (* y y)))

    (def square-sum
      (fn [x y] (+ (* x x) (* y y))))

<!SLIDE >
## Function definition is just lists and vectors

<!SLIDE >
# Anonymous functions

    #(inc %)
    #(+ %1 3 %2)
    (map #(inc %) [2 3 4])
    (map inc [2 3 4])

<!SLIDE >
# Functions are first citizens

    (defn increment [n]
      (fn [x]
        (+ n x)))

    [increment (increment 5)
     + #(+ 1 %) (foo increment)]

<!SLIDE >
# Turing Complete

    (defn factorial [n]
      (if (= n 0)
        1
        (* n (factorial (- n 1)))))

<!SLIDE title-slide>
# So it's all data

<!SLIDE >
# And that's it
## Did you notice I didn't mention assignment?

<!SLIDE smbullets>
# Persistent collections

* Immutable
* You operate on them with functions that return new collections
* They maintain performance guarantees, no copy all behavior
* Old copies also maintain guarantees
* You'll have this in you language pretty soon

<!SLIDE>
# Basic operations

    (conj '(1 2 3) 0)      ->  (0 1 2 3)
    (conj [1 2 3] 4)       ->  (1 2 3 4)
    (assoc {1 2} 3 4 1 5)  -> {3 4 1 5}

<!SLIDE>
# "Variables"
## Lexical scope

    (let [x 1 y 2]
      (if (> x 1)
       (+ x y)
       (- x y)))

<!SLIDE>
# Keywords

    (= :foo :foo)

    (let [m {:a 1, :b 2}]
      (m :a)
      (:a m))


<!SLIDE smbullets>
# Sequences

* first, rest, cons
* May be lazy
* Used everywhere in core Clojure functions

<!SLIDE small>
# Laziness & composability

    (first
      (filter
        even?
        (map inc (range 1e6))))

    (def naturals (iterate inc 1))

    (take 5 naturals)

    (defn indexed [s]
      (map vector (iterate inc 0) s))

    (indexed [:a :b :c])  ->  ([0 :a] [1 :b] [2 :c])

<!SLIDE >
# How to model without state
## You'll probably need some practice

<!SLIDE smbullets>
# How to model without state

* Values, not state
* Functions return new values
* Pure functions
* Use maps instead of objects
* Mutability propagates

<!SLIDE title-slide>
# Identity concept changes

<!SLIDE smbullets>

* In OOP identity = memory location
* Identity = pointer
* In Clojure we change the values pointed by an identity
* Values change by means of pure functions


<!SLIDE smbullets>
# Clojure offers

* A mechanism to handle identity in your program
* A way to represent time and change
* A mechanism for change coordination

<!SLIDE>
# Three types of identities

###Depending of how to handle change and coordination

<table style="width:70%; margin: auto auto; border:solid 2px; text-align: center;" >
  <tr>
    <th>Reference</td>
    <th>Coordinated</td>
    <th>Synchronic</td>
  </tr>

  <tr>
    <td>ref</td>
    <td>Yes</td>
    <td>Yes</td>
  </tr>

  <tr>
    <td>atom</td>
    <td>No</td>
    <td>Yes</td>
  </tr>

  <tr>
    <td>agent</td>
    <td>No</td>
    <td>No</td>
  </tr>
</table>


<!SLIDE>
# ref

    (defn transfer [amount acc1 acc2]
      (dosync
        (alter acc1 update-in [:balance]
               #(- % amount))
        (alter acc2 update-in [:balance]
               #(+ % amount))))

    (let [acc1 (ref {:balance 100})
          acc2 (ref {:balance 300})]
        (transfer 50 acc1 acc2)
        @acc1
        @acc2)

<!SLIDE>
# atom

    (def processed-requests (atom 0))

    (defn processed []
      (swap! processed-requests inc))

    @processed-requests

<!SLIDE>
# agent

    (def particle-position (atom [0 0]))

    (defn move [position forces]
      ...
      [new-x new-y])

    (send particle-position move acting-forces)

<!SLIDE>
# Show me the code
## tic-tac-toe
### https://github.com/paraseba/tictactoe

<!SLIDE small>
# The board

    (defn make-board [x-cells o-cells]
      {:x (set x-cells) :o (set o-cells)})

    (def x-cells :x)
    (def o-cells :o)

    (def all-cells (apply sorted-set (range 9)))

    (defn empty-cells [board]
      (clojure.set/difference all-cells
                              (x-cells board)
                              (o-cells board)))
<!SLIDE small>
# The board

    (defn mark [board cell]
      (assert (contains? (empty-cells board) cell))
      (let [turn (fn [board]
                   (if (> (count (x-cells board))
                          (count (o-cells board)))
                     :o
                     :x))

            t (turn board)]
        (assoc board
               t
               (conj (t board) cell))))

<!SLIDE smaller>
# Winner

    (def win-cells
      (let [row1 #{0 1 2}
            row2 #{3 4 5}
            row3 #{6 7 8}
            col1 #{0 3 6}
            col2 #{1 4 7}
            col3 #{2 5 8}
            dia1 #{0 4 8}
            dia2 #{2 4 6}]
        [row1 row2 row3 col1 col2 col3 dia1 dia2]))

    (defn won? [cells]
      (some #(every? cells %) win-cells))

    (defn winner [position]
      (cond
        (won? (:x position)) :x
        (won? (:o position)) :o))

<!SLIDE smbullets>
# The AI

* Start with a given position
* Generate the tree of all possible moves
* Pick the best one assuming that the opponent will pick his best one

<!SLIDE>
# Posible moves

    (defn moves [board]
      (if (winner board)
        []
        (map
          (partial mark board)
          (empty-cells board))))


<!SLIDE small>
# Game tree

    (defn game-tree [board make-move]
      {:node board
       :children (map
                   #(game-tree % make-move)
                   (make-move board))})

<!SLIDE >
# Static evaluation

    (defn evaluate-static-position [position]
      (case (winner position)
        :x 1
        :o -1
        0))

<!SLIDE small>
# Dynamic evaluation

    (defn evaluator [static-evaluator]
      (fn [tree]
        (minimize static-evaluator tree)))

    (defn best-move [tree static-evaluator]
      (:node (apply max-key
                    (evaluator static-evaluator)
                    (:children tree))))

<!SLIDE small>
# Dynamic evaluation

    (defn maximize [evaluator tree]
      (if (seq (:children tree))
        (apply max
               (map #(minimize evaluator %)
                    (:children tree)))
        (evaluator (:node tree))))

    (defn minimize [evaluator tree]
      (if (seq (:children tree))
        (apply min
               (map #(maximize evaluator %)
                    (:children tree)))
        (evaluator (:node tree))))

<!SLIDE bullets>
# What we got

* No state at all, no assignment
* Damn easy to test
* Modular

<!SLIDE bullets>
# Modular

* It plays any game of alternating turns
* A small set of functions describe the particular game
* Lazy evaluated
* Easy to prune the tree
* Easy parallelization?

<!SLIDE title-slide>
# And finally

<!SLIDE title-slide>
# Just to leave you wanting more

<!SLIDE >
# Macros
## Maybe the most important Lisp feature

<!SLIDE smbullets>
# Macros

* Run your code at compile time
* Get code as input
* Return code as output
* It's all data!

<!SLIDE code>

    (def initial-board
      (board - - -
             - - -
             - - -))

    (def mid-game
      (board - x -
             o x -
             - o x))

    (def game-end
      (board x o o
             x x o
             o x x))

<!SLIDE code>

    (defmacro board [& cells]
      (let [marks (map vector cells all-cells)
            x (map
                second
                (filter #(= 'x (first %)) marks))
            o (map
                second
                (filter #(= 'o (first %)) marks))]
        `(make-board (vector ~@x) (vector ~@o))))

<!SLIDE code>

    (defmacro mark# [board sym]
      `(mark ~board ~(symbol->cell sym)))
     
    (mark# ne (board - - -
                     - x -
                     - - o))


<!SLIDE smaller>
# When was the last time you saw tests like these

    (deftest play-test
      (is (= (best-move (board - x x
                               - o -
                               - - o))
             (board x x x
                    - o -
                    - - o)))

      (is (= (best-move (board - - o
                               - x o
                               x - -))
             (board - - o
                    - x o
                    x - x))))

<!SLIDE >
# Thank you

<!SLIDE smbullets>
# Where to go for more

* SICP: Structure & Interpretation of Computer Programs (Abelson/Sussman)
* Why functional programming matters, John Hughes
* Programming Clojure, Stuart Halloway
* The Joy of Clojure, Stuart Halloway, Fogus & Houser
* \#clojure @ freenode
* https://github.com/paraseba/tictactoe

