---
name: skeleton-laravel-implementing
description: >
  Use when implementing a backend task from a plan. Enforces TDD with Pest
  in the modular monolith architecture. RED→GREEN→REFACTOR→GUARD→COMMIT.
---

# Skeleton Laravel Implementing

TDD implementation for backend tasks. Every task follows RED→GREEN→REFACTOR→GUARD→COMMIT.

**Announce:** "Using skeleton:laravel-implementing for this backend task."

**Prerequisite:** Task definition from plan (files, code, expected output).

## Execution Environment

PHP is NOT on host. All commands use `./vendor/bin/sail`:

| Instead of | Use |
|------------|-----|
| `php artisan test` | `./vendor/bin/sail artisan test` |
| `./vendor/bin/pint` | `./vendor/bin/sail php ./vendor/bin/pint` |
| `./vendor/bin/phpstan analyse` | `./vendor/bin/sail php ./vendor/bin/phpstan analyse` |
| `composer require X` | `./vendor/bin/sail composer require X` |

## Checklist

1. **Read the task** from the plan
2. **RED — Write failing test:**
   - Location: `Modules/{Mod}/Tests/{Entity}Test.php` (single file, `describe()` blocks)
   - Framework: Pest (functional syntax, NOT PHPUnit)
   - Run: `./vendor/bin/sail artisan test --filter={TestName}`
   - **Verify: FAIL**
3. **GREEN — Minimal implementation:**
   - Actions: invocable (`__invoke()`)
   - Data: ONE `{Entity}Data.php` with Optional fields
   - FormRequests: ONE `{Entity}Request.php`, `rules()` via `match($this->method())`
   - Authorization: `Gate::authorize()` (not `$this->authorize()`)
   - No SoftDeletes — status enums
   - i18n: `__('mod::messages.key')`
   - Run: `./vendor/bin/sail artisan test --filter={TestName}`
   - **Verify: PASS**
4. **REFACTOR — Clean if needed.** Run full suite: `./vendor/bin/sail artisan test` — ALL PASS
5. **GUARD:** `./vendor/bin/sail php ./vendor/bin/pint` (auto-fix style)
6. **COMMIT:** `cd backend && git add [files] && git commit -m "feat(mod): description"`
7. **Next:** Run `skeleton:validating task` then proceed to next task

## Test Pattern (Feature — mandatory)

```php
it('creates a product', function () {
    $user = User::factory()->create();
    $user->givePermissionTo('catalog.create');

    $this->actingAs($user)
        ->postJson('/api/products', ['name' => 'Test', 'sku' => 'T-1', 'price' => 100])
        ->assertCreated()
        ->assertJsonStructure(['data' => ['id', 'name', 'price']]);

    $this->assertDatabaseHas('products', ['name' => 'Test']);
});
```

## Anti-Rationalizations

| Temptation | Reality |
|------------|---------|
| "I'll write the test after" | No. Test FIRST. Always. |
| "The migration can't be TDD'd" | The Feature test that uses the table IS the test. RED: table doesn't exist. GREEN: migration. |
| "It's just wiring, no logic to test" | Wiring is where integration bugs live. Feature tests verify wiring. |

## Transitions

| Condition | Destination |
|-----------|-------------|
| Task committed | `skeleton:validating task` → next task |
| RED phase passes (shouldn't) | Rewrite the test |
| GREEN requires >50 lines | Task too large — decompose |
| All tasks done | `skeleton:validating final` |
