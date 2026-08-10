---
layout: post
title: "A few examples of algebraic data types in Python"
categories: Programming
---

# A few examples of algebraic data types in Python

In this notebook we translate the Rust code snippets of this [blogpost](https://alex.draftist.io/blog/the-bedrock-of-software-design-ycqvcedsj) to Python. We acknowledge the helpful top answer from this [SO post](https://stackoverflow.com/questions/16258553/how-can-i-define-algebraic-data-types-in-python).


```python
from dataclasses import dataclass
from typing import Literal, Optional, assert_never
```


```python
# Snippet 1: product type
@dataclass
class User:
    id: int # UserId
    role: str # Role
```


```python
# Snippet 2: sum type
@dataclass
class AnonymousSession:
    pass

Session = AnonymousSession | User # Authenticated(User) in general
```


```python
# Note: We can improve Snippet 1 using the sum type

class UserRole:
    pass

class AdminRole:
    pass

Role = UserRole | AdminRole 

@dataclass
class User:
    id: int # UserId
    role: Role # Role
```


```python
# Snippet 3: Naive Theme
ColorMode = Literal["light", "dark"]

@dataclass
class ColorScheme:
    description: str

@dataclass
class SiteTheme:
    switchable: bool
    default_mode: ColorMode | None
    light_scheme: ColorScheme | None
    dark_scheme: ColorScheme | None
    fixed_mode: ColorMode | None
    fixed_scheme: ColorScheme | None
```


```python
# Snippet 4: Constrained Theme
@dataclass
class Switchable:
    default_mode: ColorMode
    light_scheme: ColorScheme
    dark_scheme: ColorScheme

@dataclass
class Fixed:
    mode: ColorMode
    scheme: ColorScheme

SiteTheme2 = Switchable | Fixed
```


```python
# Snippet 5: Validating with exceptions

class PortError(Exception):
    pass 

def parse_port(value: str | int) -> int | None:
    if not isinstance(value, int) or value < 1 or value > 65535:
        raise PortError("invalid port")

    return value
```


```python
ports : list[int | str] = [0, 'ss', 4]
for port in ports:
    try:
        parse_port(port)
    except PortError as err:
        print(port, err)
    else:
        print(port, 'ok')
```

    0 invalid port
    ss invalid port
    4 ok
    


```python
# Snippets 6 and 7: Result type and Validating with types

@dataclass
class PortOK:
    value: int

class PortOutOfRangeError(Exception):
    pass

PortError2 = ValueError | PortOutOfRangeError
ResultCode = PortOK | PortError2 
ParsePortResult = ResultCode

class PortRange:
    min = 1
    max = 65535

    def check_valid(self, x: int) -> ParsePortResult:
        if self.min <= x <= self.max:
            return PortOK(x)
        else:
            return PortOutOfRangeError(f"x is not within ({self.min}, {self.max})")

def parse_port2(value: int | str) -> ParsePortResult:
    match value:
        case int():
            return PortRange().check_valid(value)
        case _:
            return ValueError(f"type of value: {type(value)} is not int")
```


```python
print(parse_port2(-1))
print(parse_port2('ss'))
print(parse_port2(4))
```

    x is not within (1, 65535)
    type of value: <class 'str'> is not int
    PortOK(value=4)
    


```python
# Snippets 8 and 9: optional type
x: Optional[object] # some linters will complain and ask to replace it with
x: object | None

def find_user(id: int) -> User | None:
    pass 
```


```python
# Snippet 10: FindUser with optional and status
class OK:
    pass

class FindUserError(Exception):
    pass

Result = tuple[User | None, FindUserError | OK]
def find_user2(id: int) -> Result:
    pass # type checker will complain until implementation
```


```python
# Snippet 11: calling parse_port2
ports : list[int | str] = [0, 'ss', 4]
for port in ports:
    match res := parse_port2(port):
        case PortOK(port):
            print(f"Using port {port}!")
        case ValueError():
            print("Port must be a number")
        case PortOutOfRangeError():
            print(res)
    
```

    x is not within (1, 65535)
    Port must be a number
    Using port 4!
    


```python
# Snippets 12, 13 and 14: Roles, the article is a bit confusing because "User" denotes both the User who has a role
# i.e. user.role and the user role i.e. normal user or admin

class StaffRole:
    pass

class BanUserError(Exception):
    pass

Role = UserRole | AdminRole | StaffRole

def ban_user(user_id: str, current_user: User) -> OK | BanUserError:
    match current_user.role:
        case UserRole():
            return BanUserError("Normal user not allowed to ban!")
        case AdminRole():
            # ...
            return OK()
        # case StaffRole():
        #     return OK()
        case _ as unreachable:
            assert_never(unreachable)
            # until we add a case for the StaffRole the type checker complains:
            # Argument of type "StaffRole" cannot be assigned to parameter 
            # "arg" of type "Never" in function "assert_never"

```


```python
# Snippets 15 and 16: combainations of input and result

class Domestic:
    pass

class International:
    pass

Destination = Domestic | International

class Standard:
    pass

class Express:
    pass

ShippingSpeed = Standard | Express

def delivery_days(destination: Destination, speed: ShippingSpeed) -> int:
    match (destination, speed):
        case Domestic(), Standard():
            return 3
        case Domestic(), Express():
            return 1
        case International(), Standard():
            return 10
        case International(), Express():
            return 3
```

# Feedback and a challenge

This exercise has been my first foray into ADTs in Python, so it's possible I've made a mistake or that the code is not as pythonic/clean as it could be, let me know at `rares<dot>turliuc<at>gmail<dot>com`. My challenge to you is to do the same exercise in the language of your [choice](https://en.wikipedia.org/wiki/Comparison_of_programming_languages_(algebraic_data_type)).

If you want to execute the code and play around with this notebook click the button below:

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/git/https%3A%2F%2Fcodeberg.org%2FCalinRaresT%2Fpublic_notebooks/HEAD?filepath=adt_sandbox.ipynb)
