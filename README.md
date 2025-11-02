# SOFT2201/COMP9201 - Assignment 4 Answer Scaffold
## Section A - Multiple Choice Questions (Q1–Q20)

Mark your choice clearly for each question (A–D).

| Q | Answer     |
|---|------------|
| 1 | C| 
| 2 | B            | 
| 3 | A            | 
| 4 | B            | 
| 5 | B            | 
| 6 | A            | 
| 7 | B            | 
| 8 | A            | 
| 9 | A            | 
| 10 | A           | 
| 11 | A           | 
| 12 | B           | 
| 13 | C           | 
| 14 | A          | 
| 15 | C          | 
| 16 | A          | 
| 17 | A          | 
| 18 | D          | 
| 19 | D          | 
| 20 | A          | 

---

## Section B - A Program Before Refactoring 

### Q21. SOLID Principle Violation (4 pts)

__Principle__: Single Responsibility Principle
__Explanation (why/how violated)__:
  ```text
  The DataImputer class handles various data imputation methods such as mean, median, constant, and the implementation of each method is concentrated in the same class. This results in multiple reasons for the class to change: if we need to modify one imputation method or add a new one, we need to modify this class. This violates the Single Responsibility Principle, as a class should have only one reason for change.
  ```

---

### Q22. Add new method: `rolling median` (5 pts)

Answer: 

__Location 1 (class & method):__

__Change:__
  ```python
  def __init__(self, constant=0, tiebreak="average", window_size=3):
    self.constant = constant
    self.tiebreak = tiebreak
    self.window_size = window_size
  ```

__Location 2 (class & method):__

__Change (code/description):__
  ```python
  elif method == "rolling_median":
    return self._rolling_median(values)

# Add new method _rolling_median
def _rolling_median(self, values):
    result = []
    window = []
    
    for i, x in enumerate(values):
        if x is not None:
            window.append(x)
            if len(window) > self.window_size:
                window.pop(0)
            result.append(x)
        else:
            if len(window) > 0:
                med = self.__median(window)
                result.append(med)
            else:
                result.append(0)  
    return result
  ```


---

### Q23. Replace `if/elif` with a Design Pattern (6 pts)


Answer:

__Pattern:__ Strategy 

__Outline of code changes:__
  1. **Remove** the `if/elif` dispatch in `impute`.
  2. Create an abstract base class ImputationStrategy with an impute method
  3. Implement specific strategies for each algorithm: MeanImputer, MedianImputer, ConstantImputer, RollingMedianImputer
  4. Modify DataImputer to accept a strategy object instead of method string
  5. Update impute method to delegate to the strategy object


---

## Section C - A Program After Refactoring (Q24–Q26)

### Q24. Create a single `Move` (4 pts)

Answer:

```python
def move_diagonally(robot) -> Move:
    return GroupOfMoves(
        MoveForward(robot),
        TurnLeft(robot),
        MoveForward(robot),
        TurnRight(robot),
    )
    ...
```

---

### Q25. Identify the design pattern & participant roles (6 pts)
Answer:

__Design pattern:__ Command

__Participants:__ 

Move         : Abstract Command interface
Robot        : Receiver
MoveForward  : Concrete Command
TurnLeft     : Concrete Command
TurnRight    : Concrete Command
GroupOfMoves : Composite Command

---

### Q26. Add `undo` (5 pts)

Answer:

```python

class Move(ABC):
    @abstractmethod
    def execute(self): ...
    @abstractmethod
    def undo(self): ...

class MoveForward(Move):
    def __init__(self, robot): self.robot = robot
    def execute(self): self.robot.move_forward()
    def undo(self):    self.robot.move_backward()

class TurnLeft(Move):
    def __init__(self, robot): self.robot = robot
    def execute(self): self.robot.turn_left()
    def undo(self):    self.robot.turn_right()

class TurnRight(Move):
    def __init__(self, robot): self.robot = robot
    def execute(self): self.robot.turn_right()
    def undo(self):    self.robot.turn_left()

class GroupOfMoves(Move):
    def __init__(self, *moves): self.moves = list(moves)
    def execute(self):
        for m in self.moves: m.execute()
    def undo(self):
        for m in reversed(self.moves):
            m.undo()
  ...

```

