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

## Module Structure (20 files)

```
app/Modules/{Mod}/
├── {Mod}ServiceProvider.php           # Auto-discovered. Routes, lang, migrations, policies.
├── config.php                         # Permissions + labels. Read by ModulePermissionSeeder.
├── routes.php                         # API routes.
├── Models/{Entity}.php                # Eloquent + HasAuditUser + LogsActivity + casts.
├── Models/{Entity}Status.php          # Backed enum (active/inactive/etc.).
├── Data/{Entity}InputData.php         # Create input — required fields, fromRequest()
├── Data/{Entity}UpdateData.php        # Update input — all Optional, fromRequest()
├── Data/{Entity}Data.php              # Output / inter-Action transport, fromModel()
├── Data/{Entity}FilterData.php        # Index filters — all Optional, fromRequest()
├── Actions/Create{Entity}Action.php
├── Actions/Update{Entity}Action.php
├── Actions/List{Entity}Action.php
├── Http/Controllers/{Entity}Controller.php  # Resource (index, store, show, update). No destroy.
├── Http/Requests/{Entity}Request.php  # ONE FormRequest. rules() switches on method.
├── Http/Resources/{Entity}Resource.php # toArray() delegates to {Entity}Data::fromModel()
├── Policies/{Entity}Policy.php        # Permission checks only.
├── Database/Migrations/create_{entities}_table.php
├── Database/Factories/{Entity}Factory.php
├── Lang/es/messages.php               # UI messages only. One file per locale.
└── Tests/{Entity}Test.php             # ALL tests in one file. Uses describe() blocks.
```

## Quick Reference

### DTOs (4 per entity — base case)

- **`{Entity}InputData`** — Create input. Required fields only. No id/timestamps. `::fromRequest()`
- **`{Entity}UpdateData`** — Update input. All fields Optional. `::fromRequest()`. `toArray()` omits unset fields natively.
- **`{Entity}Data`** — Output + inter-Action transport. All fields. `::fromModel()`. NEVER used as write input.
- **`{Entity}FilterData`** — Index filters. All Optional. `::fromRequest(Request $request)`.

### Controller Pattern

```php
// index
public function index(Request $request, List{Entity}Action $action)
{
    Gate::authorize('viewAny', {Entity}::class);
    return {Entity}Resource::collection(
        $action({Entity}FilterData::fromRequest($request))
    );
}

// store
public function store({Entity}Request $request, Create{Entity}Action $action)
{
    Gate::authorize('create', {Entity}::class);
    return (new {Entity}Resource(
        $action({Entity}InputData::fromRequest($request))
    ))->response()->setStatusCode(201);
}

// show
public function show({Entity} ${entityLower})
{
    Gate::authorize('view', ${entityLower});
    return new {Entity}Resource(${entityLower});
}

// update
public function update({Entity}Request $request, {Entity} ${entityLower}, Update{Entity}Action $action)
{
    Gate::authorize('update', ${entityLower});
    return new {Entity}Resource(
        $action(${entityLower}, {Entity}UpdateData::fromRequest($request))
    );
}
```

- `response()->json()` is FORBIDDEN in Controllers
- QueryBuilder logic is FORBIDDEN in Controllers — use `List{Entity}Action`
- All output goes through `{Entity}Resource` — no exceptions

### Actions

Actions return models, not DTOs:

- **`Create{Entity}Action`** — receives `{Entity}InputData`, returns `{Entity}` model
- **`Update{Entity}Action`** — receives `{Entity}` + `{Entity}UpdateData`, returns `{Entity}` model
- **`List{Entity}Action`** — receives `{Entity}FilterData`, returns `LengthAwarePaginator`. QueryBuilder lives here — this is the ONLY place it is used

### Resources

- `{Entity}Resource` extends `JsonResource`
- `toArray()` delegates to `{Entity}Data::fromModel($this->resource)`
- Namespace: `App\Modules\{Mod}\Http\Resources`
- NEVER bypass the Resource — all Controller output goes through it

### Http/ Namespaces

