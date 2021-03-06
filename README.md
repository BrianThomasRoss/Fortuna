# Fortuna: Generative Modeling Toolkit for Python3
© 2020 Robert Sharp, all rights reserved.

Fortuna's main goal is to provide a quick and easy way to build custom random 
functions for your data that are byte-code fast. Fortuna also offers a variety 
of high-performance random number functions like `random_range()` and `dice()`.

The core functionality of Fortuna is based on the Storm RNG Engine - created by 
the same developer (Robert Sharp). While Storm is a high quality, hardware 
seeded random engine - it is not appropriate for cryptography of any kind. 
Fortuna is meant for games, data science, A.I. and experimental programming... 
not security!

- Storm: Core C++ Random Number Engine. https://github.com/BrokenShell/Storm


### Quick Install `$ pip install Fortuna`


### Installation may require the following:
- MacOS or Linux. Windows is not directly supported at this time.
- Python 3.6 or later with dev tools (setuptools, pip, etc.), 64bit preferred.
- Cython: Python library. `$ pip install Cython`. Serves as a bridge from C/C++ to Python.
- Modern C++17 Compiler and Standard Library. Typically not required on MacOS.
    - If you're running an older version of MacOS you can simply install Xcode to get Clang.
    - For Linux this is typically GCC and it comes pre-installed on many systems, Clang should also work.

### Sister Projects:
- RNG: Python3 API for the C++ Random Library. https://pypi.org/project/RNG/
- Pyewacket: Drop-in replacement for Python3 random module. https://pypi.org/project/Pyewacket/
- MonkeyScope: Framework for testing non-deterministic functors. https://pypi.org/project/MonkeyScope/

> In an effort to streamline Fortuna, the above packages are no longer included automatically. Each of them can be installed with pip as needed.

---

### Table of Contents:
- Numeric Limits
- Random Value Classes
    - `RandomValue(Iterable) -> Callable -> Value`
    - `TruffleShuffle(Iterable) -> Callable -> Value`
    - `QuantumMonty(Iterable) -> Callable -> Value`
    - `CumulativeWeightedChoice(Iterable[Tuple[int, Any]]) -> Callable -> Value`
    - `RelativeWeightedChoice(Iterable[Tuple[int, Any]]) -> Callable -> Value`
    - `FlexCat(Dict[str, Iterable[Any]]) -> Callable -> Value`
- Random Value Functions
    - `random_value(data: Sequence[Any]) -> Any`
    - `cumulative_weighted_choice(Sequence[Tuple[int, Any]]) -> Any`
    - `truffle_shuffle(data: Iterable[Any]) -> Callable -> Any`
- Random Integer Functions
    - `random_below(Integer) -> Integer`
    - `random_int(Integer, Integer) -> Integer`
    - `random_range(Integer, Integer, Integer) -> Integer`
    - `d(Integer) -> Integer`
    - `dice(Integer, Integer) -> Integer`
    - `plus_or_minus(Integer) -> Integer`
    - `plus_or_minus_linear(Integer) -> Integer`
    - `plus_or_minus_gauss(Integer) -> Integer`
- Random Index Functions 
    - ZeroCool Specification: `f(N) -> [0, N)` or `f(-N) -> [-N, 0)`
    - `random_index(Integer) -> Integer`
    - `front_gauss(Integer) -> Integer`
    - `middle_gauss(Integer) -> Integer`
    - `back_gauss(Integer) -> Integer`
    - `quantum_gauss(Integer) -> Integer`
    - `front_poisson(Integer) -> Integer`
    - `middle_poisson(Integer) -> Integer`
    - `back_poisson(Integer) -> Integer`
    - `quantum_poisson(Integer) -> Integer`
    - `front_linear(Integer) -> Integer`
    - `middle_linear(Integer) -> Integer`
    - `back_linear(Integer) -> Integer`
    - `quantum_linear(Integer) -> Integer`
    - `quantum_monty(Integer) -> Integer`
- Random Float Functions
    - `canonical() -> Float`
    - `random_float(Float, Float) -> Float`
    - `triangular(Float, Float, Float) -> Float`
- Random Boolean Functions
    - `percent_true(Float) -> Boolean`
- Inplace Shuffle Algorithms
    - `shuffle(List[Any]) -> None`
    - `knuth_a(List[Any]) -> None`
    - `fisher_yates(List[Any]) -> None`
- Utilities
    - `flatten(Object, *args, Boolean, **kwargs) -> Object`
    - `smart_clamp(Integer, Integer, Integer) -> Integer`
- Experimental
    - `MultiChoice(str, Iterable[str], str, bool, str) -> Callable -> str`
- Development Log
- Test Suite Output
- Legal Information


#### Numeric Limits:
- Integer: 64 bit signed integer.
    - Range: ±9223372036854775807, approximately ±9.2 billion billion
- Float: 64 bit floating point.
    - Range: ±1.7976931348623157e+308
    - Epsilon Delta: 5e-305 to 5e-324, platform dependent

----

## Random Value Engines

### Fortuna.RandomValue
`Fortuna.RandomValue(collection: Iterable[Any], zero_cool=random_index, flat=True) -> Callable -> Value`
Random Value Engine Class that supports dependency injection.

Takes an iterable (tuple) and a distribution function and returns
a callable to produce random values from the iterable with the same distribution.

- @param collection :: Iterable of Values. Tuple recommended.
- @param zero_cool :: Optional ZeroCool Callable, kwarg only. Default = random_index(). This function must follow the ZeroCool Spec.
- @param flat :: Bool. Default: True. Option to automatically flatten callable values with lazy evaluation.
- @return :: Callable Object. `Callable(*args, **kwargs) -> Value`
    - @param *args, **kwargs :: Optional arguments used to flatten the return Value (below) if Callable.
    - @return Value or Value(*args, **kwargs) if the value itself is callable. This is recursive.

#### RandomValue Dependency Injection: Rare Apples Example
RandomValue supports dependency injection, it is the only Fortuna class to do so.
The injected functor must follow the ZeroCool Specification:
f(x: int) -> int in [0, x) with any distribution. Many ZeroCool functions are provided,
in the following example we'll see front_linear and back_linear used together.

> Typically if one of the builtin ZeroCool functions is required, it is 
highly recommended to employ QuantumMonty rather than RandomValue. 
They're used here for demonstration purposes only. QuantumMonty offers 
the same ZeroCool functions, but as methods with less overhead. RandomValue is 
specifically designed for custom dependency injection, where you might design 
your own distribution as use it with RandomValue.

```python
from Fortuna import RandomValue, front_linear, back_linear


# Data Setup
random_apple = RandomValue((
    "Delicious", 
    "Empire", 
    "Granny Smith", 
    "Honey Crisp", 
    "Macintosh",
), zero_cool=front_linear)

random_fruit = RandomValue((
    lambda: f"Apple, {random_apple()}",
    "Banana",
    "Cherry",
    "Grapes",
    "Orange",
), zero_cool=back_linear)

# Usage
print(random_fruit())
# prints a random fruit with the correct compound distribution

```
#### QuantumMonty: Rare Apples Example
Same as above but with QuantumMonty - for syntax comparison.
```python
from Fortuna import QuantumMonty


# Data Setup
random_apple = QuantumMonty((
    "Delicious", 
    "Empire", 
    "Granny Smith", 
    "Honey Crisp", 
    "Macintosh",
)).front_linear

random_fruit = QuantumMonty((
    lambda: f"Apple, {random_apple()}",
    "Banana",
    "Cherry",
    "Grapes",
    "Orange",
)).back_linear

# Usage
print(random_fruit())
# prints a random fruit with the correct compound distribution

```

### Auto Flattening
#### RandomValue with Auto Flattening
Auto Flattening works with all random generator classes in Fortuna, and it's on by default.
Flattening is lazy: it happens at call time as the last step. 
Flattening is recursive: this allows a nested lambda stack to be collapsed automatically.
Flattening resilient: if for any reason a callable can not be flatted - it will 
be returned in an un-flattened state without error. 
```python
from Fortuna import RandomValue


auto_flat = RandomValue([lambda: 1, lambda: 2, lambda: 3])
print(auto_flat())  # will print the value 1, 2 or 3.
# Note: the lambda will not be called until call time and stays dynamic for the life of the object.

auto_flat_with = RandomValue([lambda x: x, lambda x: x + 1, lambda x:  x + 2])
print(auto_flat_with(2))  # will print the value 2, 3 or 4
# Note: if this is called with no args it will simply return the lambda in an uncalled state.

un_flat = RandomValue([lambda: 1, lambda: 2, lambda: 3], flat=False)
print(un_flat()())  # will print the value 1, 2 or 3, 
# mind the double-double parenthesis, they are required to manually unpack the lambdas

auto_un_flat = RandomValue([lambda x: x, lambda x: x + 1, lambda x:  x + 2], flat=False)
# Note: flat=False is not required here because the lambdas can not be called without input x satisfied.
# It is recommended to specify flat=False if non-flat output is intended.
print(auto_un_flat()(1))  # will print the value 1, 2 or 3, mind the double-double parenthesis.
```

#### Mixing Static Objects with Callable Objects
Auto Flattening works with all random generator classes in Fortuna, and it's on by default.
```python
from Fortuna import RandomValue


""" With automatic flattening active, `lambda() -> int` can be treated as an `int`. """
mixed_flat = RandomValue([1, 2, lambda: 3])
print(mixed_flat())  # will print 1, 2 or 3

""" Mixed Anti-pattern """
mixed_un_flat = RandomValue([1, 2, lambda: 3], flat=False) # this is not recommended.
print(mixed_flat())  # will print 1, 2 or "Function <lambda at some_address>"
# This pattern is not recommended because you wont know the nature of what you get back.
# This is almost always not what you want, and it can give rise to messy logic in other areas of your code.
```

