# SOFT2201 A4 — Completed Answers

> This file contains full solutions you can submit directly. Replace any placeholder student info if your scaffold expects it.

---

## Section A — Multiple Choice (Q1–Q20)

1. C  
2. B  
3. A  
4. B  
5. B  
6. A  
7. B  
8. A  
9. A  
10. A  
11. A  
12. B  
13. C  
14. A  
15. C  
16. A  
17. A  
18. D  
19. D  
20. A

---

## Section B — A Program Before Refactoring

### Q21. Which SOLID principle is violated? Why?

**Principle:** **Open/Closed Principle (OCP)**.  
**Why:** `DataImputer.impute` uses `if/elif` branching over string method names (`"mean" | "median" | "constant"`). Adding a new algorithm (e.g., *rolling median*) requires modifying the method and the class—i.e., we must change existing code instead of extending behavior. That violates “open for extension, closed for modification.”

---

### Q22. Add a new method: *rolling median* (no refactor)

**Requirement:** For each missing value, fill with the median of the **previous n observed values**; assume the first *n* items are all non-missing. Keep the existing style (do not refactor).

Add a small helper and one new branch:

```python
# class DataImputer:
def _rolling_median(self, values, n: int):
    out = list(values)
    observed = []  # previously observed (after imputation)
    for i, x in enumerate(values):
        if x is not None:
            observed.append(x)
            continue

        # window of last n observed values
        window = observed[-n:] if len(observed) >= n else observed
        # problem statement guarantees first n are observed
        sorted_w = sorted(window)
        m = len(sorted_w) // 2
        if len(sorted_w) % 2 == 1:
            med = sorted_w[m]
        else:
            # honour tiebreak behaviour used elsewhere in the class
            if self.tiebreak == "average":
                med = (sorted_w[m-1] + sorted_w[m]) / 2
            elif self.tiebreak == "lower":
                med = sorted_w[m-1]
            else:  # "upper"
                med = sorted_w[m]
        out[i] = med
        observed.append(med)
    return out
```

```python
def impute(self, values, method, n=None):
    original_data = [x for x in values if x is not None]

    if method == "mean":
        m = sum(original_data) / len(original_data)
        return [m if x is None else x for x in values]

    elif method == "median":
        med = self._median(original_data)
        return [med if x is None else x for x in values]

    elif method == "constant":
        c = float(self.constant)
        return [c if x is None else x for x in values]

    elif method == "rolling_median":
        if n is None or n <= 0:
            raise ValueError("rolling_median requires positive n")
        return self._rolling_median(values, n)

    else:
        raise ValueError(f"Unknown method: {method}")
```

> Notes: stays minimal, respects the class’s tie-break policy, and assumes the first *n* entries are observed as stated in the question.

---

### Q23. Replace `if/elif` with a design pattern

**Pattern:** **Strategy Pattern** (optionally with a simple registry).

- Define an interface `ImputationStrategy` with `apply(values, **kwargs) -> list`.
- Implement concrete strategies: `MeanImputer`, `MedianImputer`, `ConstantImputer`, `RollingMedianImputer`.
- `DataImputer` acts as the **Context**; it selects an algorithm from a registry (dict) or via dependency injection and delegates to `strategy.apply(...)`.
- Adding a new method then requires **only adding a new strategy class and registering it**, not editing `DataImputer`, satisfying OCP.

Sketch:

```python
from abc import ABC, abstractmethod

class ImputationStrategy(ABC):
    @abstractmethod
    def apply(self, values): ...

class MeanImputer(ImputationStrategy):
    def apply(self, values):
        ok = [v for v in values if v is not None]
        mean = sum(ok) / len(ok)
        return [mean if v is None else v for v in values]
```

```python
class DataImputer:
    def __init__(self, strategies):
        self.strategies = strategies  # e.g., {"mean": MeanImputer(), ...}

    def impute(self, values, method, **kwargs):
        try:
            return self.strategies[method].apply(values, **kwargs)
        except KeyError:
            raise ValueError(f"Unknown method: {method}")
```

---

## Section C — A Program After Refactoring

### Q24. Build a single `Move` that moves robot one step diagonally (ending facing original direction)

```python
def move_diagonally(robot) -> Move:
    return GroupOfMoves(
        MoveForward(robot),
        TurnLeft(robot),
        MoveForward(robot),
        TurnRight(robot),
    )
```

This moves to the forward-left cell and restores the original orientation.

---

### Q25. Identify the pattern and participants

**Pattern:** **Command**, with **Composite (Macro Command)** to group moves.

- `Move` – Command interface.  
- `Robot` – Receiver (executes actual actions).  
- `MoveForward`, `TurnLeft`, `TurnRight` – ConcreteCommand.  
- `GroupOfMoves` – Composite command that sequences sub-commands.

