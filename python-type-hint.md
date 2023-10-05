# Python Type Hint

Starting with Python 3.9 (PEP 585), the type objects of many collections in the standard library support subscription at runtime. This means that you no longer have to import the equivalents from typing; you can simply use the built-in collections or those from collections.abc:

```python
from collections.abc import Sequence
x: list[str]
y: dict[int, str]
z: Sequence[str] = x
```

## Basic

* `int`, `float`, `bool`, `str`, `bytes`, `object`, `Any`, `None`

    NOTE: `Any` vs `object` is similar to `any` vs `unknown` in TypeScript

* `list[str]`, `tuple[int, int]`, `tuple[int, ...]`, `dict[str, int]`, `Iterable[int]`, `Sequence[bool]`(immutable), `Mapping[str, int]`

* `queue.Queue[int]` (>= 3.9) ...
  
* `type[C]`: type object of C (C is a class/type variable/union of types)

* `NoReturn`: a function never returns normally
  
  ```python
  def stop() -> NoReturn:
      raise Exception('no way)

  def run_forever() -> NoReturn:
      while True:
          time.sleep(1)
  ```

* `Optional[int]`

* `Union[int, str]` or `int | str` (>= 3.10)

* `Callable[[int], int]` or `Callable[..., int]` (arbitrary args); can also be used against type objects, matching `__init__` or `__new__`
  
    ```python
    class C:
        def __init__(self, app: str) -> None: pass

    CallableType = Callable[[str], C]

    def class_or_callable(arg: CallableType) -> None:
        inst = arg("my_app")
        reveal_type(inst)  # Revealed type is "C"
    ```

* Named Tuple
  
    ```python
    from typing import NamedTuple

    Point = NamedTuple('Point', [('x', int), ('y', int)])

    # or
    class Point(NamedTuple):
        x: int
        y: int


    p = Point(x=1, y=2)
    ```

* Typed Dict: does not actually define a real class, and it's not possible to use `isinstance` checks

    ```python
    
    class MovieBase(TypedDict):
        name: str
        year: int

    # supports a form of inheritance, but just a notational shortcut
    class Movie(MovieBase):
        director: str
    
    # actor is a non-required key
    class Movie1(Movie, total=False):
        actor: str

    ```

* Generators: `Generator[YieldType, SendType, ReturnType]`; in most cases, use `Iterator[YieldType]` or `Iterable[YieldType]`
* Type aliases: `TextType = str | bytes`, or `TextType: TypeAlias = str | bytes` (>= 3.10)

## Class

```python
from typing import ClassVar, Callable, Self

class Foo:
    x: int
    y: ClassVar[int] = 0  # Class variable only

    def __init__(self, *args: int, **kw: int) -> None:
        pass

    def fn(self) -> Self:
        pass

# An explicit ClassVar may be particularly handy to distinguish between class and instance variables with callable types.
class A:
    foo: Callable[[int], None]
    bar: ClassVar[Callable[[A, int], None]]
    bad: Callable[[A], None]

A().foo(42)  # OK
A().bar(42)  # OK
A().bad()  # Error: Too few arguments

# explicitly mark a method as overriding a base method
from typing import override # >= 3.12

class Base:
    def f(self, x: int) -> None:
        ...
    def g_renamed(self, y: str) -> None:
        ...

class Derived1(Base):
    @override
    def f(self, x: int) -> None:   # OK
        ...

    @override
    def g(self, y: str) -> None:   # Error: no corresponding base method found
        ...
```

## Function overload

## Literal

Indicate that an expression is equal to some specific primitive value.
`enum.Enum` and its subclasses are treated as `Literal`.

```python
@overload
def fetch_data(raw: Literal[True]) -> bytes: ...
@overload
def fetch_data(raw: Literal[False]) -> str: ...
# the last overload is a fallback in case the caller provides a regular bool:
@overload
def fetch_data(raw: bool) -> Union[bytes, str]: ...

def fetch_data(raw: bool) -> Union[bytes, str]:
    # Implementation is omitted
    ...

# can contain one or more literals
PrimaryColors = Literal["red", "blue", "yellow"]
SecondaryColors = Literal["purple", "green", "orange"]
AllowedColors = Literal[PrimaryColors, SecondaryColors]

def paint(color: AllowedColors) -> None: ...

# declare literal variables
a: Literal[19] = 19
b: Final = 19

def expects_literal(x: Literal[19]) -> None: pass

reveal_type(a)  # Revealed type is "Literal[19]"
expects_literal(b)  # also ok!
```

## Final

1. Final names are variables or attributes that should not be reassigned after initialization.

2. Final methods should not be overridden in a subclass.

3. Final classes should not be subclassed.

```python
RATE: Final = 3_000

class Base:
    # Final and ClassVar should not be used together.
    # The scope of a final declaration will be inferred automatically depending on whether it was initialized in the class body or in __init__.
    A: Final = 0  # type will be inferred, not the same as Final[Any]
    # or provide explicit type
    B: Final[int] = 0

    C: Final[int]

    def __init__(self):
        self.C: Final = 0

class Child(Base):
    A: Final = 1 # Error: cannot override a final attr

# Final can only be used as the outermost type
x: list[Final[int]] = []  # Error!

def fun(x: Final[list[int]]) ->  None:  # Error!
    ...

# For overloaded methods add @final on the implementation:
class Base:
    @overload
    def method(self) -> None: ...
    @overload
    def method(self, arg: int) -> int: ...
    @final
    def method(self, x=None):
        ...
```

## Protocols and structural subtyping

## Generics

## 其他

* Class name forward references

```python
from __future__ import annotations # >= 3.7

def f(x: A) -> None: ...  # OK
# def f(x: 'A') -> None: ...  # OK <= 3.6

class A: ...

```

* Import cycles

```python
# foo.py
from typing import TYPE_CHECKING

if TYPE_CHECKING:  # 运行时不会执行
    import bar

def listify(arg: 'bar.BarClass') -> 'list[bar.BarClass]':
    return [arg]

# bar.py

from foo import listify

class BarClass:
    def listifyme(self) -> 'list[BarClass]':
        return listify(self)
```

* Using generic builtins