#### Dynamic Strings
To successfully express a dynamic string, and keep it dynamic for the duration of the program, at least one level of indirection is required. Without a lambda - the f-string would collapse into a static string too soon.
This works with all random generator classes in Fortuna.

```python
from Fortuna import RandomValue, d


# d() is a simple dice function, d(n) -> [1, n] flat uniform distribution.
dynamic_string = RandomValue((
    # while the probability of all A == all B == all C, individual probabilities of each possible string will differ based on the number of possible outputs of each category.
    lambda: f"A{d(2)}",  # -> A1 - A2, each are twice as likely as any particular B, and three times as likely as any C.
    lambda: f"B{d(4)}",  # -> B1 - B4, each are half as likely as any particular A, and 3/2 as likely as any C.
    lambda: f"C{d(6)}",  # -> C1 - C6, each are 1/3 as likely as any particular A and 2/3 as likely of any B.
))

print(dynamic_string())  # prints a random dynamic string, generated at call time.
```

### TruffleShuffle
`Fortuna.TruffleShuffle(collection: Iterable[Any], flat=True) -> Callable -> Value`
- @param collection :: Iterable of Values. Set recommended but not required.
- @param flat :: Bool. Default: True. Option to automatically flatten callable values with lazy evaluation.
- @return :: Callable Object. `Callable(*args, **kwargs) -> Value`
    - @param *args, **kwargs :: Optional arguments used to flatten the return Value (below) if Callable.
    - @return :: Random value from the collection with a Wide Uniform Distribution. 
    
Wide Uniform Distribution: "Wide" refers to the average distance between consecutive occurrences of the same value. The average width of the output distribution will naturally scale up with the size of the collection. The goal of this type of distribution is to keep the output sequence free of clumps or streaks of the same value, while maintaining randomness and uniform probability. This is not the same as a flat uniform distribution. The two distributions over time will be statistically similar for any given set, but the repetitiveness of the output sequence will be very different.

#### TruffleShuffle, Basic Use
```python
from Fortuna import TruffleShuffle

# Data Setup
list_of_values = { 1, 2, 3, 4, 5, 6 }
truffle_shuffle = TruffleShuffle(list_of_values)

# Usage
print(truffle_shuffle())  # this will print one of the numbers 1-6, 
# repeated calls will produce a wide distribution.
```

#### Nesting Dolls
This works with all random generator classes in Fortuna, interchangeably.
```python
from Fortuna import RandomValue, TruffleShuffle

# Data Setup
nesting_dolls = TruffleShuffle((
    RandomValue(("A", "B", "C", "D", "E")),
    RandomValue(("F", "G", "H", "I", "J")),
    RandomValue(("K", "L", "M", "N", "O")),
    RandomValue(("P", "Q", "R", "S", "T")),
))

# Usage
print(nesting_dolls())  
# Prints one of the letters A-T.
# Produces a wide distribution of each category -
# and a flat uniform distribution within each category.
```

### QuantumMonty
`Fortuna.QuantumMonty(collection: Iterable[Any], flat=True) -> Callable -> Value`
- @param collection :: Iterable of Values. Tuple recommended.
- @param flat :: Bool. Default: True. Option to automatically flatten callable values with lazy evaluation.
- @return :: Callable Object with Monty Methods for producing various distributions of the data.
    - @param *args, **kwargs :: Optional arguments used to flatten the return Value (below) if Callable.
    - @return :: Random value from the data. The instance will produce random values from the list using the selected distribution model or "monty". The default monty is the Quantum Monty Algorithm.

```python
from Fortuna import QuantumMonty

# Data Setup
list_of_values = [1, 2, 3, 4, 5, 6]
monty = QuantumMonty(list_of_values)

# Usage
print(monty())               # prints a random value from the list_of_values.
                             # uses the default Quantum Monty Algorithm.

print(monty.flat_uniform())  # prints a random value from the list_of_values.
                             # uses the "flat_uniform" monty.
                             # equivalent to random.choice(list_of_values).
```
The QuantumMonty class represents a diverse collection of strategies for producing random values from a sequence where the output distribution is based on the method you choose. Generally speaking, each value in the sequence will have a probability that is based on its position in the sequence. For example: the "front" monty produces random values where the beginning of the sequence is geometrically more common than the back. Given enough samples the "front" monty will always converge to a 45 degree slope down for any list of unique values.

There are three primary method families: linear, gaussian, and poisson. Each family has three base methods; 'front', 'middle', 'back', plus a 'quantum' method that incorporates all three base methods. The quantum algorithms for each family produce distributions by overlapping the probability waves of the other methods in their family. The Quantum Monty Algorithm incorporates all nine base methods.

```python
import Fortuna

# Data Setup
monty = Fortuna.QuantumMonty(
    ["Alpha", "Beta", "Delta", "Eta", "Gamma", "Kappa", "Zeta"]
)

# Usage
# Each of the following methods will return a random value from the sequence.
# Each method has its own unique distribution model.
""" Flat Base Case """
monty.flat_uniform()        # Flat Uniform Distribution
""" Geometric Positional """
monty.front_linear()        # Linear Descending, Triangle
monty.middle_linear()       # Linear Median Peak, Equilateral Triangle
monty.back_linear()         # Linear Ascending, Triangle
monty.quantum_linear()      # Linear Overlay, 3-way monty.
""" Gaussian Positional """
monty.front_gauss()         # Front Gamma
monty.middle_gauss()        # Scaled Gaussian
monty.back_gauss()          # Reversed Gamma
monty.quantum_gauss()       # Gaussian Overlay, 3-way monty.
""" Poisson Positional """
monty.front_poisson()       # 1/4 Mean Poisson
monty.middle_poisson()      # 1/2 Mean Poisson
monty.back_poisson()        # 3/4 Mean Poisson
monty.quantum_poisson()     # Poisson Overlay, 3-way monty.
""" Quantum Monty Algorithm """
monty()                     # Quantum Monty Algorithm, 9-way monty.
monty.quantum_monty()       #  same as above
```

### Weighted Choice: Base Class
Weighted Choice offers two strategies for selecting random values from a sequence where programmable rarity is desired. Both produce a custom distribution of values based on the weights of the values.

The choice to use one strategy over the other is purely about which one suits you or your data best. Relative weights are easier to understand at a glance. However, many RPG Treasure Tables map rather nicely to a cumulative weighted strategy.

#### Cumulative Weighted Choice
`Fortuna.CumulativeWeightedChoice(weighted_table: Iterable[Tuple[int, Any]], flat=True) -> Callable -> Value`
- @param weighted_table :: Table of weighted pairs. Tuple of Tuples recommended.
- @param flat :: Bool. Default: True. Option to automatically flatten callable values with lazy evaluation.
- @return :: Callable Instance
    - @param *args, **kwargs :: Optional arguments used to flatten the return Value (below) if Callable.
    - @return :: Random value from the weighted_table, distribution based on the weights of the values.

_Note: Logic dictates Cumulative Weights must be unique!_

```python
from Fortuna import CumulativeWeightedChoice

# Data Setup
cum_weighted_choice = CumulativeWeightedChoice((
    (7, "Apple"),
    (11, "Banana"),
    (13, "Cherry"),
    (23, "Grape"),
    (26, "Lime"),
    (30, "Orange"),  # same as relative weight 4 because 30 - 26 = 4
))
# Usage
print(cum_weighted_choice())  # prints a weighted random value
```

#### Relative Weighted Choice
`Fortuna.RelativeWeightedChoice(weighted_table: Iterable[Tuple[int, Any]]) -> Callable -> Value`
- @param weighted_table :: Table of weighted pairs. Tuple of Tuples recommended.
- @param flat :: Bool. Default: True. Option to automatically flatten callable values with lazy evaluation.
- @return :: Callable Instance
    - @param *args, **kwargs :: Optional arguments used to flatten the return Value (below) if Callable.
    - @return :: Random value from the weighted_table, distribution based on the weights of the values.

```python
from Fortuna import RelativeWeightedChoice

# Data
population = ["Apple", "Banana", "Cherry", "Grape", "Lime", "Orange"]
rel_weights = [7, 4, 2, 10, 3, 4]

# Setup
rel_weighted_choice = RelativeWeightedChoice(zip(rel_weights, population))

# Usage
print(rel_weighted_choice())  # prints a weighted random value
```

### FlexCat
`Fortuna.FlexCat(matrix_data: Matrix, key_bias="front_linear", val_bias="truffle_shuffle", flat=True) -> Callable -> Value`
- @param matrix_data :: Dictionary of Sequences. `Dict[str, Iterable[Any]]`
- @parm key_bias :: Default is "front_linear". String indicating the name of the algorithm to use for random key selection.
- @parm val_bias :: Default is "truffle_shuffle". String indicating the name of the algorithm to use for random value selection.
- @param flat :: Bool. Default is True. Option to automatically flatten callable values with lazy evaluation.
- @return :: Callable Instance
    - @param cat_key :: Optional String. Default is None. Key selection by name. If specified, this will override the key_bias for a single call.
    - @param *args, **kwargs :: Optional arguments used to flatten the return Value (below) if Callable.
    - @return :: Value. Returns a random value generated with val_bias from a random sequence generated with key_bias.

FlexCat is like a multi dimensional QuantumMonty.

The constructor takes two optional keyword arguments to specify the algorithms to be used to make random selections. The algorithm specified for selecting a key need not be the same as the one for selecting values. An optional key may be provided at call time to bypass the random key selection. Keys passed in this way must exactly match a key in the Matrix.