---

### Q26. Add `undo` support

Add `undo()` to the command interface; each command implements its inverse. Composite undoes in **reverse order**:

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
```

---

## Section D — Testing (Composite)

The question’s line numbers refer to the given program. Provide tests that achieve both statement and branch coverage for reachable branches.

### Q27. Lines covered by the given test

Given:

```python
container = Container("greeting", [Text("Hello")])
assert container.render() == "<greeting>Hello,</greeting>"
```

This covers:
- **Text**: constructor and `render()` when `self.text` is truthy.  
- **Container**: constructor branch where `name is not None` and `render()` path with children, including the loop and `if inner_text` being true.

### Q28. Branch coverage table (reachable branches)

| Line | Condition               | True | False |
|----:|--------------------------|:----:|:-----:|
|  11 | `if self.text`           |  Yes   |   NO   |
|  18 | `if name is not None`    |  Yes   |   NO   |
|  24 | `if not self.children`   |  Yes   |   NO   |
|  30 | `if inner_text`          |  Yes   |   NO   |

(The table reflects what the single given test hits; we’ll add tests below to cover the opposite sides.)

### Q29. Additional tests to maximize coverage

Add tests to hit the other sides of each reachable conditional:

```python
# 0) baseline (already given)
container = Container("greeting", [Text("Hello")])
assert container.render() == "<greeting>Hello,</greeting>"

# 1) Text.render false branch (text is None)
assert Text(None).render() == ""

# 2) Container with no children -> line 24 True
empty = Container("empty")
assert empty.render() == "<empty></empty>"

# 3) Container name=None -> line 18 False; default to "container"
auto_named = Container(None, [])
assert auto_named.render() == "<container></container>"

# 4) inner_text False (child renders to empty string)
with_empty_child = Container("wrap", [Text(None)])
assert with_empty_child.render() == "<wrap></wrap>"

# 5) mixed children: one empty, one non-empty (comma logic stable)
mix = Container("greeting2", [Text(None), Text("Hi")])
assert mix.render() == "<greeting2>Hi,</greeting2>"
```

These exercises cover the true/false sides of the four main reachable branches.

---

## Section E — Class Diagram (Facade)

### Q30. Participants & roles

- **APIGateway** — **Facade** that exposes a unified `place_order` to clients.  
- **PaymentService** — subsystem for charging/settlement.  
- **CurrencyService** — subsystem for FX rates and conversion.  
- **InventoryService** — subsystem for item totals and inventory currency context.

### Q31. `place_order` with currency conversion

```python
class APIGateway:
    def __init__(self, payment: PaymentService, currency: CurrencyService, inv: InventoryService):
        self.payment, self.currency, self.inv = payment, currency, inv

    def place_order(self, order_id: str, user_currency: str):
        total = self.inv.order_total(order_id)
        inv_ccy = self.inv.inventory_currency()

        pay_amount = (total if inv_ccy == user_currency
                      else self.currency.convert(amount=total,
                                                 from_ccy=inv_ccy,
                                                 to_ccy=user_currency))
        return self.payment.charge(order_id=order_id,
                                   amount=pay_amount,
                                   currency=user_currency)
```

### Q32. Add `DiscountService` without breaking the Facade

Introduce **DiscountService** with `calc_discount(order_id, amount, currency) -> discount_amount` and update only **APIGateway** to orchestrate it. Subsystems remain unchanged.

```python
class APIGateway:
    def __init__(self, payment, currency, inv, discount):
        self.payment, self.currency, self.inv, self.discount = payment, currency, inv, discount

    def place_order(self, order_id: str, user_currency: str):
        total = self.inv.order_total(order_id)
        inv_ccy = self.inv.inventory_currency()

        # compute discount in inventory currency for determinism
        disc_inv = self.discount.calc_discount(order_id, total, inv_ccy)
        net_inv = max(0, total - disc_inv)

        # convert once for payment
        amount_user = (net_inv if inv_ccy == user_currency
                       else self.currency.convert(net_inv, inv_ccy, user_currency))
        return self.payment.charge(order_id, amount_user, user_currency)
```

---

## Section F — Sequence Diagram (Chain of Responsibility)

### Q33. Successful path messages

1. **Client → AuthMiddleware**: authenticate token/session (ok).  
2. **AuthMiddleware → Validator**: validate request params/body (ok).  
3. **Validator → UserController**: execute business logic.  
4. **UserController → Client**: return success response.

### Q34. Insert `RateLimiter` into the chain

Rewire the chain assembly to:  
`AuthMiddleware → RateLimiter → Validator → UserController`.

No existing handlers need code changes; only the **wiring** (the “next” pointers) changes, which is the key extensibility benefit of Chain of Responsibility.

---

**End of Answers.**
