---
name: skeleton-laravel-implementing
description: >
  Use when implementing a backend task from a plan. Enforces TDD with Pest
  in the modular monolith architecture (Modules/{Mod}/) with Pint code
  style and PHPStan type checking. Never skip the RED phase.
---

# Skeleton Laravel Implementing

TDD implementation for backend tasks. RED→GREEN→REFACTOR→GUARD→COMMIT for every task.

**Announce:** "Using skeleton:laravel-implementing for this backend task."

**Prerequisite:** Task definition from plan (files, code, expected output).

## Execution Environment

PHP is not available on the host machine. ALL commands must be prefixed with `./vendor/bin/sail`:

| Instead of | Use |
|------------|-----|
| `php artisan test` | `./vendor/bin/sail artisan test` |
| `php artisan test --filter=X` | `./vendor/bin/sail artisan test --filter=X` |
| `./vendor/bin/pint` | `./vendor/bin/sail php ./vendor/bin/pint` |
| `./vendor/bin/phpstan analyse` | `./vendor/bin/sail php ./vendor/bin/phpstan analyse` |
| `composer require X` | `./vendor/bin/sail composer require X` |

**Detection:** Run `which php`. If not found, use Sail for all commands.

## Checklist

1. **Read the current task** from the plan (paths, code, expected output)
2. **Verify Sail/Docker runs:**
   - Check: `cd backend && ./vendor/bin/sail ps`
   - If containers not running: `./vendor/bin/sail up -d`
   - If `compose.yaml` doesn't exist: `docker run --rm -v "$(pwd):/opt" -w /opt laravelsail/php84-composer:latest php artisan sail:install --with=pgsql,redis`
   - Wait for containers to be healthy before proceeding
3. **RED — Write failing test:**
   - Location: `Modules/{Mod}/Tests/Feature/Http/` (endpoints) or `Modules/{Mod}/Tests/Unit/Actions/` (complex logic)
   - Framework: Pest (functional syntax, NOT PHPUnit classes)
   - Use factories for test data
   - Run: `cd backend && php artisan test --filter={TestName}`
   - **Verify: FAIL** with the expected message
4. **GREEN — Minimal implementation:**
   - Create files per plan (Action, DTO, Controller, FormRequest, etc.)
   - Respect CLAUDE.md conventions:
     - Actions: invocable (`__invoke()`)
     - DTOs: spatie/laravel-data, separated (Create/Update/Output)
     - No SoftDeletes — status with enums
     - Permissions: `{mod}.{action}`
     - i18n: `__('mod::messages.key')`
     - Validation: ONLY in Form Requests
     - Controllers: validate → authorize → Action → response (no business logic)
     - Authorization: Use `Gate::authorize()` (not `$this->authorize()` — Laravel 12's base Controller no longer includes AuthorizesRequests trait)
   - Run: `cd backend && php artisan test --filter={TestName}`
   - **Verify: PASS**
5. **REFACTOR — Clean if needed:**
   - Remove duplication, improve names
   - Do NOT add functionality
   - Run: `cd backend && php artisan test` (full suite)
   - **Verify: ALL PASS**
6. **Code style:** `cd backend && ./vendor/bin/pint`
7. **Commit:**
   ```bash
   cd backend && git add [specific files] && git commit -m "feat(mod): description"
   ```

## Test Patterns

### Feature test for endpoint (MANDATORY)

```php
it('creates a product', function () {
    $user = User::factory()->create();
    $user->givePermissionTo('catalog.create');

    $response = $this->actingAs($user)
        ->postJson('/api/products', [
            'name' => 'Test Product',
            'price' => 100,
        ]);

    $response->assertCreated()
        ->assertJsonStructure(['data' => ['id', 'name', 'price']]);
    $this->assertDatabaseHas('products', ['name' => 'Test Product']);
});

it('rejects unauthorized users', function () {
    $user = User::factory()->create();
    // No permission given

    $this->actingAs($user)
        ->postJson('/api/products', ['name' => 'Test'])
        ->assertForbidden();
});

it('validates required fields', function () {
    $user = User::factory()->create();
    $user->givePermissionTo('catalog.create');

    $this->actingAs($user)
        ->postJson('/api/products', [])
        ->assertUnprocessable()
        ->assertJsonValidationErrors(['name', 'price']);
});
```

### Unit test for complex Action (only if non-trivial logic)

```php
it('throws InsufficientStockException when stock is not available', function () {
    $inventory = Mockery::mock(InventoryServiceContract::class);
    $inventory->shouldReceive('hasStock')->andReturn(false);

    $action = new CreateSaleAction($inventory);

    expect(fn () => $action(new CreateSaleData(...)))
        ->toThrow(InsufficientStockException::class);
});
```

## Red Flags — Anti-Rationalizations

| Agent's excuse | Reality |
|----------------|---------|
| "It's just a DTO, it doesn't need a test" | DTOs are tested indirectly in Feature tests of the endpoint that uses them. Write the Feature test first. |
| "The migration can't be tested with TDD" | The migration is tested with the Feature test that uses the table. RED: test that uses the table → FAIL (table doesn't exist). GREEN: migration + model. |
| "I'm just wiring things up, there's no logic" | "Wiring" is where integration bugs live. The Feature test verifies the wiring works. |
| "I'll write the test after" | No. Delete the code. Start over. Test goes FIRST. Always. |
| "The test is obvious, it doesn't add value" | If it's obvious, writing it takes 30 seconds. 30 seconds of writing → permanent confidence. No test → temporary hope. |
| "I don't know how to test this" | Signal that the design needs review. Stop and think. If it's not testable, it's not good design. |
| "It's a trivial feature" | Trivial features with trivial tests = accumulated confidence. Trivial features without tests = accumulated debt. |
| "The framework already tests this" | Laravel tests Laravel. You test YOUR use of Laravel. Your FormRequest, your Action, your Policy. |
| "I need to create infrastructure first" | No. Test first. Infrastructure emerges from the test. That's TDD. |
| "The seeder doesn't need a test" | Correct. Seeders aren't tested. But everything else is. Don't use the seeder as an excuse for other files. |
| "spatie/laravel-data handles it automatically, no test needed" | The DTO abstraction hides complexity. Tests caught real issues with enum casting and Optional field serialization. Test the behavior, not the framework. |

## Correct Example

```
Task: Create CreateProductAction

1. ✅ Write test: it('creates a product', ...) → POST /api/products
2. ✅ Run test → FAIL: "Route [POST /api/products] not defined"
3. ✅ Create: Route + Controller + FormRequest + Action + DTO
4. ✅ Run test → PASS
5. ✅ Run full suite → ALL PASS
6. ✅ pint → no changes
7. ✅ Commit: "feat(catalog): add create product endpoint"
```

## Incorrect Example

```
Task: Create CreateProductAction

1. ❌ Create Action first (no test)
2. ❌ Create Controller, Route, FormRequest
3. ❌ Write test after everything is built
4. ❌ Test passes on first run (not TDD — no RED phase)
5. ❌ Skip pint
6. ❌ Commit everything at end of session
```

## Transitions

| Condition | Destination |
|-----------|-------------|
| Task completed (commit done) | `skeleton:architecture-guarding` (backend mode) |
| RED phase test passes (shouldn't) | Rewrite the test |
| GREEN requires >50 lines | Task too large, return to `skeleton:planning` to decompose |
| Full suite fails after GREEN | Investigate before continuing |
| Task blocked | Escalate to human |