By default, FlexCat will use key_bias="front_linear" and val_bias="truffle_shuffle", this will make the top of the data structure geometrically more common than the bottom and it will truffle shuffle the sequence values. This config is known as TopCat, it produces a descending-step, micro-shuffled distribution sequence. Many other combinations are available.

Algorithmic Options: _See QuantumMonty & TruffleShuffle for more details._
- "front_linear", Linear Descending
- "middle_linear", Linear Median Peak
- "back_linear", Linear Ascending
- "quantum_linear", Linear 3-way monty
- "front_gauss", Gamma Descending
- "middle_gauss", Scaled Gaussian
- "back_gauss", Gamma Ascending
- "quantum_gauss", Gaussian 3-way monty
- "front_poisson", Front 1/3 Mean Poisson
- "middle_poisson", Middle Mean Poisson
- "back_poisson", Back 1/3 Mean Poisson
- "quantum_poisson", Poisson 3-way monty
- "quantum_monty", Quantum Monty Algorithm, 9-way monty
- "flat_uniform", uniform flat distribution
- "truffle_shuffle", TruffleShuffle wide uniform distribution

```python
from Fortuna import FlexCat, d


#                           |- Collection Generator, does not require lambda.
# Data                      |
matrix_data = {#            $                         |- Dynamic Value Expression
    "Cat_A": (f"A{i}" for i in range(1, 6)),  #       |  Lazy, 1 of 4 possibilities
    "Cat_B": ("B1", "B2", "B3", "B4", "B5"),  #       $  lambda required for dynamic eval
    "Cat_C": ("C1", "C2", "C3", f"C4.{d(2)}", lambda: f"C5.{d(4)}"),
}#   $       $       $              $                        $
#    |       |       |- Value       |                        |- Fair die method: d4
#    |       |                      |
#    |       |- Collection          |- Static Value Expression
#    |                              |  Eager, 1 or 2 permanently
#    |- Collection Key, "cat_key"

#                               |- Collection Algorithm     |- Value Algorithm
# Setup                         $  y-axis                   $  x-axis
flex_cat = FlexCat(matrix_data, key_bias="front_linear", val_bias="flat_uniform")
#    $       $       $
#    |       |       |- Dictionary of Collections
#    |       |
#    |       |- FlexCat Constructor
#    |       
#    |- Callable Random Value Generator

# Usage
flex_cat()  # returns a Value from the Matrix.
flex_cat(cat_key="Cat_B")  # returns a Value specifically from the "Cat_B" Collection.
```
### Random Value Functions
`Fortuna.random_value(Sequence[Any]) -> Any`

Essentially the same as Random.choice()
- @param data :: Sequence of Values
- @return :: Random value from the sequence. Flat uniform distribution.

`Fortuna.cumulative_weighted_choice(weighted_table: Sequence[Tuple[int, Any]]) -> Any`

Similar to Random.choices()
- @param weighted_table :: Sequence of weighted value pairs. `[(w1, v1), (w2, v2)...]`
- @return :: Returns a random value. Distribution depends on weights.

### Random Integer Functions
`Fortuna.random_below(limit: int) -> int`
- @param limit :: Any Integer
- @return :: Returns a random integer in the range...
    - `random_below(N) -> [0, N)` for positive limit.
    - `random_below(N) -> (N, 0]` for negative limit.
    - `random_below(0) -> 0` Always returns zero when limit is zero
- Flat uniform distribution.


`Fortuna.random_int(left_limit: int, right_limit: int) -> int`

Essentially the same as Random.randint()
- @param left_limit :: Any Integer
- @param right_limit :: Any Integer
- @return :: Returns a random integer in the range [left_limit, right_limit]
    - `random_int(1, 10) -> [1, 10]`
    - `random_int(10, 1) -> [1, 10]` same as above.
    - `random_int(A, B)` Always returns A when A == B
- Flat uniform distribution.


`Fortuna.random_range(start: int, stop: int = 0, step: int = 1) -> int`

Essentially the same as Random.randrange()
- @param start :: Required starting point.
    - `random_range(0) -> [0]`
    - `random_range(10) -> [0, 10)` from 0 to 9. Same as `Fortuna.random_index(N)`
    - `random_range(-10) -> [-10, 0)` from -10 to -1. Same as `Fortuna.random_index(-N)`
- @param stop :: Zero by default. Optional range bound. With at least two arguments, the order of the first two does not matter.
    - `random_range(0, 0) -> [0]`
    - `random_range(0, 10) -> [0, 10)` from 0 to 9.
    - `random_range(10, 0) -> [0, 10)` same as above.
- @param step :: One by default. Optional step size.
    - `random_range(0, 0, 0) -> [0]`
    - `random_range(0, 10, 2) -> [0, 10) by 2` even numbers from 0 to 8.
    - The sign of the step parameter controls the phase of the output. Negative stepping will flip the inclusively.
    - `random_range(0, 10, -1) -> (0, 10]` starts at 10 and ranges down to 1.
    - `random_range(10, 0, -1) -> (0, 10]` same as above.
    - `random_range(10, 10, 0) -> [10]` step size or range size of zero always returns the first parameter.
- @return :: Returns a random integer in the range [A, B) by increments of C.
- Flat uniform distribution.


`Fortuna.d(sides: int) -> int`

Represents a single roll of a given size die.
- @param sides :: Represents the size or number of sides, most commonly six.
- @return :: Returns a random integer in the range [1, sides].
- Flat uniform distribution.


`Fortuna.dice(rolls: int, sides: int) -> int`

Represents the sum total of multiple rolls of the same size die.
- @param rolls :: Represents the number of times to roll the die.
- @param sides :: Represents the die size or number of sides, most commonly six.
- @return :: Returns a random integer in range [X, Y] where X = rolls and Y = rolls * sides.
- Geometric distribution based on the number and size of the dice rolled.
- Complexity scales primarily with the number of rolls, not the size of the dice.


`Fortuna.plus_or_minus(number: int) -> int`
- @param number :: input to determine the output distribution range.
- @return :: Returns a random integer in range [-number, number].
- Flat uniform distribution.


`Fortuna.plus_or_minus_linear(number: int) -> int`
- @param number :: input to determine the output distribution range.
- @return :: Returns a random integer in range [-number, number].
- Linear geometric, 45 degree triangle distribution centered on zero.


`Fortuna.plus_or_minus_gauss(number: int) -> int`
- @param number :: input to determine the output distribution range.
- @return :: Returns a random integer in range [-number, number].
- Stretched gaussian distribution centered on zero.


### Random Index, ZeroCool Specification
ZeroCool Methods are used to generate random Sequence indices.

ZeroCool methods must have the following properties:
- Any distribution model is acceptable such that...
- The method or function must take exactly one Integer parameter N.
- The method returns a random int in range `[0, N)` for positive values of N.
- The method returns a random int in range `[N, 0)` for negative values of N.
- This symmetry matches how python can index a list from the back for negative values or the front for positive values of N.


```python
from Fortuna import random_index


some_list = [i for i in range(100)] # [0..99]

print(some_list[random_index(10)])  # prints one of the first 10 items of some_list, [0, 9]
print(some_list[random_index(-10)])  # prints one of the last 10 items of some_list, [90, 99]
```
### ZeroCool Methods
- `Fortuna.random_index(size: int) -> int` Flat uniform distribution
- `Fortuna.front_gauss(size: int) -> int` Gamma Distribution: Front Peak
- `Fortuna.middle_gauss(size: int) -> int` Stretched Gaussian Distribution: Median Peak
- `Fortuna.back_gauss(size: int) -> int` Gamma Distribution: Back Peak
- `Fortuna.quantum_gauss(size: int) -> int` Quantum Gaussian: Three-way Monty
- `Fortuna.front_poisson(size: int) -> int` Poisson Distribution: Front 1/3 Peak
- `Fortuna.middle_poisson(size: int) -> int` Poisson Distribution: Middle Peak
- `Fortuna.back_poisson(size: int) -> int` Poisson Distribution: Back 1/3 Peak
- `Fortuna.quantum_poisson(size: int) -> int` Quantum Poisson: Three-way Monty
- `Fortuna.front_geometric(size: int) -> int` Linear Geometric: 45 Degree Front Peak
- `Fortuna.middle_geometric(size: int) -> int` Linear Geometric: 45 Degree Middle Peak
- `Fortuna.back_geometric(size: int) -> int` Linear Geometric: 45 Degree Back Peak
- `Fortuna.quantum_geometric(size: int) -> int` Quantum Geometric: Three-way Monty
- `Fortuna.quantum_monty(size: int) -> int` Quantum Monty: Nine-way Monty

```python
from Fortuna import front_gauss, middle_gauss, back_gauss, quantum_gauss


some_list = [i for i in range(100)]

# Each of the following prints one of the first 10 items of some_list with the appropriate distribution
print(some_list[front_gauss(10)])
print(some_list[middle_gauss(10)])
print(some_list[back_gauss(10)])
print(some_list[quantum_gauss(10)])

# Each of the following prints one of the last 10 items of some_list with the appropriate distribution
print(some_list[front_gauss(-10)])  
print(some_list[middle_gauss(-10)])  
print(some_list[back_gauss(-10)])  
print(some_list[quantum_gauss(-10)])
```

### Random Float Functions
`Fortuna.canonical() -> float`
- @return :: random float in range [0.0, 1.0), flat uniform.

`Fortuna.random_float(a: Float, b: Float) -> Float`
- @param a :: Float input
- @param b :: Float input
- @return :: random Float in range `[a, b)`, flat uniform distribution.

`Fortuna.triangular(low: Float, high: Float, mode: Float) -> Float`
- @param low :: Float, minimum output
- @param high :: Float, maximum output
- @param mode :: Float, most common output, mode must be in range `[low, high]`
- @return :: random number in range `[low, high]` with a linear distribution about the mode.


