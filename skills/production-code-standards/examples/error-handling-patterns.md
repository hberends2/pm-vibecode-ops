# Error Handling Patterns

Practical before/after examples demonstrating fail-fast error handling versus silent failure patterns.

## Example 1: Configuration Loading

### WRONG: Silent Fallback

```typescript
// Hides missing configuration - bugs appear as unexpected behavior later
function getDbConfig() {
  const config = process.env.DATABASE_URL || 'postgres://localhost:5432/dev';
  return config;
}

// In production: silently connects to wrong database
const db = connectToDatabase(getDbConfig());
```

### RIGHT: Fail-Fast Validation

```typescript
// Fails immediately with clear error message
function getDbConfig(): string {
  const config = process.env.DATABASE_URL;
  if (!config) {
    throw new ConfigurationError(
      'DATABASE_URL environment variable is required',
      { hint: 'Set DATABASE_URL in .env or environment' }
    );
  }
  return config;
}

// Application fails to start if misconfigured - caught in deployment
const db = connectToDatabase(getDbConfig());
```

**Why this matters**: Silent fallbacks create bugs that manifest far from their source. A missing env var in production might cause writes to a dev database, corrupting real data.

---

## Example 2: User Lookup

### WRONG: Null Coalescing to Guest

```typescript
async function getCurrentUser(sessionId: string): Promise<User> {
  const user = await userRepository.findBySession(sessionId);
  // Silently returns guest - authorization checks may pass incorrectly
  return user ?? { id: 'guest', role: 'anonymous', name: 'Guest User' };
}

// Later: "Guest User" appears in audit logs with no explanation
await auditService.log(getCurrentUser(sessionId), 'accessed_report');
```

### RIGHT: Explicit Session Validation

```typescript
async function getCurrentUser(sessionId: string): Promise<User> {
  const user = await userRepository.findBySession(sessionId);
  if (!user) {
    throw new AuthenticationError('Invalid or expired session', {
      code: 'SESSION_NOT_FOUND',
      sessionId: sessionId.substring(0, 8) + '...',
    });
  }
  return user;
}

// Caller must handle the error explicitly
try {
  const user = await getCurrentUser(sessionId);
  await auditService.log(user, 'accessed_report');
} catch (error) {
  if (error instanceof AuthenticationError) {
    return redirect('/login');
  }
  throw error;
}
```

**Why this matters**: Guest fallbacks bypass authorization. An expired session should redirect to login, not silently grant anonymous access.

---

## Example 3: API Response Handling

### WRONG: Empty Catch Block

```typescript
async function fetchUserProfile(userId: string) {
  try {
    const response = await api.get(`/users/${userId}`);
    return response.data;
  } catch (error) {
    // Swallows all errors - network issues, 404s, 500s all return null
    return null;
  }
}

// Caller has no idea why profile is null
const profile = await fetchUserProfile(userId);
if (!profile) {
  showGenericError(); // "Something went wrong" - unhelpful
}
```

### RIGHT: Typed Error Propagation

```typescript
async function fetchUserProfile(userId: string): Promise<UserProfile> {
  try {
    const response = await api.get(`/users/${userId}`);
    return response.data;
  } catch (error) {
    if (isAxiosError(error)) {
      if (error.response?.status === 404) {
        throw new NotFoundError('User profile not found', { userId });
      }
      if (error.response?.status === 403) {
        throw new ForbiddenError('Not authorized to view profile', { userId });
      }
    }
    // Re-throw with context for unexpected errors
    throw new ApiError('Failed to fetch user profile', {
      cause: error,
      userId
    });
  }
}

// Caller can show appropriate UI for each case
try {
  const profile = await fetchUserProfile(userId);
  renderProfile(profile);
} catch (error) {
  if (error instanceof NotFoundError) {
    showUserNotFound();
  } else if (error instanceof ForbiddenError) {
    showAccessDenied();
  } else {
    showErrorWithRetry(error);
  }
}
```

**Why this matters**: Different errors require different responses. A 404 should show "User not found", a 403 should show "Access denied", and a network error should offer retry.

---

## Example 4: Payment Processing

### WRONG: Fallback Amount

```typescript
function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => {
    // parseFloat returns NaN for invalid prices - || 0 hides the bug
    const price = parseFloat(item.price) || 0;
    return sum + price * item.quantity;
  }, 0);
}

// Customer charged $0 for items with corrupted price data
const total = calculateTotal(cart);
await chargeCustomer(total);
```

### RIGHT: Fail on Invalid Data

```typescript
function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => {
    const price = parseFloat(item.price);
    if (!Number.isFinite(price) || price < 0) {
      throw new ValidationError('Invalid item price', {
        itemId: item.id,
        price: item.price,
        message: 'Price must be a valid positive number',
      });
    }
    if (!Number.isInteger(item.quantity) || item.quantity < 1) {
      throw new ValidationError('Invalid item quantity', {
        itemId: item.id,
        quantity: item.quantity,
      });
    }
    return sum + price * item.quantity;
  }, 0);
}

// Data corruption is caught before charging
try {
  const total = calculateTotal(cart);
  await chargeCustomer(total);
} catch (error) {
  if (error instanceof ValidationError) {
    await alertOps('Cart data corruption detected', error);
    throw new CheckoutError('Unable to process order - please contact support');
  }
  throw error;
}
```

**Why this matters**: Financial calculations must never silently use fallback values. A `$0` charge or a negative amount indicates data corruption that requires investigation.

---

## Summary

| Pattern | Problem | Solution |
|---------|---------|----------|
| `value \|\| default` | Hides missing required data | Validate and throw with context |
| `catch (e) { }` | Swallows all error types | Catch specific errors, re-throw others |
| `return null` on error | Caller cannot distinguish error types | Throw typed errors with codes |
| Default objects | Bypasses validation/authorization | Require explicit handling |

**The principle**: Errors should be loud, specific, and caught as close to the source as possible. Silent failures create debugging nightmares and security holes.
