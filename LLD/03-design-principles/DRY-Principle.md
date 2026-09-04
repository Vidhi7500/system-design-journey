# DRY Principle

> **DRY = Don't Repeat Yourself**

The **DRY Principle** states that every piece of knowledge in a software system should have **one authoritative representation**.

In simple terms:

> **Define something once, and reuse it instead of duplicating it.**

DRY is not just about avoiding duplicate lines of code. It is about avoiding **duplicate knowledge and business logic** throughout the system.

---

# 1. What Does DRY Mean?

Consider a system with email validation.

Suppose three different modules contain:

```java
if (email != null && email.contains("@")) {
    // valid
}
```

Initially, this may look harmless.

But imagine the business rule changes:

> Emails must now end with `.com` or `.org`.

If the validation logic exists in three places, we need to modify all three.

```text
                Email Validation Rule
                         │
            ┌────────────┼────────────┐
            ↓            ↓            ↓
        Auth Module  Payment Module  Messaging
            │            │            │
            ↓            ↓            ↓
          Copy 1       Copy 2       Copy 3
```

If we update only two copies:

```text
Auth       → New Rule ✓
Payment    → New Rule ✓
Messaging  → Old Rule ✗
```

the application becomes inconsistent.

This is exactly the kind of problem DRY tries to prevent.

---

# 2. DRY Is About Knowledge, Not Just Code

A common misunderstanding is:

> "DRY means never write the same code twice."

That is too simplistic.

DRY is fundamentally about **duplicated knowledge**.

Knowledge can include:

### Business Rules

Example:

```text
Users must be at least 18 years old.
```

This rule should not be implemented differently in five different places.

---

### Configuration

Instead of:

```java
// Service A
timeout = 5000;

// Service B
timeout = 5000;

// Service C
timeout = 5000;
```

centralize the configuration:

```text
             Configuration
                  │
                  ↓
             timeout = 5000
              /    |    \
             ↓     ↓     ↓
         Service A B    C
```

---

### Data Models

If a `User` has:

```text
name
email
age
```

that model should have a single authoritative definition rather than being independently redefined across modules.

---

### Documentation

If an API field uses:

```text
ISO 8601 date format
```

that definition should ideally come from one authoritative source rather than being manually duplicated across multiple documents.

---

### Tests

Common test setup can also be shared when appropriate.

For example:

```java
createTestUser();
populateDatabase();
authenticateUser();
```

can be extracted into reusable helpers when the duplication represents genuine shared knowledge.

---

# 3. The Core Idea

Think of DRY like this:

```text
              One Piece of Knowledge
                       │
                       ↓
              ┌─────────────────┐
              │ Single Source   │
              │   of Truth      │
              └────────┬────────┘
                       │
              ┌────────┼────────┐
              ↓        ↓        ↓
            Module A Module B Module C
```

Instead of:

```text
Knowledge
   ├── Copy A
   ├── Copy B
   └── Copy C
```

we want:

```text
Knowledge
     │
     ↓
Single Source of Truth
     │
     ├── Module A
     ├── Module B
     └── Module C
```

---

# 4. Rule of Three

One of the most useful guidelines when applying DRY is the **Rule of Three**.

> **Don't rush to abstract code after seeing it only once or twice.**

If you see the same pattern three times, you have stronger evidence that it represents genuine shared knowledge.

```text
1 occurrence
     ↓
Probably unique

2 occurrences
     ↓
Maybe coincidence

3 occurrences
     ↓
Likely a reusable pattern
     ↓
Consider abstraction
```

### Why?

Two pieces of code may look similar today but evolve differently tomorrow.

For example:

```java
calculateShippingCost();
calculateDeliveryCost();
```

They may currently contain almost identical logic.

But perhaps later:

```text
Shipping → distance + weight
Delivery → distance + weight + delivery zone
```

If we prematurely combine them, we may create an abstraction that doesn't actually represent the domain.

---

# 5. Real-World Example — Email Validation

Suppose we have three modules:

```text
Authentication
Payment
Messaging
```

All three validate email addresses.

### Before DRY

```text
Authentication
    └── Email validation logic

Payment
    └── Email validation logic

Messaging
    └── Email validation logic
```

Now the business changes:

```text
Old:
Any email containing "@"

New:
Email must end with ".com" or ".org"
```

We now have to modify:

```text
Authentication ✓
Payment        ✓
Messaging      ✓
```

Missing one creates inconsistent behavior.

---

# 6. Applying DRY

Extract the common logic into a single component.

```text
                 EmailValidator
                       │
              ┌────────┼────────┐
              ↓        ↓        ↓
           Auth      Payment  Messaging
```

### Shared Validator