### Random Truth Function
`Fortuna.percent_true(truth_factor: Float = 50.0) -> bool`
- @param truth_factor :: The probability of True as a percentage. Default is 50 percent.
- @return :: Produces True or False based on the truth_factor as a percent of true.
    - Always returns False if num is 0 or less
    - Always returns True if num is 100 or more.


### Shuffle Algorithm
`Fortuna.shuffle(array: List[Any]) -> None`
- Knuth B Shuffle Algorithm. Destructive, in-place shuffle.
- @param array :: List to be shuffled.
- @return :: None

`Fortuna.knuth_a(array: List[Any]) -> None`
- Knuth A Shuffle Algorithm. Destructive, in-place shuffle.
- @param array :: List to be shuffled.
- @return :: None

`Fortuna.fisher_yates(array: List[Any]) -> None`
- Fisher Yates Shuffle Algorithm. Destructive, in-place shuffle.
- @param array :: List to be shuffled.
- @return :: None


### Utilities
`Fortuna.flatten(maybe_callable, *args, flat=True, **kwargs) -> flatten(maybe_callable(*args, **kwargs))`
- Recursively calls the input object and returns the result. The arguments are only passed in on the first evaluation.
- If the maybe_callable is not callable it is simply returned without error. 
- Conceptually this is somewhat like collapsing the wave function. Often used as the last step in lazy evaluation.
- @param maybe_callable :: Any Object that might be callable.
- @param flat :: Boolean, default is True. Optional, keyword only. Disables flattening if flat is set to False, conceptually turns flatten into the identity function.
- @param *args, **kwargs :: Optional arguments used to flatten the maybe_callable object.
- @return :: Recursively Flattened Object.

`Fortuna.smart_clamp(target: int, lo: int, hi: int) -> int`
- Used to clamp the target in range `[lo, hi]` by saturating the bounds.
- Essentially the same as median for exactly three integers.
- @return :: Returns the middle value, input order does not matter.

`Fortuna.distribution_range(func: Callable, lo, hi)`
Higher-order function for producing arbitrary distribution ranges.
If given a function like random_below, this function will produce random values
with the same distribution but in the range lo to hi, rather than from zero to N-1.
Essentially, this turns a function like random_below(N) into random_int(A, B).

- @param func: ZeroCool random distribution, F(N) -> `[0, N-1]`
- @param lo: minimum limit
- @param hi: maximum limit
- @return: random value in range `[lo, hi]`


### Experimental
```
Fortuna.MultiChoice(
    query: str, 
    *,
    options: Iterable[str] = (), 
    default: str = "", 
    strict: bool = False, 
    cursor: str = ">>>",
) -> str
```

Generates multiple-choice questions for user input on the terminal. 
If there is no user input and `options` is not empty and there's no `default` - 
a random choice will be made from the `options`, otherwise the `default` 
will be used. If there is no user input and there are no `options` and no 
`default` - the question will be repeated. If `strict` is set to true - the 
user input string must be in the `options`, or the question will be repeated. 
Options are stored lowercase and printed title case. 
User input is not case sensitive.

- @param query: String.
    - Question for the user.
- @param options: Optional Iterable of Strings. Default=()
    - Options presented to the user as a numbered sequence.
    - The user may enter an answer as text or by number.
- @param default: Optional String. 
    - This is used if no user input is provided.
    - If no default is provided a random choice will be made.
- @param strict: Optional Bool. Default=False
    - True: Answer must be in the options tuple. Not case-sensitive.
    - False: Accepts any answer.
- @param cursor: Optional String. Default='>>>' 
    - Indicates user input field.


## Fortuna Development Log
##### Fortuna 3.18.2
- Fixes typos

##### Fortuna 3.18.1
- Install update

##### Fortuna 3.18.0
- Performance Enhancement
- Storm 3.3.4 update
- updated test output

##### Fortuna 3.17.13
- Fixes some typos

##### Fortuna 3.17.12
- Adds pyproject.toml file to ease the installation process.

##### Fortuna 3.17.11
- Documentation Update

##### Fortuna 3.17.10
- Documentation Update

##### Fortuna 3.17.9
- Typo

##### Fortuna 3.17.8
- Typo

##### Fortuna 3.17.7
- Typo

##### Fortuna 3.17.6
- Documentation update

##### Fortuna 3.17.5
- Typos

##### Fortuna 3.17.4
- Reorganized

##### Fortuna 3.17.3
- Streamlining: dropped auxiliary packages.

##### Fortuna 3.17.2
- Documentation update

##### Fortuna 3.17.1
- Update for backwards compatibility for platforms that do not support std::clamp.

##### Fortuna 3.17.0, internal
- Installation issue detected on some platforms that lack std::clamp.

##### Fortuna 3.16.9
- Documentation Update

##### Fortuna 3.16.8
- TruffleShuffle update
- Adds truffle_shuffle()

##### Fortuna 3.16.7
- Documentation Update

##### Fortuna 3.16.6
- Adds distribution_range()

##### Fortuna 3.16.5
- Documentation Update

##### Fortuna 3.16.4
- Documentation Update

##### Fortuna 3.16.3
- Major TruffleShuffle performance upgrade

##### Fortuna 3.16.2 - Internal
- Testing

##### Fortuna 3.16.1
- Documentation Update

##### Fortuna 3.16.0
- Storm 3.3.2 Update

##### Fortuna 3.15.1
- Docs updated

##### Fortuna 3.15.0
- Type Hints Clarified via Typing Module

##### Fortuna 3.14.1
- Fixed another installer bug affecting gcc.

##### Fortuna 3.14.0
- Minor TruffleShuffle Update
- Fisher Yates, and Knuth A Shuffle Algorithms added for comparison with Fortuna.shuffle()
    - Some platforms may prefer one over another. Intel favors Knuth B (Fortuna.shuffle) by more than double.

##### Fortuna 3.13.0 - Internal
- Development & Testing Environment Updated to Python 3.8
    - Python3.8 brings a 10-20% performance boost over all.
- RandomValue API redesign. Dependency Injection is now handled at instantiation rather than call time.

##### Fortuna 3.12.2
- Installer update.
- Clarified the docs for MultiChoice.

##### Fortuna 3.12.1
- MultiChoice now accepts a default.

##### Fortuna 3.12.0
- MultiChoice added

##### Fortuna 3.10.2
- Doc string update for clarity.
- Test update
- MonkeyScope Update

##### Fortuna 3.10.1
- Documentation fix, RandomValue examples are now together.

##### Fortuna 3.10.0
- Fortuna now includes both RNG and Pyewacket.
- Documentation update.

##### Fortuna 3.9.11
- Installer Update, properly installs MonkeyScope as intended.

##### Fortuna 3.9.10
- Fixed Typos

##### Fortuna 3.9.9
- Docs Update

##### Fortuna 3.9.8
- Test Update

##### Fortuna 3.9.7
- Tests for RNG and Pyewacket are now included in `fortuna_extras` package.

##### Fortuna 3.9.6
- Documentation update.

##### Fortuna 3.9.5
- Storm 3.2.2 Update.

##### Fortuna 3.9.4
- Documentation update.

##### Fortuna 3.9.3
- MonkeyScope update, 10% test suite performance improvement.

##### Fortuna 3.9.2
- Documentation update.

##### Fortuna 3.9.1
- `flatten_with` has been renamed to `flatten`. This should be non-breaking, please report any bugs.

##### Fortuna 3.9.0 - Internal
- Added many doc strings.
- Corrected many typos in Docs.
- The `flatten` function has been fully replaced by `flatten_with`. 
    - All classes that support automatic flattening can now accept arbitrary arguments at call time.
    - `flatten_with` will be renamed to `flatten` in a future release.

##### Fortuna 3.8.9
- Fixed some typos.

##### Fortuna 3.8.8
- Fortuna now supports Python notebooks, python3.6 or higher required.

##### Fortuna 3.8.7
- Storm Update

##### Fortuna 3.8.6
- Attempting to make Fortuna compatible with Python Notebooks. 

##### Fortuna 3.8.5
- Installer Config Update

##### Fortuna 3.8.4
- Installer Config Update

##### Fortuna 3.8.3
- Storm Update 3.2.0

##### Fortuna 3.8.2
- More Typo Fix

##### Fortuna 3.8.1
- Typo Fix

##### Fortuna 3.8.0
- Major API Update, several utilities have been deprecated. See MonkeyScope for replacements.
    - distribution
    - distribution_timer
    - timer

##### Fortuna 3.7.7
- Documentation Update

##### Fortuna 3.7.6
- Install script update.

##### Fortuna 3.7.5 - internal
- Storm 3.1.1 Update
- Added triangular function.

##### Fortuna 3.7.4
- Fixed: missing header in the project manifest, this may have caused building from source to fail.

##### Fortuna 3.7.3
- Storm Update

##### Fortuna 3.7.2
- Storm Update

##### Fortuna 3.7.1
- Bug fixes

##### Fortuna 3.7.0 - internal
- flatten_with() is now the default flattening algorithm for all Fortuna classes.

##### Fortuna 3.6.5
- Documentation Update
- RandomValue: New flatten-with-arguments functionality.

##### Fortuna 3.6.4
- RandomValue added for testing

##### Fortuna 3.6.3
- Developer Update

##### Fortuna 3.6.2
- Installer Script Update

##### Fortuna 3.6.1
- Documentation Update

##### Fortuna 3.6.0
- Storm Update
- Test Update
- Bug fix for random_range(), negative stepping is now working as intended. This bug was introduced in 3.5.0.
- Removed Features
    - lazy_cat(): use QuantumMonty class instead.
    - flex_cat(): use FlexCat class instead.
    - truffle_shuffle(): use TruffleShuffle class instead.

