# Python Type Hints

The content mainly comes from [mypy documentation](https://mypy.readthedocs.io/en/stable/index.html).

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

* `NewType`: create an entirely new and unique type when used.

    ```python
    from typing import NewType

    UserId = NewType('UserId', int)

    def name_by_id(user_id: UserId) -> str:
        ...

    UserId('user')          # Fails type check

    name_by_id(42)          # Fails type check
    name_by_id(UserId(42))  # OK

    num: int = UserId(5) + 1
    ```

* Type aliases: `TextType = str | bytes`, or `TextType: TypeAlias = str | bytes` (>= 3.10)

* Cast

    ```python
    from typing import cast

    o: object = [1]
    x = cast(list[int], o)  # OK
    y = cast(list[str], o)  # OK (cast performs no actual runtime check)
    ```

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

Precise typing of alternative constructors

```python
T = TypeVar('T')

class Base(Generic[T]):
    Q = TypeVar('Q', bound='Base[T]')

    def __init__(self, item: T) -> None:
        self.item = item

    @classmethod
    def make_pair(cls: Type[Q], item: T) -> tuple[Q, Q]:
        return cls(item), cls(item)

class Sub(Base[T]):
    ...

pair = Sub.make_pair('yes')  # Type is "tuple[Sub[str], Sub[str]]"
bad = Sub[int].make_pair('no')  # Error: Argument 1 to "make_pair" of "Base"
                                # has incompatible type "str"; expected "int"
```

## Function overload

1. Basic function overload

    ```python
    from typing import Union, overload

    # Overload *variants* for 'mouse_event'.
    # These variants give extra information to the type checker.
    # They are ignored at runtime.

    @overload
    def mouse_event(x1: int, y1: int) -> ClickEvent: ...
    @overload
    def mouse_event(x1: int, y1: int, x2: int, y2: int) -> DragEvent: ...

    # The actual *implementation* of 'mouse_event'.
    # The implementation contains the actual runtime logic.
    #
    # It may or may not have type hints. If it does, mypy
    # will check the body of the implementation against the
    # type hints.
    #
    # Mypy will also check and make sure the signature is
    # consistent with the provided variants.

    def mouse_event(x1: int,
                    y1: int,
                    x2: Optional[int] = None,
                    y2: Optional[int] = None) -> Union[ClickEvent, DragEvent]:
        if x2 is None and y2 is None:
            return ClickEvent(x1, y1)
        elif x2 is not None and y2 is not None:
            return DragEvent(x1, y1, x2, y2)
        else:
            raise TypeError("Bad arguments")
    ```

2. Method overload

    ```python
    from typing import Sequence, TypeVar, Union, overload

    T = TypeVar('T')

    class MyList(Sequence[T]):
        @overload
        def __getitem__(self, index: int) -> T: ...

        @overload
        def __getitem__(self, index: slice) -> Sequence[T]: ...

        def __getitem__(self, index: Union[int, slice]) -> Union[T, Sequence[T]]:
            if isinstance(index, int):
                # Return a T here
            elif isinstance(index, slice):
                # Return a sequence of Ts here
            else:
                raise TypeError(...)
    ```

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

1. Simple user-defined protocols

    ```python
    from typing import Iterable
    from typing_extensions import Protocol

    class SupportsClose(Protocol):
        # Empty method body (explicit '...')
        def close(self) -> None: ...

    class Resource:  # No SupportsClose base class!

        def close(self) -> None:
        self.resource.release()

        # ... other methods ...

    def close_all(items: Iterable[SupportsClose]) -> None:
        for item in items:
            item.close()

    close_all([Resource(), open('some/file')])  # OK
    ```

2. Defining subprotocols and subclassing protocols

    ```python
    class SupportsRead(Protocol):
        def read(self, amount: int) -> bytes: ...

    class TaggedReadableResource(SupportsClose, SupportsRead, Protocol):
        label: str

    class AdvancedResource(Resource):
        def __init__(self, label: str) -> None:
            self.label = label

        def read(self, amount: int) -> bytes:
            # some implementation
            ...

    resource: TaggedReadableResource
    resource = AdvancedResource('handle with care')  # OK
    ```

3. Inheriting protocols

    ```python
    class SomeProto(Protocol):
        attr: int
        def foo(self) -> str: ...
        # can include default implementations
        def bar(self) -> str:
            print('bar')

    # not protocol, in this situation SomeProto behaves like abstract class
    class ExplicitSubclass(SomeProto):
        def __init__(self) -> None:
            self.attr = 1
        
        def foo(self) -> str:
            print('foo)
    ```

4. Invariance of protocol attributes

    Protocol attributes are invariant.

    ```python
        class Box(Protocol):
            content: object

        class IntBox:
            content: int

        def takes_box(box: Box) -> None: ...

        takes_box(IntBox())  # error: Argument 1 to "takes_box" has incompatible type "IntBox"; expected "Box"

        # this is because Box defines content as a mutable attribute;
        # and can be fixed by declaring content to be read-only
        class Box(Protocol):
            @property
            def content(self) -> object: ...
    ```

5. Recursive protocols

    ```python
    from typing import TypeVar, Optional, Protocol

    class TreeLike(Protocol):
        value: int

        @property
        def left(self) -> Optional['TreeLike']: ...

        @property
        def right(self) -> Optional['TreeLike']: ...

    class SimpleTree:
        def __init__(self, value: int) -> None:
            self.value = value
            self.left: Optional['SimpleTree'] = None
            self.right: Optional['SimpleTree'] = None

    root: TreeLike = SimpleTree(0)  # OK
    ```

6. Using isinstance() with protocols

    ```python
    from typing import Protocol, runtime_checkable

    @runtime_checkable
    class Portable(Protocol):
        handles: int
    ```

    NOTE: The runtime implementation only checks that all protocol members exist, not that they have the correct type.

7. Callback protocols

    ```python
    from typing import Optional, Iterable, Protocol

    class Combiner(Protocol):
        def __call__(self, *vals: bytes, maxlen: Optional[int] = None) -> list[bytes]: ...

    def batch_proc(data: Iterable[bytes], cb_results: Combiner) -> bytes:
        for item in data:
            ...

    def good_cb(*vals: bytes, maxlen: Optional[int] = None) -> list[bytes]:
        ...
    def bad_cb(*vals: bytes, maxitems: Optional[int]) -> list[bytes]:
        ...

    batch_proc([], good_cb)  # OK
    batch_proc([], bad_cb)   # Error! Argument 2 has incompatible type because of different name and kind
    ```

## Generics

1. A simple generic class

    ```python
    from typing import TypeVar, Generic

    T = TypeVar('T')

    class Stack(Generic[T]):
        def __init__(self) -> None:
            # Create an empty list with items of type T
            self.items: list[T] = []

        def push(self, item: T) -> None:
            self.items.append(item)

        def pop(self) -> T:
            return self.items.pop()

        def empty(self) -> bool:
            return not self.items
    
    # Construct an empty Stack[int] instance
    stack = Stack[int]()
    stack.push(2)
    stack.pop()
    stack.push('x')  # error

    # Construction of instances of generic types is type checked:

    class Box(Generic[T]):
        def __init__(self, content: T) -> None:
            self.content = content

    Box(1)       # OK, inferred type is Box[int]
    Box[int](1)  # Also OK
    Box[int]('some string')  # error
    ```

2. Defining subclasses of generic classes

    ```python
    from typing import Generic, TypeVar, Mapping, Iterator

    KT = TypeVar('KT')
    VT = TypeVar('VT')

    # This is a generic subclass of Mapping
    class MyMap(Mapping[KT, VT]):
        def __getitem__(self, k: KT) -> VT: ...
        def __iter__(self) -> Iterator[KT]: ...
        def __len__(self) -> int: ...

    items: MyMap[str, int]  # OK

    # This is a non-generic subclass of dict
    class StrDict(dict[str, str]):
        def __str__(self) -> str:
            return f'StrDict({super().__str__()})'


    data: StrDict[int, int]  # Error! StrDict is not generic
    data2: StrDict  # OK

    # This is a user-defined generic class
    class Receiver(Generic[T]):
        def accept(self, value: T) -> None: ...

    # This is a generic subclass of Receiver
    class AdvancedReceiver(Receiver[T]): ...
    ```

3. Generic functions

    ```python
    from typing import TypeVar, Sequence

    T = TypeVar('T')

    # A generic function!
    def first(seq: Sequence[T]) -> T:
        return seq[0]
    ```

4. Generic Self

    ```python
    from typing import TypeVar

    T = TypeVar('T', bound='Shape')

    class Shape:
        def set_scale(self: T, scale: float) -> T:
            self.scale = scale
            return self

    class Circle(Shape):
        def set_radius(self, r: float) -> 'Circle':
            self.radius = r
            return self

    class Square(Shape):
        def set_width(self, w: float) -> 'Square':
            self.width = w
            return self
    
    # Without using generic self, the following two lines could not be type checked properly, since the return type of set_scale would be Shape, which doesn’t define set_radius or set_width.
    circle: Circle = Circle().set_scale(0.5).set_radius(2.7)
    square: Square = Square().set_scale(0.5).set_width(3.2)


    # factory methods

    from typing import TypeVar, Type

    T = TypeVar('T', bound='Friend')

    class Friend:
        other: "Friend" = None

        @classmethod
        def make_pair(cls: Type[T]) -> tuple[T, T]:
            a, b = cls(), cls()
            a.other = b
            b.other = a
            return a, b

    class SuperFriend(Friend):
        pass

    a, b = SuperFriend.make_pair()
    ```

5. Automatic self types using typing.Self

    ```python
    from typing import Self

    class Friend:
        other: Self | None = None

        @classmethod
        def make_pair(cls) -> tuple[Self, Self]:
            a, b = cls(), cls()
            a.other = b
            b.other = a
            return a, b

    class SuperFriend(Friend):
        pass

    a, b = SuperFriend.make_pair()
    ```

6. Variance of generic types

    There are three main kinds of generic types with respect to subtype relations between them: invariant, covariant, and contravariant. Assuming that we have a pair of types A and B, and B is a subtype of A, these are defined as follows:

    A generic class MyCovGen[T] is called covariant in type variable T if MyCovGen[B] is always a subtype of MyCovGen[A].

    A generic class MyContraGen[T] is called contravariant in type variable T if MyContraGen[A] is always a subtype of MyContraGen[B].

    A generic class MyInvGen[T] is called invariant in T if neither of the above is true.

    * Most immutable containers, such as Sequence and FrozenSet are covariant. Union is also covariant in all variables: Union[Triangle, int] is a subtype of Union[Shape, int].

        ```python
        def count_lines(shapes: Sequence[Shape]) -> int:
            return sum(shape.num_sides for shape in shapes)

        triangles: Sequence[Triangle]
        count_lines(triangles)  # OK

        def foo(triangle: Triangle, num: int):
            shape_or_number: Union[Shape, int]
            # a Triangle is a Shape, and a Shape is a valid Union[Shape, int]
            shape_or_number = triangle
        ```

    * Callable is an example of type that behaves contravariant in types of arguments. That is, Callable[[Shape], int] is a subtype of Callable[[Triangle], int], despite Shape being a supertype of Triangle. T

        ```python
        def cost_of_paint_required(
            triangle: Triangle,
            area_calculator: Callable[[Triangle], float]
        ) -> float:
            return area_calculator(triangle) * DOLLAR_PER_SQ_FT

        # This straightforwardly works
        def area_of_triangle(triangle: Triangle) -> float: ...
        cost_of_paint_required(triangle, area_of_triangle)  # OK

        # But this works as well!
        def area_of_any_shape(shape: Shape) -> float: ...
        cost_of_paint_required(triangle, area_of_any_shape)  # OK
        ```

        cost_of_paint_required needs a callable that can calculate the area of a triangle. If we give it a callable that can calculate the area of an arbitrary shape (not just triangles), everything still works.

    * List is an invariant generic type. Naively, one would think that it is covariant, like Sequence above, but consider this code. Another example of invariant type is Dict. Most mutable containers are invariant.

        ```python
        class Circle(Shape):
            # The rotate method is only defined on Circle, not on Shape
            def rotate(self): ...

        def add_one(things: list[Shape]) -> None:
            things.append(Shape())

        my_circles: list[Circle] = []
        add_one(my_circles)     # This may appear safe, but...
        my_circles[-1].rotate()  # ...this will fail, since my_circles[0] is now a Shape, not a Circle
        ```

    * By default, all user-defined generics are invariant. To declare a given generic class as covariant or contravariant use type variables defined with special keyword arguments covariant or contravariant.

        ```python
        from typing import Generic, TypeVar

        T_co = TypeVar('T_co', covariant=True)

        class Box(Generic[T_co]):  # this type is declared covariant
            def __init__(self, content: T_co) -> None:
                self._content = content

            def get_content(self) -> T_co:
                return self._content

        def look_into(box: Box[Animal]): ...

        my_box = Box(Cat())
        look_into(my_box)  # OK, but mypy would complain here for an invariant type
        ```

7. Type variables with upper bounds

    A type variable can also be restricted to having values that are subtypes of a specific type.

    ```python
    from typing import TypeVar, SupportsAbs

    T = TypeVar('T', bound=SupportsAbs[float])
    ```

8. Type variables with value restriction

    ```python
    from typing import TypeVar

    AnyStr = TypeVar('AnyStr', str, bytes)
    ```

9. Declaring decorators
    Basic usage:

    ```python
    from typing import Any, Callable, TypeVar, cast

    F = TypeVar('F', bound=Callable[..., Any])

    # A decorator that preserves the signature.
    def printing_decorator(func: F) -> F:
        def wrapper(*args, **kwds): # NOTE: wrapper is not type checked
            print("Calling", func)
            return func(*args, **kwds)
        return cast(F, wrapper)  # NOTE: cast is required

    @printing_decorator
    def add_forty_two(value: int) -> int:
        return value + 42

    a = add_forty_two(3)
    reveal_type(a)      # Revealed type is "builtins.int"
    add_forty_two('x')  # Argument 1 to "add_forty_two" has incompatible type "str"; expected "int"
    ```

    More faithful but complexed way:

    ```python
    from typing import Callable, TypeVar, ParamSpec

    P = ParamSpec('P')
    T = TypeVar('T')

    def printing_decorator(func: Callable[P, T]) -> Callable[P, T]:
        def wrapper(*args: P.args, **kwds: P.kwargs) -> T:
            print("Calling", func)
            return func(*args, **kwds)
        return wrapper
    ```

    Parameter specifications also allow describe decorators that alter the signature of the input function:

    ```python
    from typing import Callable, TypeVar, ParamSpec

    P = ParamSpec('P')
    T = TypeVar('T')

    # We reuse 'P' in the return type, but replace 'T' with 'str'
    def stringify(func: Callable[P, T]) -> Callable[P, str]:
        def wrapper(*args: P.args, **kwds: P.kwargs) -> str:
            return str(func(*args, **kwds))
        return wrapper

    @stringify
    def add_forty_two(value: int) -> int:
        return value + 42

    a = add_forty_two(3)
    reveal_type(a)      # Revealed type is "builtins.str"
    add_forty_two('x')  # error: Argument 1 to "add_forty_two" has incompatible type "str"; expected "int"
    ```

    Or insert an argument:

    ```python
    from typing import Callable, TypeVar, Concatenate, ParamSpec

    P = ParamSpec('P')
    T = TypeVar('T')

    def printing_decorator(func: Callable[P, T]) -> Callable[Concatenate[str, P], T]:
        def wrapper(msg: str, /, *args: P.args, **kwds: P.kwargs) -> T:
            print("Calling", func, "with", msg)
            return func(*args, **kwds)
        return wrapper

    @printing_decorator
    def add_forty_two(value: int) -> int:
        return value + 42

    a = add_forty_two('three', 3)
    ```

10. Decorator factories

    Functions that take arguments and return a decorator (also called second-order decorators), are similarly supported via generics:

    ```python
    from typing import Any, Callable, TypeVar

    F = TypeVar('F', bound=Callable[..., Any])

    def route(url: str) -> Callable[[F], F]:
        ...

    @route(url='/')
    def index(request: Any) -> str:
        return 'Hello world'
    ```

    Sometimes the same decorator supports both bare calls and calls with arguments. This can be achieved by combining with @overload:

    ```python
    from typing import Any, Callable, Optional, TypeVar, overload

    F = TypeVar('F', bound=Callable[..., Any])

    # Bare decorator usage
    @overload
    def atomic(__func: F) -> F: ...
    # Decorator with arguments
    @overload
    def atomic(*, savepoint: bool = True) -> Callable[[F], F]: ...

    # Implementation
    def atomic(__func: Optional[Callable[..., Any]] = None, *, savepoint: bool = True):
        def decorator(func: Callable[..., Any]):
            ...  # Code goes here
        if __func is not None:
            return decorator(__func)
        else:
            return decorator

    # Usage
    @atomic
    def func1() -> None: ...

    @atomic(savepoint=False)
    def func2() -> None: ...
    ```

11. Generic protocols

    ```python
    from typing import TypeVar, Protocol

    T = TypeVar('T')

    class Box(Protocol[T]):  # NOTE: a shorthand for class Box(Protocol, Generic[T])
        content: T

    def do_stuff(one: Box[str], other: Box[bytes]) -> None:
        ...

    class StringWrapper:
        def __init__(self, content: str) -> None:
            self.content = content

    class BytesWrapper:
        def __init__(self, content: bytes) -> None:
            self.content = content

    do_stuff(StringWrapper('one'), BytesWrapper(b'other'))  # OK

    x: Box[float] = ...
    y: Box[int] = ...
    x = y  # Error -- Box is invariant
    ```

    The main difference between generic protocols and ordinary generic classes is that mypy checks that the declared variances of generic type variables in a protocol match how they are used in the protocol definition.

    ```python
    from typing import Protocol, TypeVar

    T_co = TypeVar('T_co', covariant=True)  # NOTE: `covariant=True` is required

    class ReadOnlyBox(Protocol[T_co]):  # OK
        def content(self) -> T_co: ...

    ax: ReadOnlyBox[float] = ...
    ay: ReadOnlyBox[int] = ...
    ax = ay  # OK -- ReadOnlyBox is covariant
    ```

    Generic protocols can also be recursive.

    ```python
    T = TypeVar('T')

    class Linked(Protocol[T]):
        val: T
        def next(self) -> 'Linked[T]': ...

    class L:
        val: int
        def next(self) -> 'L': ...

    def last(seq: Linked[T]) -> T: ...

    result = last(L())
    reveal_type(result)  # Revealed type is "builtins.int"
    ```

12. Generic type aliases

    ```python
    from typing import TypeVar, Iterable, Union, Callable

    S = TypeVar('S')

    TInt = tuple[int, S]
    UInt = Union[S, int]
    CBack = Callable[..., S]

    def response(query: str) -> UInt[str]:  # Same as Union[str, int]
        ...
    def activate(cb: CBack[S]) -> S:        # Same as Callable[..., S]
        ...
    table_entry: TInt  # Same as tuple[int, Any]

    T = TypeVar('T', int, float, complex)

    Vec = Iterable[tuple[T, T]]

    def inproduct(v: Vec[T]) -> T:
        return sum(x*y for x, y in v)

    def dilate(v: Vec[T], scale: T) -> Vec[T]:
        return ((x * scale, y * scale) for x, y in v)

    v1: Vec[int] = []      # Same as Iterable[tuple[int, int]]
    v2: Vec = []           # Same as Iterable[tuple[Any, Any]]
    v3: Vec[int, int] = [] # Error: Invalid alias, too many type arguments!
    ```

    Type aliases can be imported from modules just like other names. An alias can also target another alias, although building complex chains of aliases is not recommended – this impedes code readability, thus defeating the purpose of using aliases.

## User-Defined Type Guards

1. Basically, a TypeGuard is a “smart” alias for a bool type.

    ```python
    from typing import TypeGuard  # use `typing_extensions` for Python 3.9 and below

    def is_str_list(val: list[object]) -> TypeGuard[list[str]]:
        """Determines whether all objects in the list are strings"""
        return all(isinstance(x, str) for x in val)

    def func1(val: list[object]) -> None:
        if is_str_list(val):
            reveal_type(val)  # list[str]
            print(" ".join(val)) # ok
    ```

2. Generic TypeGuards: typeGuard can also work with generic types.

    ```python
    from typing import TypeVar
    from typing import TypeGuard  # use `typing_extensions` for `python<3.10`

    _T = TypeVar("_T")

    def is_two_element_tuple(val: tuple[_T, ...]) -> TypeGuard[tuple[_T, _T]]:
        return len(val) == 2

    def func(names: tuple[str, ...]):
        if is_two_element_tuple(names):
            reveal_type(names)  # tuple[str, str]
        else:
            reveal_type(names)  # tuple[str, ...]
    ```

3. TypeGuards with parameters: type guard functions can accept extra arguments:

    ```python
    from typing import Type, TypeVar
    from typing import TypeGuard  # use `typing_extensions` for `python<3.10`

    _T = TypeVar("_T")

    def is_set_of(val: set[Any], type: Type[_T]) -> TypeGuard[set[_T]]:
        return all(isinstance(x, type) for x in val)

    items: set[Any]
    if is_set_of(items, str):
        reveal_type(items)  # set[str]

    ```

## 其他

* Fix import cycles

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