```java
public class EmailValidator {

    public boolean isValid(String email) {

        if (email == null) {
            return false;
        }

        return email.endsWith(".com")
            || email.endsWith(".org");
    }
}
```

Now the modules delegate validation to the same implementation.

```java
EmailValidator validator = new EmailValidator();

if (validator.isValid(email)) {
    // Continue
}
```

The validation rule now has a **single source of truth**.

If the rule changes again, we modify:

```text
EmailValidator
       │
       ↓
All modules automatically use
the updated rule
```

---

# 7. Problems Caused by Repetition

Repeated knowledge creates several problems as a codebase grows.

---

## 7.1 Harder to Maintain

Suppose the same logic exists in:

```text
File A
File B
File C
File D
File E
```

A requirement changes.

Now we have to:

1. Find every copy.
2. Modify every copy.
3. Verify every copy.
4. Test every copy.

The more copies there are, the greater the maintenance cost.

---

## 7.2 Higher Risk of Bugs

Suppose the original implementation contains:

```java
if (email != null && email.contains("@")) {
    // ...
}
```

Someone copies it but forgets the null check:

```java
if (email.contains("@")) {
    // ...
}
```

Now one module may throw a `NullPointerException` while the others don't.

```text
          Same Business Rule
                 │
       ┌─────────┼─────────┐
       ↓         ↓         ↓
    Module A  Module B  Module C
       ✓         ✓         ✗
```

Duplicated logic creates multiple opportunities for mistakes.

---

## 7.3 Bloated Codebase

Suppose a 10-line block is copied into 15 files.

```text
10 lines × 15 copies = 150 lines
```

But those 150 lines may represent only **one piece of knowledge**.

The duplication adds noise without adding new behavior.

---

## 7.4 Poor Test Coverage

Suppose email validation is implemented independently in three modules.

Ideally, we need to test:

```text
Auth validation
Payment validation
Messaging validation
```

Even though they are supposed to perform the same job.

With a shared validator:

```text
             EmailValidator
                   │
                   ↓
              One test suite
```

we can test the shared behavior once and keep the calling modules focused on their own responsibilities.

---

# 8. Copy-Paste Is a Red Flag

Copy-pasting code isn't automatically wrong.

But repeated copy-paste should make you stop and ask:

> **"If this logic changes tomorrow, will I remember every place where it exists?"**

If the answer is:

```text
No / Maybe
```

there may be duplicated knowledge that should be centralized.

```text
Copy → Paste → Paste → Paste
             ↓
       Warning Sign ⚠️
             ↓
       Look for shared knowledge
```

---

# 9. DRY Does NOT Mean "Never Repeat Code"

This is extremely important.

DRY is a **principle**, not an absolute rule.

Sometimes duplication is better than creating a bad abstraction.

> **Good duplication can be better than a wrong abstraction.**

---

# 10. When NOT to Apply DRY

## 10.1 Avoid Premature Abstraction

Don't immediately extract code just because two pieces look similar.

Example:

```java
calculateShippingCost();
calculateDeliveryCost();
```

They may look similar today but could represent different business concepts.

If we combine them too early:

```java
calculateCost();
```

we may end up with a generic method full of flags and conditions:

```java
calculateCost(type, zone, weight, priority, mode);
```

This can be harder to understand than the original duplication.

### Better approach

Let the duplication reveal whether the concepts are actually the same.

```text
Similar code
     ↓
Observe
     ↓
Does the same knowledge keep appearing?
     ↓
YES
     ↓
Extract abstraction
```

---

# 11. Tests Can Intentionally Contain Repetition

Tests should be **easy to understand in isolation**.

Consider:

```java
@Test
void shouldCreateUser() {

    User user = new User(
        "Alice",
        "alice@example.com"
    );

    // test...
}
```

Another test may contain:

```java
@Test
void shouldUpdateUser() {

    User user = new User(
        "Alice",
        "alice@example.com"
    );

    // test...
}
```

We could extract:

```java
createTestUser();
```

But now a reader has to navigate somewhere else to understand what kind of user is being created.

In tests:

> **Readability can be more valuable than eliminating every small duplication.**

A little repetition is acceptable if it makes the test's intent immediately obvious.

---

# 12. Don't Abstract Trivial Code

Consider:

```java
x + 1
```

Suppose it appears twice.

Creating:

```java
MathUtils.addOne(x);
```

just to avoid duplication would be unnecessary.

You have technically removed duplication, but the code has become worse.

### Before

```java
x + 1
```

### After

```java
MathUtils.addOne(x)
```

Now the developer has to:

```text
Read MathUtils.addOne()
        ↓
Open another file
        ↓
Understand implementation
        ↓
Discover it simply does x + 1
```

The abstraction adds more complexity than value.