##### Fortuna 3.5.3 - internal
- Features added for testing & development
    - ActiveChoice class
    - random_rotate() function

##### Fortuna 3.5.2
- Documentation Updates

##### Fortuna 3.5.1
- Test Update

##### Fortuna 3.5.0
- Storm Update
- Minor Bug Fix: Truffle Shuffle
- Deprecated Features
    - lazy_cat(): use QuantumMonty class instead.
    - flex_cat(): use FlexCat class instead.
    - truffle_shuffle(): use TruffleShuffle class instead.

##### Fortuna 3.4.9
- Test Update

##### Fortuna 3.4.8
- Storm Update

##### Fortuna 3.4.7
- Bug fix for analytic_continuation.

##### Fortuna 3.4.6
- Docs Update

##### Fortuna 3.4.5
- Docs Update
- Range Tests Added, see extras folder.

##### Fortuna 3.4.4
- ZeroCool Algorithm Bug Fixes
- Typos Fixed

##### Fortuna 3.4.3
- Docs Update

##### Fortuna 3.4.2
- Typos Fixed

##### Fortuna 3.4.1
- Major Bug Fix: random_index()

##### Fortuna 3.4.0 - internal
- ZeroCool Poisson Algorithm Family Updated

##### Fortuna 3.3.8 - internal
- Docs Update

##### Fortuna 3.3.7
- Fixed Performance Bug: ZeroCool Linear Algorithm Family

##### Fortuna 3.3.6
- Docs Update

##### Fortuna 3.3.5
- ABI Updates
- Bug Fixes

##### Fortuna 3.3.4
- Examples Update

##### Fortuna 3.3.3
- Test Suite Update

##### Fortuna 3.3.2 - internal
- Documentation Update

##### Fortuna 3.3.1 - internal
- Minor Bug Fix

##### Fortuna 3.3.0 - internal
- Added `plus_or_minus_gauss(N: int) -> int` random int in range [-N, N] Stretched Gaussian Distribution

##### Fortuna 3.2.3
- Small Typos Fixed

##### Fortuna 3.2.2
- Documentation update.

##### Fortuna 3.2.1
- Small Typo Fixed

##### Fortuna 3.2.0
- API updates:
    - QunatumMonty.uniform -> QunatumMonty.flat_uniform
    - QunatumMonty.front -> QunatumMonty.front_linear
    - QunatumMonty.middle -> QunatumMonty.middle_linear
    - QunatumMonty.back -> QunatumMonty.back_linear
    - QunatumMonty.quantum -> QunatumMonty.quantum_linear
    - randindex -> random_index
    - randbelow -> random_below
    - randrange -> random_range
    - randint   -> random_int

##### Fortuna 3.1.0
- `discrete()` has been removed, see Weighted Choice.
- `lazy_cat()` added.
- All ZeroCool methods have been raised to top level API, for use with lazy_cat()

##### Fortuna 3.0.1
- minor typos.

##### Fortuna 3.0.0
- Storm 2 Rebuild.

##### Fortuna 2.1.1
- Small bug fixes.
- Test updates.

##### Fortuna 2.1.0, Major Feature Update
- Fortuna now includes the best of RNG and Pyewacket.

##### Fortuna 2.0.3
- Bug fix.

##### Fortuna 2.0.2
- Clarified some documentation.

##### Fortuna 2.0.1
- Fixed some typos.

##### Fortuna 2.0.0b1-10
- Total rebuild. New RNG Storm Engine.

##### Fortuna 1.26.7.1
- README updated.

##### Fortuna 1.26.7
- Small bug fix.

##### Fortuna 1.26.6
- Updated README to reflect recent changes to the test script.

##### Fortuna 1.26.5
- Fixed small bug in test script.

##### Fortuna 1.26.4
- Updated documentation for clarity.
- Fixed a minor typo in the test script.

##### Fortuna 1.26.3
- Clean build.

##### Fortuna 1.26.2
- Fixed some minor typos.

##### Fortuna 1.26.1
- Release.

##### Fortuna 1.26.0 beta 2
- Moved README and LICENSE files into fortuna_extras folder.

##### Fortuna 1.26.0 beta 1
- Dynamic version scheme implemented.
- The Fortuna Extension now requires the fortuna_extras package, previously it was optional.

##### Fortuna 1.25.4
- Fixed some minor typos in the test script.

##### Fortuna 1.25.3
- Since version 1.24 Fortuna requires Python 3.7 or higher. This patch corrects an issue where the setup script incorrectly reported requiring Python 3.6 or higher.

##### Fortuna 1.25.2
- Updated test suite.
- Major performance update for TruffleShuffle.
- Minor performance update for QuantumMonty & FlexCat: cycle monty.

##### Fortuna 1.25.1
- Important bug fix for TruffleShuffle, QuantumMonty and FlexCat.

##### Fortuna 1.25
- Full 64bit support.
- The Distribution & Performance Tests have been redesigned.
- Bloat Control: Two experimental features have been removed.
    - RandomWalk
    - CatWalk
- Bloat Control: Several utility functions have been removed from the top level API. These function remain in the Fortuna namespace for now, but may change in the future without warning.
    - stretch_bell, internal only.
    - min_max, not used anymore.
    - analytic_continuation, internal only.
    - flatten, internal only.

##### Fortuna 1.24.3
- Low level refactoring, non-breaking patch.

##### Fortuna 1.24.2
- Setup config updated to improve installation.

##### Fortuna 1.24.1
- Low level patch to avoid potential ADL issue. All low level function calls are now qualified.

##### Fortuna 1.24
- Documentation updated for even more clarity.
- Bloat Control: Two naïve utility functions that are no longer used in the module have been removed.
    - n_samples -> use a list comprehension instead. `[f(x) for _ in range(n)]`
    - bind -> use a lambda instead. `lambda: f(x)`

##### Fortuna 1.23.7
- Documentation updated for clarity.
- Minor bug fixes.
- TruffleShuffle has been redesigned slightly, it now uses a random rotate instead of swap.
- Custom `__repr__` methods have been added to each class.

##### Fortuna 1.23.6
- New method for QuantumMonty: quantum_not_monty - produces the upside down quantum_monty.
- New bias option for FlexCat: not_monty.

##### Fortuna 1.23.5.1
- Fixed some small typos.

##### Fortuna 1.23.5
- Documentation updated for clarity.
- All sequence wrappers can now accept generators as input.
- Six new functions added:
    - random_float() -> float in range [0.0..1.0) exclusive, uniform flat distribution.
    - percent_true_float(num: float) -> bool, Like percent_true but with floating point precision.
    - plus_or_minus_linear_down(num: int) -> int in range [-num..num], upside down pyramid.
    - plus_or_minus_curve_down(num: int) -> int in range [-num..num], upside down bell curve.
    - mostly_not_middle(num: int) -> int in range [0..num], upside down pyramid.
    - mostly_not_center(num: int) -> int in range [0..num], upside down bell curve.
- Two new methods for QuantumMonty:
    - mostly_not_middle
    - mostly_not_center
- Two new bias options for FlexCat, either can be used to define x and/or y axis bias:
    - not_middle
    - not_center

##### Fortuna 1.23.4.2
- Fixed some minor typos in the README.md file.

##### Fortuna 1.23.4.1
- Fixed some minor typos in the test suite.

##### Fortuna 1.23.4
- Fortuna is now Production/Stable!
- Fortuna and Fortuna Pure now use the same test suite.

##### Fortuna 0.23.4, first release candidate.
- RandomCycle, BlockCycle and TruffleShuffle have been refactored and combined into one class: TruffleShuffle.
- QuantumMonty and FlexCat will now use the new TruffleShuffle for cycling.
- Minor refactoring across the module.

##### Fortuna 0.23.3, internal
- Function shuffle(arr: list) added.

##### Fortuna 0.23.2, internal
- Simplified the plus_or_minus_curve(num: int) function, output will now always be bounded to the range [-num..num].
- Function stretched_bell(num: int) added, this matches the previous behavior of an unbounded plus_or_minus_curve.

##### Fortuna 0.23.1, internal
- Small bug fixes and general clean up.

##### Fortuna 0.23.0
- The number of test cycles in the test suite has been reduced to 10,000 (down from 100,000). The performance of the pure python implementation and the c-extension are now directly comparable.
- Minor tweaks made to the examples in `.../fortuna_extras/fortuna_examples.py`

##### Fortuna 0.22.2, experimental features
- BlockCycle class added.
- RandomWalk class added.
- CatWalk class added.

##### Fortuna 0.22.1
- Fortuna classes no longer return lists of values, this behavior has been extracted to a free function called n_samples.

##### Fortuna 0.22.0, experimental features
- Function bind added.
- Function n_samples added.

##### Fortuna 0.21.3
- Flatten will no longer raise an error if passed a callable item that it can't call. It correctly returns such items in an uncalled state without error.
- Simplified `.../fortuna_extras/fortuna_examples.py` - removed unnecessary class structure.

##### Fortuna 0.21.2
- Fix some minor bugs.

##### Fortuna 0.21.1
- Fixed a bug in `.../fortuna_extras/fortuna_examples.py`

##### Fortuna 0.21.0
- Function flatten added.
- Flatten: The Fortuna classes will recursively unpack callable objects in the data set.

##### Fortuna 0.20.10
- Documentation updated.

##### Fortuna 0.20.9
- Minor bug fixes.

##### Fortuna 0.20.8, internal
- Testing cycle for potential new features.

##### Fortuna 0.20.7
- Documentation updated for clarity.

