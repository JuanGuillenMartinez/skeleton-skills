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

0. **Read `backend/ARCHITECTURE.md`** — verify plan alignment with current conventions before implementing.
1. **Read the task** from the plan
2. **RED — Write failing test:**
   - Location: `Modules/{Mod}/Tests/{Entity}Test.php` (single file)
   - Structure: `describe()` blocks — Http, Actions, Policy, Model, Data
   - Framework: Pest (functional syntax, NOT PHPUnit)
   - Run: `./vendor/bin/sail artisan test --filter={TestName}`
   - **Verify: FAIL**
3. **GREEN — Minimal implementation:**
   Implement per ARCHITECTURE.md. Quick-ref:
   - Controller: thin — validate → `Gate::authorize()` → Action → response. NEVER `$this->authorize()`
   - Actions: `__invoke()`. All business logic here. NEVER in controllers/policies/DTOs
   - DTOs: pure typed data with `Optional` fields. NEVER `#[Rule()]` or validation
   - FormRequest: ONE per entity. `rules()` via `match($this->method())`
   - Models: `HasAuditUser` + `LogsActivity`. Status enums. NEVER SoftDeletes
   - Inter-module: only `Contracts/` + `Data/`. NEVER `Services/`, `Actions/` from other modules
   - No destroy endpoints. Status transitions only
   - Permissions: `{module_snake_case}.{action}`. NEVER entity name as prefix
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