Http/ uses subdirectories, not flat:

- Controllers: `App\Modules\{Mod}\Http\Controllers`
- Requests: `App\Modules\{Mod}\Http\Requests`
- Resources: `App\Modules\{Mod}\Http\Resources`

## Checklist

### Plan Tracking (MANDATORY)

- RULE: Read the plan file from `docs/plans/` at the START of every task. Never work from memory.
- RULE: Find the first unchecked `- [ ]` — that is your current task.
- RULE: After COMMIT (step 6), update the plan file: `- [ ]` → `- [x]` for completed steps.
- NEVER: Skip reading the plan. The file is the source of truth, not your memory.

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
   - Controller: thin — validate → `Gate::authorize()` → Action → Resource. NEVER `$this->authorize()`. NEVER `response()->json()`
   - Actions: `__invoke()`. Return models, not DTOs. All business logic here. NEVER in controllers/policies/DTOs
   - DTOs: 4 per entity — InputData, UpdateData, Data, FilterData. Pure typed data. NEVER `#[Rule()]`
   - Resources: `{Entity}Resource` — `toArray()` delegates to `{Entity}Data::fromModel()`. ALL output goes through Resources
   - FormRequest: ONE per entity. `rules()` via `match($this->method())`
   - Models: `HasAuditUser` + `LogsActivity`. Status enums. NEVER SoftDeletes
   - QueryBuilder: ONLY in `List{Entity}Action`. NEVER in Controllers
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

### Checkpoint (for plans with >5 tasks)

After every 3 completed tasks:
1. Re-read the plan file from disk
2. Count completed vs remaining
3. Report: "Checkpoint: N/Total tasks complete. Completed: [list]. Remaining: [list]. Continue?"
4. Wait for user confirmation before proceeding
- RULE: The checkpoint is part of the workflow. A task is not done until the plan file is updated.

## Test Pattern (Feature — mandatory)

Minimum `describe()` blocks: **Http, Actions, Policy, Model, Data** — follow test.stub for complete structure.

### Http — index (pagination meta)

```php
it('lists {entitiesLower} with pagination', function () {
    $user = User::factory()->create();
    $user->givePermissionTo('{moduleLower}.view');
    {Entity}::factory()->count(3)->create();

    $response = $this->actingAs($user)->getJson('/api/{entitiesLower}');

    $response->assertOk()
        ->assertJsonStructure(['data', 'meta']);
    expect($response->json('data'))->toHaveCount(3);
});
```

### Http — store (HTTP 201)

```php
it('creates a {entityLower}', function () {
    $user = User::factory()->create();
    $user->givePermissionTo('{moduleLower}.create');

    $this->actingAs($user)
        ->postJson('/api/{entitiesLower}', ['name' => 'Test'])
        ->assertCreated()
        ->assertJsonStructure(['data' => ['id', 'name', 'status']]);

    $this->assertDatabaseHas('{entitiesSnake}', ['name' => 'Test']);
});
```

### Http — authorization denied (403)

```php
it('rejects unauthorized create', function () {
    $user = User::factory()->create();

    $this->actingAs($user)
        ->postJson('/api/{entitiesLower}', ['name' => 'Forbidden'])
        ->assertForbidden();
});
```

### Actions — direct invocation

```php
it('creates a {entityLower} via action', function () {
    $data = {Entity}InputData::from([
        'name' => 'Test {Entity}',
    ]);

    $result = app(Create{Entity}Action::class)($data);

    expect($result)->toBeInstanceOf({Entity}::class)
        ->and($result->name)->toBe('Test {Entity}');

    $this->assertDatabaseHas('{entitiesSnake}', ['name' => 'Test {Entity}']);
});
```

### Http — validation (422)

```php
it('validates required fields on create', function () {
    $user = User::factory()->create();
    $user->givePermissionTo('{moduleLower}.create');

    $this->actingAs($user)
        ->postJson('/api/{entitiesLower}', [])
        ->assertUnprocessable()
        ->assertJsonValidationErrors(['name']);
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