##### Fortuna 0.20.6
- Tests updated based on recent changes.

##### Fortuna 0.20.5, internal
- Documentation updated based on recent changes.

##### Fortuna 0.20.4, internal
- WeightedChoice (both types) can optionally return a list of samples rather than just one value, control the length of the list via the n_samples argument.

##### Fortuna 0.20.3, internal
- RandomCycle can optionally return a list of samples rather than just one value,
control the length of the list via the n_samples argument.

##### Fortuna 0.20.2, internal
- QuantumMonty can optionally return a list of samples rather than just one value,
control the length of the list via the n_samples argument.

##### Fortuna 0.20.1, internal
- FlexCat can optionally return a list of samples rather than just one value,
control the length of the list via the n_samples argument.

##### Fortuna 0.20.0, internal
- FlexCat now accepts a standard dict as input. The ordered(ness) of dict is now part of the standard in Python 3.7.1. Previously FlexCat required an OrderedDict, now it accepts either and treats them the same.

##### Fortuna 0.19.7
- Fixed bug in `.../fortuna_extras/fortuna_examples.py`.

##### Fortuna 0.19.6
- Updated documentation formatting.
- Small performance tweak for QuantumMonty and FlexCat.

##### Fortuna 0.19.5
- Minor documentation update.

##### Fortuna 0.19.4
- Minor update to all classes for better debugging.

##### Fortuna 0.19.3
- Updated plus_or_minus_curve to allow unbounded output.

##### Fortuna 0.19.2
- Internal development cycle.
- Minor update to FlexCat for better debugging.

##### Fortuna 0.19.1
- Internal development cycle.

##### Fortuna 0.19.0
- Updated documentation for clarity.
- MultiCat has been removed, it is replaced by FlexCat.
- Mostly has been removed, it is replaced by QuantumMonty.

##### Fortuna 0.18.7
- Fixed some more README typos.

##### Fortuna 0.18.6
- Fixed some README typos.

##### Fortuna 0.18.5
- Updated documentation.
- Fixed another minor test bug.

##### Fortuna 0.18.4
- Updated documentation to reflect recent changes.
- Fixed some small test bugs.
- Reduced default number of test cycles to 10,000 - down from 100,000.

##### Fortuna 0.18.3
- Fixed some minor README typos.

##### Fortuna 0.18.2
- Fixed a bug with Fortuna Pure.

##### Fortuna 0.18.1
- Fixed some minor typos.
- Added tests for `.../fortuna_extras/fortuna_pure.py`

##### Fortuna 0.18.0
- Introduced new test format, now includes average call time in nanoseconds.
- Reduced default number of test cycles to 100,000 - down from 1,000,000.
- Added pure Python implementation of Fortuna: `.../fortuna_extras/fortuna_pure.py`
- Promoted several low level functions to top level.
    - `zero_flat(num: int) -> int`
    - `zero_cool(num: int) -> int`
    - `zero_extreme(num: int) -> int`
    - `max_cool(num: int) -> int`
    - `max_extreme(num: int) -> int`
    - `analytic_continuation(func: staticmethod, num: int) -> int`
    - `min_max(num: int, lo: int, hi: int) -> int`

##### Fortuna 0.17.3
- Internal development cycle.

##### Fortuna 0.17.2
- User Requested: dice() and d() functions now support negative numbers as input.

##### Fortuna 0.17.1
- Fixed some minor typos.

##### Fortuna 0.17.0
- Added QuantumMonty to replace Mostly, same default behavior with more options.
- Mostly is depreciated and may be removed in a future release.
- Added FlexCat to replace MultiCat, same default behavior with more options.
- MultiCat is depreciated and may be removed in a future release.
- Expanded the Treasure Table example in `.../fortuna_extras/fortuna_examples.py`

##### Fortuna 0.16.2
- Minor refactoring for WeightedChoice.

##### Fortuna 0.16.1
- Redesigned fortuna_examples.py to feature a dynamic random magic item generator.
- Raised cumulative_weighted_choice function to top level.
- Added test for cumulative_weighted_choice as free function.
- Updated MultiCat documentation for clarity.

##### Fortuna 0.16.0
- Pushed distribution_timer to the .pyx layer.
- Changed default number of iterations of tests to 1 million, up form 1 hundred thousand.
- Reordered tests to better match documentation.
- Added Base Case Fortuna.fast_rand_below.
- Added Base Case Fortuna.fast_d.
- Added Base Case Fortuna.fast_dice.

##### Fortuna 0.15.10
- Internal Development Cycle

##### Fortuna 0.15.9
- Added Base Cases for random_value()
- Added Base Case for randint()

##### Fortuna 0.15.8
- Clarified MultiCat Test

##### Fortuna 0.15.7
- Fixed minor typos.

##### Fortuna 0.15.6
- Fixed minor typos.
- Simplified MultiCat example.

##### Fortuna 0.15.5
- Added MultiCat test.
- Fixed some minor typos in docs.

##### Fortuna 0.15.4
- Performance optimization for both WeightedChoice() variants.
- Cython update provides small performance enhancement across the board.
- Compilation now leverages Python3 all the way down.
- MultiCat pushed to the .pyx layer for better performance.

##### Fortuna 0.15.3
- Reworked the MultiCat example to include several randomizing strategies working in concert.
- Added Multi Dice 10d10 performance tests.
- Updated sudo code in documentation to be more pythonic.

##### Fortuna 0.15.2
- Fixed: Linux installation failure.
- Added: complete source files to the distribution (.cpp .hpp .pyx).

##### Fortuna 0.15.1
- Updated & simplified distribution_timer in `fortuna_tests.py`
- Readme updated, fixed some typos.
- Known issue preventing successful installation on some linux platforms.

##### Fortuna 0.15.0
- Performance tweaks.
- Readme updated, added some details.

##### Fortuna 0.14.1
- Readme updated, fixed some typos.

##### Fortuna 0.14.0
- Fixed a bug where the analytic continuation algorithm caused a rare issue during compilation on some platforms.

##### Fortuna 0.13.3
- Fixed Test Bug: percent sign was missing in output distributions.
- Readme updated: added update history, fixed some typos.

##### Fortuna 0.13.2
- Readme updated for even more clarity.

##### Fortuna 0.13.1
- Readme updated for clarity.

##### Fortuna 0.13.0
- Minor Bug Fixes.
- Readme updated for aesthetics.
- Added Tests: `.../fortuna_extras/fortuna_tests.py`

##### Fortuna 0.12.0
- Internal test for future update.

##### Fortuna 0.11.0
- Initial Release: Public Beta

##### Fortuna 0.10.0
- Module name changed from Dice to Fortuna

##### Dice 0.1.x - 0.9.x
- Experimental Phase


