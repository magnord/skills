# Good and Bad Tests

## Good Tests

**Integration-style**: Test through real interfaces, not mocks of internal parts.

```typescript
// GOOD: Tests observable behavior
test("user can checkout with valid cart", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});
```

Characteristics:

- Tests behavior users/callers care about
- Uses public API only
- Survives internal refactors
- Describes WHAT, not HOW
- One logical assertion per test

## Bad Tests

**Implementation-detail tests**: Coupled to internal structure.

```typescript
// BAD: Tests implementation details
test("checkout calls paymentService.process", async () => {
  const mockPayment = jest.mock(paymentService);
  await checkout(cart, payment);
  expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
});
```

Red flags:

- Mocking internal collaborators
- Testing private methods
- Asserting on call counts/order
- Test breaks when refactoring without behavior change
- Test name describes HOW not WHAT
- Verifying through external means instead of interface

```typescript
// BAD: Bypasses interface to verify
test("createUser saves to database", async () => {
  await createUser({ name: "Alice" });
  const row = await db.query("SELECT * FROM users WHERE name = ?", ["Alice"]);
  expect(row).toBeDefined();
});

// GOOD: Verifies through interface
test("createUser makes user retrievable", async () => {
  const user = await createUser({ name: "Alice" });
  const retrieved = await getUser(user.id);
  expect(retrieved.name).toBe("Alice");
});
```

**Tautological tests**: Expected value restates the implementation, so the test passes by construction.

```typescript
// BAD: Expected value is recomputed the way the code computes it
test("calculateTotal sums line items", () => {
  const items = [{ price: 10 }, { price: 5 }];
  const expected = items.reduce((sum, i) => sum + i.price, 0);
  expect(calculateTotal(items)).toBe(expected);
});

// GOOD: Expected value is an independent, known literal
test("calculateTotal sums line items", () => {
  expect(calculateTotal([{ price: 10 }, { price: 5 }])).toBe(15);
});
```

**Output-copy tests**: the test-after version of a tautology. The expected value was produced by running the code, so the test locks in whatever the code does, bugs included.

```typescript
// BAD: Expected value pasted from a run of the code
test("formatDuration formats long durations", () => {
  expect(formatDuration(90061)).toMatchInlineSnapshot(`"1d 1h 1m 1s"`);
});

// GOOD: Expected value taken from the spec's worked example
test("formatDuration formats long durations", () => {
  // Spec: "90061 seconds displays as 1 day 1 h 1 min 1 s"
  expect(formatDuration(90061)).toBe("1 day 1 h 1 min 1 s");
});
```

## Choosing the level

```typescript
// BAD: Unit test for logic every checkout e2e test already runs through
test("applyDiscount subtracts ten percent", () => {
  expect(applyDiscount(100, "TEN")).toBe(90);
});

// GOOD: The e2e test for the checkout journey asserts the discounted total,
// and sabotaging applyDiscount turns it red, so no unit test is written.

// GOOD: Unit-level, with its reason recorded
// Why not higher: the parser has dozens of boundary cases; e2e would sample two.
test.each(durationCases)("parseDuration(%s)", (input, expected) => {
  expect(parseDuration(input)).toEqual(expected);
});
```