> **DRY should improve the design, not become an excuse for unnecessary abstraction.**

---

# 13. Practical Example — Notification System

Consider an application with three services:

```text
OrderService
ShippingService
SupportService
```

All three need to send notifications.

Initially, each service contains:

1. Message formatting
2. Notification sending

---

## Before DRY

```text
┌─────────────────┐
│  OrderService   │
│                 │
│ Format message  │
│ Send message    │
└─────────────────┘

┌─────────────────┐
│ ShippingService │
│                 │
│ Format message  │
│ Send message    │
└─────────────────┘

┌─────────────────┐
│ SupportService  │
│                 │
│ Format message  │
│ Send message    │
└─────────────────┘
```

We now have duplicated knowledge.

---

# 14. Apply DRY

Separate the shared responsibilities.

```text
                  Notification System
                         │
              ┌──────────┴──────────┐
              ↓                     ↓
       MessageFormatter     NotificationSender
              │                     │
              ↓                     ↓
        Format message          Send message
```

The services now use these shared components:

```text
OrderService ───────┐
                    │
ShippingService ────┼──→ MessageFormatter
                    │
SupportService ─────┘

OrderService ───────┐
                    │
ShippingService ────┼──→ NotificationSender
                    │
SupportService ─────┘
```

---

## MessageFormatter

```java
public class MessageFormatter {

    public String format(String message) {
        return "[Notification] " + message;
    }
}
```

---

## NotificationSender

```java
public class NotificationSender {

    public void send(String message) {
        // Call external notification API
    }
}
```

---

## OrderService

```java
public class OrderService {

    private final MessageFormatter formatter;
    private final NotificationSender sender;

    public OrderService(
            MessageFormatter formatter,
            NotificationSender sender) {

        this.formatter = formatter;
        this.sender = sender;
    }

    public void notifyUser(String message) {

        String formatted = formatter.format(message);

        sender.send(formatted);
    }
}
```

`ShippingService` and `SupportService` can use the same shared components.

---

# 15. Why This Design Works

### 1. Single Source of Truth

Message formatting exists in:

```text
MessageFormatter
```

If the message format changes, we modify one class.

---

### 2. Single Sending Logic

Notification delivery exists in:

```text
NotificationSender
```

If the external API changes, we update one place.

For example:

```text
Old API
   ↓
NotificationSender
   ↓
New API
```

The services don't need to know the implementation details.

---

### 3. Services Stay Focused

```text
OrderService
    ↓
Orders

ShippingService
    ↓
Shipping

SupportService
    ↓
Support
```

They don't need to know:

```text
How messages are formatted
How HTTP requests are made
How authentication works
How retries are implemented
```

Those details belong to dedicated components.

---

### 4. Easier Testing

We can test:

```text
MessageFormatter
       ↓
Unit Tests

NotificationSender
       ↓
Unit Tests
```

Then mock them while testing:

```text
OrderService
ShippingService
SupportService
```

---

### 5. Easier Extension

Suppose we add:

```text
BillingService
```

It can reuse:

```text
MessageFormatter
NotificationSender
```

without duplicating the implementation.

---

# 16. DRY and Single Responsibility

DRY often works well with the **Single Responsibility Principle (SRP)**.

Instead of:

```text
OrderService
├── Order logic
├── Message formatting
└── Notification API
```

we separate responsibilities:

```text
OrderService
└── Order logic

MessageFormatter
└── Message formatting

NotificationSender
└── Notification delivery
```

This results in smaller, focused components.

---

# 17. DRY in Low-Level Design

In LLD interviews, look for repeated:

### Business Logic

```text
validateUser()
calculateDiscount()
calculateFee()
checkEligibility()
```

If the same rule appears across multiple classes, consider centralizing it.

---

### Constants

Avoid:

```java
if (age >= 18) { ... }
```

in multiple places.

Instead:

```java
public static final int MINIMUM_AGE = 18;
```

and reuse the same source of truth.

---

### Validation

Instead of:

```text
AuthService → validation
PaymentService → validation
OrderService → validation
```

consider:

```text
                 Validator
                    │
          ┌─────────┼─────────┐
          ↓         ↓         ↓
        Auth     Payment    Order
```

---

### Shared Calculations

If the same calculation represents the same business concept, centralize it.

For example:

```text
Tax calculation
Discount calculation
Shipping calculation
```

---

# 18. DRY Decision Checklist

Before extracting duplicated code, ask:

```text
Is this actually the same knowledge?
             │
        ┌────┴────┐
       NO        YES
       ↓           ↓
 Keep separate   How many times?
                   │
             ┌─────┴─────┐
             ↓           ↓
           1–2 times   3+ times
             │           │
             ↓           ↓
         Wait/observe  Consider
                       abstraction
```