## Distribution and Performance Tests
Testbed:
- Hardware: 2.3Ghz 8-core i9, 16GB RAM, 1TB SSD
- Software: MacOS 10.15.5, Python 3.8, MonkeyScope: Fortuna
```
MonkeyScope: Fortuna Quick Test

Random Sequence Values:

some_list = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

Base Case
Output Analysis: Random.choice(some_list)
Typical Timing: 472 ± 42 ns
Statistics of 1000 samples:
 Minimum: 0
 Median: 5
 Maximum: 9
 Mean: 4.725
 Std Deviation: 2.935982798824781
Distribution of 100000 samples:
 0: 9.823%
 1: 10.09%
 2: 10.03%
 3: 9.827%
 4: 10.038%
 5: 10.032%
 6: 10.037%
 7: 10.06%
 8: 10.048%
 9: 10.015%

Output Analysis: random_value(some_list)
Typical Timing: 60 ± 2 ns
Statistics of 1000 samples:
 Minimum: 0
 Median: 4
 Maximum: 9
 Mean: 4.466
 Std Deviation: 2.801551836517164
Distribution of 100000 samples:
 0: 9.886%
 1: 9.857%
 2: 10.087%
 3: 10.034%
 4: 9.947%
 5: 10.019%
 6: 9.904%
 7: 10.067%
 8: 10.04%
 9: 10.159%


Wide Distribution

Truffle = TruffleShuffle(some_list)
Output Analysis: Truffle()
Typical Timing: 346 ± 2 ns
Statistics of 1000 samples:
 Minimum: 0
 Median: 5
 Maximum: 9
 Mean: 4.565
 Std Deviation: 2.8590118833171974
Distribution of 100000 samples:
 0: 9.988%
 1: 9.966%
 2: 10.02%
 3: 10.019%
 4: 10.053%
 5: 10.115%
 6: 9.955%
 7: 9.878%
 8: 10.039%
 9: 9.967%

truffle = truffle_shuffle(some_list)
Output Analysis: truffle()
Typical Timing: 143 ± 26 ns
Statistics of 1000 samples:
 Minimum: 0
 Median: 5
 Maximum: 9
 Mean: 4.545
 Std Deviation: 2.9076501559557384
Distribution of 100000 samples:
 0: 10.072%
 1: 9.951%
 2: 10.012%
 3: 10.027%
 4: 10.003%
 5: 9.933%
 6: 9.881%
 7: 10.084%
 8: 10.086%
 9: 9.951%


Single objects with many distribution possibilities

some_tuple = tuple(i for i in range(10))

monty = QuantumMonty(some_tuple)
Output Analysis: monty()
Typical Timing: 390 ± 30 ns
Statistics of 1000 samples:
 Minimum: 0
 Median: 4
 Maximum: 9
 Mean: 4.438
 Std Deviation: 2.82136777399121
Distribution of 100000 samples:
 0: 10.778%
 1: 8.892%
 2: 9.042%
 3: 9.749%
 4: 11.685%
 5: 11.616%
 6: 9.693%
 7: 8.909%
 8: 8.902%
 9: 10.734%

rand_value = RandomValue(collection, zero_cool, flat)
Output Analysis: rand_value()
Typical Timing: 305 ± 20 ns
Statistics of 1000 samples:
 Minimum: 0
 Median: 5
 Maximum: 9
 Mean: 4.527
 Std Deviation: 2.860673776307682
Distribution of 100000 samples:
 0: 10.015%
 1: 10.011%
 2: 9.926%
 3: 10.04%
 4: 10.027%
 5: 10.118%
 6: 9.968%
 7: 9.951%
 8: 9.9%
 9: 10.044%


Weighted Tables:

population = ('A', 'B', 'C', 'D')
cum_weights = (1, 3, 6, 10)
rel_weights = (1, 2, 3, 4)
cum_weighted_table = zip(cum_weights, population)
rel_weighted_table = zip(rel_weights, population)

Cumulative Base Case
Output Analysis: Random.choices(population, cum_weights=cum_weights)
Typical Timing: 1179 ± 89 ns
Distribution of 100000 samples:
 A: 10.187%
 B: 19.965%
 C: 29.89%
 D: 39.958%

cum_weighted_choice = CumulativeWeightedChoice(cum_weighted_table)
Output Analysis: cum_weighted_choice()
Typical Timing: 298 ± 21 ns
Distribution of 100000 samples:
 A: 10.003%
 B: 19.936%
 C: 30.053%
 D: 40.008%

Output Analysis: cumulative_weighted_choice(tuple(zip(cum_weights, population)))
Typical Timing: 125 ± 23 ns
Distribution of 100000 samples:
 A: 10.129%
 B: 20.17%
 C: 29.789%
 D: 39.912%

Relative Base Case
Output Analysis: Random.choices(population, weights=rel_weights)
Typical Timing: 1497 ± 126 ns
Distribution of 100000 samples:
 A: 10.012%
 B: 19.969%
 C: 29.911%
 D: 40.108%

rel_weighted_choice = RelativeWeightedChoice(rel_weighted_table)
Output Analysis: rel_weighted_choice()
Typical Timing: 304 ± 14 ns
Distribution of 100000 samples:
 A: 10.015%
 B: 20.016%
 C: 30.066%
 D: 39.903%


Random Matrix Values:

some_matrix = {'A': (1, 2, 3, 4), 'B': (10, 20, 30, 40), 'C': (100, 200, 300, 400)}

flex_cat = FlexCat(some_matrix)
Output Analysis: flex_cat()
Typical Timing: 691 ± 39 ns
Statistics of 1000 samples:
 Minimum: 1
 Median: 4
 Maximum: 400
 Mean: 37.94
 Std Deviation: 84.48752891724638
Distribution of 100000 samples:
 1: 13.869%
 2: 13.999%
 3: 14.023%
 4: 13.936%
 10: 8.248%
 20: 8.335%
 30: 8.189%
 40: 8.324%
 100: 2.764%
 200: 2.783%
 300: 2.785%
 400: 2.745%

Output Analysis: flex_cat("C")
Typical Timing: 486 ± 49 ns
Statistics of 1000 samples:
 Minimum: 100
 Median: 300
 Maximum: 400
 Mean: 251.5
 Std Deviation: 112.47399877482974
Distribution of 100000 samples:
 100: 24.879%
 200: 25.03%
 300: 24.936%
 400: 25.155%


Random Integers:

Base Case
Output Analysis: Random.randrange(10)
Typical Timing: 427 ± 45 ns
Statistics of 1000 samples:
 Minimum: 0
 Median: 4
 Maximum: 9
 Mean: 4.428
 Std Deviation: 2.8065446179764706
Distribution of 100000 samples:
 0: 10.136%
 1: 10.057%
 2: 10.014%
 3: 9.929%
 4: 9.937%
 5: 9.845%
 6: 9.874%
 7: 10.074%
 8: 10.102%
 9: 10.032%

Output Analysis: random_below(10)
Typical Timing: 56 ± 5 ns
Statistics of 1000 samples:
 Minimum: 0
 Median: 4
 Maximum: 9
 Mean: 4.484
 Std Deviation: 2.898300216358572
Distribution of 100000 samples:
 0: 10.075%
 1: 10.088%
 2: 10.014%
 3: 10.011%
 4: 9.869%
 5: 9.993%
 6: 9.862%
 7: 10.059%
 8: 10.078%
 9: 9.951%

Output Analysis: random_index(10)
Typical Timing: 54 ± 4 ns
Statistics of 1000 samples:
 Minimum: 0
 Median: 5
 Maximum: 9
 Mean: 4.485
 Std Deviation: 2.8223643883702794
Distribution of 100000 samples:
 0: 9.943%
 1: 9.955%
 2: 10.199%
 3: 10.26%
 4: 9.986%
 5: 10.044%
 6: 10.033%
 7: 9.892%
 8: 9.774%
 9: 9.914%

Output Analysis: random_range(10)
Typical Timing: 64 ± 2 ns
Statistics of 1000 samples:
 Minimum: 0
 Median: 5
 Maximum: 9
 Mean: 4.627
 Std Deviation: 2.858328364803638
Distribution of 100000 samples:
 0: 9.977%
 1: 10.124%
 2: 10.039%
 3: 9.957%
 4: 10.035%
 5: 10.07%
 6: 9.924%
 7: 9.945%
 8: 9.989%
 9: 9.94%

Output Analysis: random_below(-10)
Typical Timing: 74 ± 4 ns
Statistics of 1000 samples:
 Minimum: -9
 Median: -5
 Maximum: 0
 Mean: -4.651
 Std Deviation: 2.832529297681177
Distribution of 100000 samples:
 -9: 10.009%
 -8: 9.936%
 -7: 10.028%
 -6: 10.029%
 -5: 10.043%
 -4: 9.943%
 -3: 9.979%
 -2: 9.961%
 -1: 9.912%
 0: 10.16%

Output Analysis: random_index(-10)
Typical Timing: 92 ± 6 ns
Statistics of 1000 samples:
 Minimum: -10
 Median: -5
 Maximum: -1
 Mean: -5.417
 Std Deviation: 2.917811615684711
Distribution of 100000 samples:
 -10: 9.919%
 -9: 9.906%
 -8: 9.993%
 -7: 10.103%
 -6: 10.187%
 -5: 9.993%
 -4: 9.902%
 -3: 9.904%
 -2: 10.03%
 -1: 10.063%

Output Analysis: random_range(-10)
Typical Timing: 94 ± 4 ns
Statistics of 1000 samples:
 Minimum: -10
 Median: -6
 Maximum: -1
 Mean: -5.507
 Std Deviation: 2.8817104738774946
Distribution of 100000 samples:
 -10: 9.807%
 -9: 9.977%
 -8: 9.989%
 -7: 10.06%
 -6: 9.929%
 -5: 10.156%
 -4: 10.111%
 -3: 10.058%
 -2: 10.014%
 -1: 9.899%

Base Case
Output Analysis: Random.randrange(1, 10)
Typical Timing: 547 ± 20 ns
Statistics of 1000 samples:
 Minimum: 1
 Median: 5
 Maximum: 9
 Mean: 4.945
 Std Deviation: 2.576939195172369
Distribution of 100000 samples:
 1: 11.24%
 2: 11.041%
 3: 11.05%
 4: 11.142%
 5: 11.185%
 6: 11.035%
 7: 11.109%
 8: 11.018%
 9: 11.18%

Output Analysis: random_range(1, 10)
Typical Timing: 79 ± 13 ns
Statistics of 1000 samples:
 Minimum: 1
 Median: 5
 Maximum: 9
 Mean: 5.02
 Std Deviation: 2.6000692991457672
Distribution of 100000 samples:
 1: 11.12%
 2: 11.223%
 3: 11.154%
 4: 11.101%
 5: 11.015%
 6: 11.138%
 7: 11.073%
 8: 11.094%
 9: 11.082%

Output Analysis: random_range(10, 1)
Typical Timing: 77 ± 7 ns
Statistics of 1000 samples:
 Minimum: 1
 Median: 5
 Maximum: 9
 Mean: 4.949
 Std Deviation: 2.55596008026963
Distribution of 100000 samples:
 1: 11.112%
 2: 11.264%
 3: 11.222%
 4: 11.062%
 5: 10.988%
 6: 11.002%
 7: 10.981%
 8: 11.299%
 9: 11.07%

Base Case
Output Analysis: Random.randint(-5, 5)
Typical Timing: 645 ± 43 ns
Statistics of 1000 samples:
 Minimum: -5
 Median: 0
 Maximum: 5
 Mean: -0.081
 Std Deviation: 3.135326022803573
Distribution of 100000 samples:
 -5: 8.994%
 -4: 9.202%
 -3: 9.024%
 -2: 9.016%
 -1: 9.207%
 0: 8.973%
 1: 9.025%
 2: 9.249%
 3: 9.011%
 4: 9.14%
 5: 9.159%

Output Analysis: random_int(-5, 5)
Typical Timing: 50 ± 7 ns
Statistics of 1000 samples:
 Minimum: -5
 Median: 0
 Maximum: 5
 Mean: -0.036
 Std Deviation: 3.0561485638044568
Distribution of 100000 samples:
 -5: 9.079%
 -4: 9.012%
 -3: 9.213%
 -2: 9.131%
 -1: 9.159%
 0: 8.983%
 1: 8.973%
 2: 8.961%
 3: 9.147%
 4: 9.196%
 5: 9.146%

Base Case
Output Analysis: Random.randrange(1, 20, 2)
Typical Timing: 736 ± 76 ns
Statistics of 1000 samples:
 Minimum: 1
 Median: 11
 Maximum: 19
 Mean: 10.31
 Std Deviation: 5.781463943096534
Distribution of 100000 samples:
 1: 9.948%
 3: 10.133%
 5: 10.209%
 7: 10.026%
 9: 9.956%
 11: 9.878%
 13: 10.22%
 15: 9.814%
 17: 10.021%
 19: 9.795%

Output Analysis: random_range(1, 20, 2)
Typical Timing: 77 ± 14 ns
Statistics of 1000 samples:
 Minimum: 1
 Median: 9
 Maximum: 19
 Mean: 9.672
 Std Deviation: 5.65299676033627
Distribution of 100000 samples:
 1: 9.929%
 3: 10.124%
 5: 10.06%
 7: 9.9%
 9: 10.053%
 11: 10.019%
 13: 10.066%
 15: 9.978%
 17: 10.038%
 19: 9.833%

Output Analysis: random_range(1, 20, -2)
Typical Timing: 72 ± 9 ns
Statistics of 1000 samples:
 Minimum: 2
 Median: 12
 Maximum: 20
 Mean: 11.016
 Std Deviation: 5.77867953230988
Distribution of 100000 samples:
 2: 10.059%
 4: 9.894%
 6: 9.873%
 8: 10.039%
 10: 9.873%
 12: 9.954%
 14: 9.946%
 16: 10.239%
 18: 10.018%
 20: 10.105%

Output Analysis: random_range(20, 1, -2)
Typical Timing: 78 ± 12 ns
Statistics of 1000 samples:
 Minimum: 2
 Median: 10
 Maximum: 20
 Mean: 10.86
 Std Deviation: 5.708676640780154
Distribution of 100000 samples:
 2: 9.98%
 4: 9.989%
 6: 10.018%
 8: 9.852%
 10: 10.062%
 12: 10.15%
 14: 9.981%
 16: 9.889%
 18: 10.064%
 20: 10.015%

Output Analysis: d(10)
Typical Timing: 60 ± 13 ns
Statistics of 1000 samples:
 Minimum: 1
 Median: 5
 Maximum: 10
 Mean: 5.459
 Std Deviation: 2.9351207699060584
Distribution of 100000 samples:
 1: 9.977%
 2: 10.092%
 3: 10.064%
 4: 9.843%
 5: 9.99%
 6: 10.02%
 7: 9.854%
 8: 10.063%
 9: 10.076%
 10: 10.021%

Output Analysis: dice(3, 6)
Typical Timing: 107 ± 13 ns
Statistics of 1000 samples:
 Minimum: 3
 Median: 10
 Maximum: 18
 Mean: 10.524
 Std Deviation: 3.035562326594307
Distribution of 100000 samples:
 3: 0.443%
 4: 1.388%
 5: 2.842%
 6: 4.604%
 7: 6.87%
 8: 9.721%
 9: 11.596%
 10: 12.471%
 11: 12.438%
 12: 11.562%
 13: 9.854%
 14: 7.02%
 15: 4.686%
 16: 2.679%
 17: 1.355%
 18: 0.471%

Output Analysis: ability_dice(4)
Typical Timing: 151 ± 7 ns
Statistics of 1000 samples:
 Minimum: 5
 Median: 12
 Maximum: 18
 Mean: 12.204
 Std Deviation: 2.797536453776824
Distribution of 100000 samples:
 3: 0.078%
 4: 0.277%
 5: 0.748%
 6: 1.578%
 7: 2.963%
 8: 4.837%
 9: 6.926%
 10: 9.492%
 11: 11.35%
 12: 13.047%
 13: 13.179%
 14: 12.28%
 15: 10.129%
 16: 7.278%
 17: 4.196%
 18: 1.642%

Output Analysis: plus_or_minus(5)
Typical Timing: 44 ± 1 ns
Statistics of 1000 samples:
 Minimum: -5
 Median: 0
 Maximum: 5
 Mean: -0.103
 Std Deviation: 3.124764815974981
Distribution of 100000 samples:
 -5: 9.144%
 -4: 9.07%
 -3: 9.047%
 -2: 9.054%
 -1: 9.038%
 0: 9.075%
 1: 9.078%
 2: 9.054%
 3: 9.083%
 4: 9.237%
 5: 9.12%

Output Analysis: plus_or_minus_linear(5)
Typical Timing: 80 ± 17 ns
Statistics of 1000 samples:
 Minimum: -5
 Median: 0
 Maximum: 5
 Mean: 0.115
 Std Deviation: 2.41694365193246
Distribution of 100000 samples:
 -5: 2.689%
 -4: 5.533%
 -3: 8.319%
 -2: 11.064%
 -1: 13.821%
 0: 16.634%
 1: 14.055%
 2: 11.252%
 3: 8.338%
 4: 5.494%
 5: 2.801%

Output Analysis: plus_or_minus_gauss(5)
Typical Timing: 93 ± 15 ns
Statistics of 1000 samples:
 Minimum: -5
 Median: 0
 Maximum: 5
 Mean: -0.027
 Std Deviation: 1.6083711816175574
Distribution of 100000 samples:
 -5: 0.204%
 -4: 1.197%
 -3: 4.403%
 -2: 11.691%
 -1: 20.338%
 0: 24.488%
 1: 20.32%
 2: 11.533%
 3: 4.439%
 4: 1.154%
 5: 0.233%


Random Floats:

Base Case
Output Analysis: Random.random()
Typical Timing: 37 ± 2 ns
Statistics of 1000 samples:
 Minimum: 0.000314571881973702
 Median: (0.48471457907903714, 0.4853495592145515)
 Maximum: 0.999861423354397
 Mean: 0.4958229892867465
 Std Deviation: 0.2934851731001712
Post-processor distribution of 100000 samples using round method:
 0: 50.095%
 1: 49.905%

Output Analysis: canonical()
Typical Timing: 41 ± 2 ns
Statistics of 1000 samples:
 Minimum: 0.000614372825352497
 Median: (0.4962570104034604, 0.49735078630700397)
 Maximum: 0.9998198760823169
 Mean: 0.5008301543263457
 Std Deviation: 0.2876964574785179
Post-processor distribution of 100000 samples using round method:
 0: 50.025%
 1: 49.975%

Output Analysis: random_float(0.0, 10.0)
Typical Timing: 41 ± 2 ns
Statistics of 1000 samples:
 Minimum: 0.019843418905310814
 Median: (5.0367567406596905, 5.04265209789758)
 Maximum: 9.983775271102553
 Mean: 5.013876330997951
 Std Deviation: 2.85806125795982
Post-processor distribution of 100000 samples using floor method:
 0: 9.92%
 1: 10.102%
 2: 9.831%
 3: 9.923%
 4: 10.114%
 5: 10.129%
 6: 9.939%
 7: 9.924%
 8: 10.02%
 9: 10.098%

Base Case
Output Analysis: Random.triangular(0.0, 10.0, 5.0)
Typical Timing: 344 ± 16 ns
Statistics of 1000 samples:
 Minimum: 0.2830330935066536
 Median: (4.90935140748834, 4.913870639709567)
 Maximum: 9.901019587399393
 Mean: 4.966933693460001
 Std Deviation: 2.0416381591291914
Post-processor distribution of 100000 samples using round method:
 0: 0.513%
 1: 4.062%
 2: 7.964%
 3: 12.082%
 4: 15.916%
 5: 18.875%
 6: 15.896%
 7: 12.193%
 8: 7.954%
 9: 4.065%
 10: 0.48%

Output Analysis: triangular(0.0, 10.0, 5.0)
Typical Timing: 54 ± 3 ns
Statistics of 1000 samples:
 Minimum: 0.3232415614904267
 Median: (4.9962504105669145, 4.998047410485638)
 Maximum: 9.932314430597117
 Mean: 5.021856840304916
 Std Deviation: 2.053981647013735
Post-processor distribution of 100000 samples using round method:
 0: 0.517%
 1: 4.08%
 2: 8.042%
 3: 11.962%
 4: 15.941%
 5: 18.999%
 6: 15.929%
 7: 12.042%
 8: 8.049%
 9: 3.931%
 10: 0.508%


Random Booleans:

Output Analysis: percent_true(33.33)
Typical Timing: 42 ± 3 ns
Statistics of 1000 samples:
 Minimum: False
 Median: False
 Maximum: True
 Mean: 0.321
 Std Deviation: 0.46709439857182744
Distribution of 100000 samples:
 False: 66.594%
 True: 33.406%


Shuffle Performance:

some_small_list = [i for i in range(10)]
some_med_list = [i for i in range(100)]
some_large_list = [i for i in range(1000)]

Base Case:
Random.shuffle()
Typical Timing: 4055 ± 198 ns
Typical Timing: 35417 ± 131 ns
Typical Timing: 342183 ± 11183 ns

Fortuna.shuffle()
Typical Timing: 342 ± 57 ns
Typical Timing: 3178 ± 35 ns
Typical Timing: 31790 ± 362 ns

Fortuna.knuth_a()
Typical Timing: 837 ± 122 ns
Typical Timing: 6389 ± 389 ns
Typical Timing: 71500 ± 929 ns

Fortuna.fisher_yates()
Typical Timing: 756 ± 42 ns
Typical Timing: 5916 ± 59 ns
Typical Timing: 75634 ± 3063 ns


-------------------------------------------------------------------------
Total Test Time: 2.671 seconds
```


## Legal Information
Fortuna is licensed under a Creative Commons Attribution-NonCommercial 3.0 Unported License. 
See online version of this license here: <http://creativecommons.org/licenses/by-nc/3.0/>

Other licensing options are available, please contact the author for details: [Robert Sharp](mailto:webmaster@sharpdesigndigital.com)