---

## Section D - Testing (Composite)

### Q27. Statement coverage: line numbers of covered statements (6 pts)

Answer:

```
Lines covered: 3, 4, 
Text：7, 8, 9, 10, 11, 12
Container：17, 18, 19, 22, 23, 24, 26, 27, 28, 29, 30, 31, 32
```

---

### Q28. Branch coverage (10 pts)

Answer:

| Line | Condition                              | True Taken? | False Taken? | 
|------|----------------------------------------|-------------|--------------| 
| 7    | `if self.text`                         | Yes         |      No      | 
|11      |if self.text                            | Yes         | No             | 
|18      |if name is not None                                        | Yes            | No             | 
|24      |if not self.children                                        | NO            | Yes             | 
|30      |if inner_text                                        | Yes            | No             | 
|      |                                        |             |              | 
|      |                                        |             |              | 

---

### Q29. Improve tests to maximize coverage (9 pts)

Answer:

```python
# Given the existing tests in the test suite
container = Container("greeting", [Text("Hello")])
assert container.render() == "<greeting>Hello,</greeting>"

# Test empty text
empty_text = Text()
assert empty_text.render() == ""

# Test container with no name  
no_name_container = Container(None)
assert no_name_container.render() == "<container></container>"

# Test empty container
empty_container = Container("div")
assert empty_container.render() == "<div></div>"

# Test nested containers
nested = Container("div", [Container("span", [Text("nested")])])
assert nested.render() == "<div><span>nested,</span></div>"


```

---

## Section E - Class Diagram (Facade)

### Q30. (6 pts)

Participants:
APIGateway        : Facade - provides simplified interface to complex subsystems
PaymentService    : Subsystem - handles payment processing
CurrencyService   : Subsystem - handles currency conversion
InventoryService  : Subsystem - manages inventory and currency info

---

### Q31. `place_order` handling currency conversion when placing an order (6 pts)

Answer:  
# Write code or describe the Facade coordinating the other subsystems.
class APIGateway:
    def __init__(self, payment: PaymentService, currency: CurrencyService, inv: InventoryService):
        self.payment, self.currency, self.inv = payment, currency, inv

    def place_order(self, order_id: str, user_currency: str):
        total, inv_ccy = self.inv.order_total(order_id), self.inv.inventory_currency()
        pay_amount = (total if inv_ccy == user_currency
                      else self.currency.convert(amount=total,
                                                from_ccy=inv_ccy,
                                                to_ccy=user_currency))
        receipt = self.payment.charge(order_id=order_id,
                                      amount=pay_amount,
                                      currency=user_currency)
        return receipt


---

### Q32. Adding `DiscountService` (3 pts)

Answer:
__New class(es):__ 
DiscountService

__Existing class(es) to modify (if any)__ 
APIGateway

__Brief explanation:__
Add DiscountService as a dependency to APIGateway and modify place_order to calculate discounts before payment processing.
---

## Section F - Sequence Diagram (Chain of Responsibility)

### Q33. (4 pts)

Client sends request to AuthMiddleware. If authentication succeeds, AuthMiddleware passes request to Validator. If validation succeeds, Validator passes request to UserController. UserController processes request and returns response through the chain back to Client.

---

### Q34. (6 pts)

No, adding RateLimiter would not require changing existing handlers or client code. The Chain of Responsibility pattern allows handlers to be added/removed dynamically. We would simply insert RateLimiter between AuthMiddleware and Validator in the chain. Each handler only needs to know about the next handler in the chain, not the specific implementation.

---

