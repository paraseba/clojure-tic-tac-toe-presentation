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
* 3 years old
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

* Inmutables
* Se opera sobre ellas con funciones
* Garantías de performance
* Las copias mantienen las garantías
* Es lo que se viene

<!SLIDE>
# Operaciones

    (conj '(1 2 3) 0)      ->  (0 1 2 3)
    (conj [1 2 3] 4)       ->  (1 2 3 4)
    (assoc {1 2} 3 4 1 5)  -> {3 4 1 5}

<!SLIDE>
# "Variables"

    (let [x 1 y 2]
      (+ x y))

<!SLIDE>
# Keywords

    (= :foo :foo)

    (let [m {:a 1, :b 2}]
      (m :a)
      (:a m))


<!SLIDE smbullets>
# Secuencias

* first, rest, cons
* Lazy
* Usadas extensivamente en las funciones core de Clojure

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

<!SLIDE title-slide>
# Cómo modelar sin estado?
## Como sea, pero hay que aprender

<!SLIDE smbullets>

* Valores en vez de estado
* Mapas en vez de objetos
* Todo inmutable
* Las acciones devuelven un nuevo valor
* Las funciones son puras

<!SLIDE title-slide>
# La idea de identidad cambia

<!SLIDE bullets>

* En OOP identidad = espacio de memoria
* Identidad = puntero
* Los "valores" de una identidad cambian


<!SLIDE bullets>
# En Clojure

* Mecanismo de manejo de identidad
* Representación del tiempo y el cambio
* Coordinación de cambio

<!SLIDE>
# Tres tipos de identidades

###Dependiendo del tipo de cambio deseado

<table style="width:70%; margin: auto auto; border:solid 2px; text-align: center;" >
  <tr>
    <th>Reference</td>
    <th>Coordinado</td>
    <th>Sincrónico</td>
  </tr>

  <tr>
    <td>ref</td>
    <td>Si</td>
    <td>Si</td>
  </tr>

  <tr>
    <td>atom</td>
    <td>No</td>
    <td>Si</td>
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
## ta-te-ti

<!SLIDE small>
# El tablero

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
# El tablero

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
# Ganador

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
# Cómo jugar

* Arranco con una posición
* Genero el árbol de todas las jugadas posibles
* Selecciono la mejor asumiendo que el otro juega su mejor

<!SLIDE>
# Jugadas posibles

    (defn plays [board]
      (if (winner board)
        []
        (map
          (partial mark board)
          (empty-cells board))))


<!SLIDE small>
# Árbol de jugadas

    (defn game-tree [board generator]
      {:node board
       :children (map
                   #(game-tree % generator)
                   (generator board))})

<!SLIDE >
# Evaluación estática

    (defn evaluate-static-position [position]
      (case (winner position)
        :x 1
        :o -1
        0))

<!SLIDE small>
# Evaluación dinámica

    (defn evaluate [tree]
      (minimize tree))

    (defn best-play [position]
      (let [tree (game-tree position plays)
            children (:children tree)]
        (:node (apply max-key evaluate children))))

<!SLIDE small>
# Evaluación dinámica

    (defn maximize [tree]
      (if (seq (:children tree))
        (apply max (map minimize (:children tree)))
        (evaluate-static-position (:node tree))))

    (defn minimize [tree]
      (if (seq (:children tree))
        (apply min (map maximize (:children tree)))
        (evaluate-static-position (:node tree))))

<!SLIDE bullets>
# Qué conseguimos

* No hay estado, no hay assignment
* Fácil de testear
* Modular

<!SLIDE bullets>
# Modular

* Sirve para cualquier juego
* El juego está representado por plays y winner
* Lazy
* Prune

<!SLIDE >
# Macros
## La característica más importante de Lisp

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

    (def initial-board
      (board - - -
             - - -
             - - -))

    (def game-end
      (board x o o
             x x o
             o x x))

<!SLIDE code>

    (def symbol->cell
      {'n 1 's 7 'e 5
       'w 3 'c 4 'o 4
       'ne 2 'nw 0 'se 8 'sw 6})

    (defmacro mark# [board sym]
      `(mark ~board ~(symbol->cell sym)))
     
    (mark# ne (board - - -
                     - x -
                     - - o))


<!SLIDE smaller>

    (deftest play-test
      (is (= (best-play (board - x x
                               - o -
                               - - o))
             (board x x x
                    - o -
                    - - o)))

      (is (= (best-play (board - - o
                               - x o
                               x - -))
             (board - - o
                    - x o
                    x - x))))

<!SLIDE >
# Eso es todo

<!SLIDE smbullets>
# Bibliografía

* SICP: Structure & Interpretation of Computer Programs (Abelson/Sussman)
* Why functional programming matters, John Hughes
* Programming Clojure, Stuart Halloway
* The Joy of Clojure, Stuart Halloway, Fogus & Houser
* \#clojure @ freenode