Then ask:

1. Is the duplicated logic representing the **same business rule**?
2. Is it likely to change for the **same reason**?
3. Will centralizing it make the code easier to understand?
4. Will the abstraction reduce complexity?
5. Does the abstraction have a clear responsibility?
6. Would the abstraction make tests harder to read?

If the abstraction makes the code harder to understand, don't force DRY.

---

# 19. DRY vs Overengineering

### Good DRY

```text
Repeated business rule
        ↓
Shared abstraction
        ↓
Simpler system
```

### Bad DRY

```text
Tiny duplication
        ↓
Unnecessary abstraction
        ↓
More classes
        ↓
More indirection
        ↓
Harder code
```

The goal is **not minimum lines of code**.

The goal is:

> **One source of truth with a clean and understandable design.**

---

# 20. Quick Comparison

| Situation                                      | DRY?             |
| ---------------------------------------------- | ---------------- |
| Same business rule repeated 5 times            | ✅ Yes            |
| Same validation in multiple services           | ✅ Yes            |
| Same configuration repeated everywhere         | ✅ Yes            |
| Same complex calculation repeated              | ✅ Yes            |
| Two similar pieces that may evolve differently | ⚠️ Wait          |
| Small readable duplication in tests            | ⚠️ Often okay    |
| `x + 1` repeated twice                         | ❌ Don't abstract |
| Copy-pasted business logic                     | ⚠️ Investigate   |

---

# 21. Common Interview Questions

### Q1. What is DRY?

> DRY stands for **Don't Repeat Yourself**. It states that every piece of knowledge in a system should have a single authoritative representation instead of being duplicated across multiple places.

---

### Q2. Is DRY only about duplicate code?

**No.**

DRY is primarily about **duplicate knowledge**.

It applies to:

* Business rules
* Configuration
* Data models
* Documentation
* Test setup
* Reusable behavior

---

### Q3. Should we always remove duplicate code?

**No.**

DRY is a guideline, not an absolute rule.

Sometimes small duplication is preferable to creating a complex or incorrect abstraction.

---

### Q4. What is the Rule of Three?

> The Rule of Three suggests waiting until a pattern appears approximately three times before extracting it into a shared abstraction.

The first two occurrences may be coincidental, while three occurrences provide stronger evidence of a genuine common pattern.

---

### Q5. Why is copy-pasting code dangerous?

Because when the original logic changes, every copy must be updated.

Missing one copy can lead to:

```text
Inconsistent behavior
       ↓
Production bugs
       ↓
Hard debugging
```

---

### Q6. What is a Single Source of Truth?

A **Single Source of Truth (SSOT)** means a particular piece of knowledge is defined in one authoritative location.

Other parts of the system reference or use that source rather than maintaining independent copies.

---

# 22. DRY Mental Model

```text
                 DRY
                  │
                  ↓
       Don't Repeat Yourself
                  │
                  ↓
       Identify duplicated knowledge
                  │
                  ↓
        Create one source of truth
                  │
                  ↓
            Reuse it
                  │
                  ↓
        Easier maintenance
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
      Fewer     Fewer     Cleaner
      bugs      changes     code
```

---

# 23. Final Takeaways

> **DRY is about eliminating duplicated knowledge, not blindly eliminating every repeated line of code.**

Remember:

1. **DRY = Don't Repeat Yourself.**
2. Each piece of knowledge should have a **single authoritative representation**.
3. DRY applies to more than code—it includes business rules, configuration, models, documentation, and tests.
4. Repetition increases **maintenance cost and bug risk**.
5. Copy-paste is a useful **warning sign**.
6. Use the **Rule of Three** before creating an abstraction.
7. Don't create abstractions prematurely.
8. Small repetition can sometimes be better than a bad abstraction.
9. Tests may intentionally contain some duplication for readability.
10. Avoid abstractions for trivial code.
11. In LLD, look for duplicated **business rules and responsibilities**.
12. Good DRY creates a **single source of truth without adding unnecessary complexity**.

---

## One-Line Interview Answer

> **The DRY principle states that every piece of knowledge in a software system should have a single, unambiguous, authoritative representation, so that changes need to be made in one place rather than across multiple duplicated implementations.**

---

## Quick Revision

```text
             DUPLICATION
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
  Same knowledge?       Different knowledge?
        │                   │
       YES                  YES
        ↓                   ↓
  Consider DRY          Keep separate
        │
        ↓
  Rule of Three
        │
        ↓
  Create abstraction
        │
        ↓
 Single Source of Truth
```

### Remember

**DRY does NOT mean:**

```text
"Never write the same code twice."
```

It means:

```text
"Don't represent the same knowledge
in multiple independent places."
```
