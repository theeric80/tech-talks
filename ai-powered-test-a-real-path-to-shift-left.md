# AI-Powered Test - A Real Path to Shift-Left

---

## Slide 1: Title

# AI-Powered Test - A Real Path to Shift-Left

---

## Slide 2: Agenda

1. **Why Shift-Left Hurts**

2. **AI: Fast but Flawed**

3. **Guidelines + Workflow**

4. **Results + Next Steps**

---

## Slide 3: Core Principle

**Bugs are cheap when caught young.**

**Visual:** Bug icons growing in size and cost from Development → Production

**$ → $$ → $$$ → $$$$**

**Development ←――――――――――――――――――――→ Production**

**Why is shift-left hard in practice?**

---

## Slide 4: The Cost Problem

### High Initial Cost
- Design test cases & data
- Write boilerplate and mocks
- Debug tests

### Heavy Maintenance
- Update tests when code changes
- Maintain growing test suite

**Can AI help reduce the cost?**

---

## Slide 5: AI Workflow - First Try

### 1. List Cases
```text
Here is my code @[file]. List all test scenarios for [method],
covering happy paths, edge cases, and error handling.
```

### 2. Generate Code
```text
Based on the scenarios above, write tests for the code.
```

### 3. Run & Debug
```text
If it fails → Ask AI why
```

### 4. Review
```text
✓ Pass?
✓ Tests behavior (not implementation)?
✓ Coverage complete?
```

**AI was fast. Really fast.**

---

## Slide 6: But Speed Came With A Price

```go
func TestDeposit(t *testing.T) {
  wallet := NewWallet()
  wallet.Deposit(100)
  // ✗ tests internal state
  if wallet.balance != 100.0 {
    // ✗ unclear message
    t.Error("failed")
  }
}
```

**Looks green, but…**

Tests internal state → breaks on refactor

**We needed guidelines.**

---

## Slide 7: Guideline #1 - Test Behavior, Not Implementation

### ✗ Bad
```go
func TestDeposit(t *testing.T) {
  wallet := NewWallet()
  wallet.Deposit(100)

  assert.Equal(t, 100.0, wallet.balance) // internal field
}
```

### ✓ Good
```go
func TestDeposit(t *testing.T) {
  wallet := NewWallet()
  wallet.Deposit(100)

  assert.Equal(t, 100.0, wallet.GetBalance()) // public API
}
```

---

## Slide 8: Guideline #2 - AAA Pattern

**Arrange → Act → Assert**

Structure for Clarity

```go
func TestSum(t *testing.T) {
  // Arrange
  a := 5
  b := 10

  // Act
  result := sum(a, b)

  // Assert
  if result != 15 {
    t.Errorf("Expected 15, got %d", result)
  }
}
```

---

## Slide 9: Guideline #3 - Table-Driven Tests

### ✗ Bad
```go
func TestSumPositive(t *testing.T) {
  if got := sum(5, 10); got != 15 { t.Errorf(...) }
}
func TestSumZero(t *testing.T) {
  if got := sum(0, 8); got != 8 { t.Errorf(...) }
}
func TestSumNegative(t *testing.T) {
  if got := sum(-2, -5); got != -7 { t.Errorf(...) }
}
```

### ✓ Good
```go
tests := []struct{ name string; a, b, expected int }{
  {"positive", 5, 10, 15},
  {"zero", 0, 8, 8},
  {"negative", -2, -5, -7},
}

for _, tt := range tests {
  t.Run(tt.name, func(t *testing.T) {
    if got := sum(tt.a, tt.b); got != tt.expected {
      t.Errorf("got %d, want %d", got, tt.expected)
    }
  })
}
```

---

## Slide 10: Guideline #4 - Testing Library

### ✗ Bad
```go
func TestDeposit(t *testing.T) {
  wallet := NewWallet()
  wallet.Deposit(100)

  if got := wallet.GetBalance(); got != 100.0 {
    t.Errorf("Expected 100.0, got %f", got)
  }
}
```

### ✓ Good
```go
func TestDeposit(t *testing.T) {
  wallet := NewWallet()
  wallet.Deposit(100)

  assert.Equal(t, 100.0, wallet.GetBalance())
}
```

---

## Slide 11: AI Workflow - The Right Way

**4 Steps:**

1. **List Cases** → Ask AI for scenarios

2. **Generate Code** ⭐
   Write tests following **@testing-guideline.md**

3. **Run & Debug** → Ask AI to fix

4. **Review** → Pass? Test Behavior? Coverage?

**The key: @testing-guideline.md turns speed into quality**

---

## Slide 12: When to Write Tests

### 1. New Features
Write tests while coding

### 2. Bug Fixes
Write failing test → Fix → Pass

### 3. Refactoring
Add tests first → Refactor safely

---

## Slide 13: What to Test

### Prioritize ✓
- Complex logic
- High churn
- Error-prone modules

### Deprioritize ⚠
- Simple getters/setters
- Deprecated features
- Stable legacy code

**Start with today's work**

---

## Slide 14: The Real-World Impact

| Metric | Before | After | Impact |
|--------|--------|-------|--------|
| New Code Coverage | 0% | **47%** | Safety net |
| Defect discovery | QA / Prod | **Dev** | Shifted left |
| Regression time | Hours | **3 min** | ↓ 99% |
| Delivery speed | Baseline | **Same** | Quality ↑ Debt ↓ |


---

## Slide 15: Get Started

### 1. Create guideline
Start simple, iterate later

### 2. First test
Bug fix or small feature

### 3. Team share
What worked? What didn't?

**Shift-Left starts with one test.**
