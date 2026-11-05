# Demo data seeder Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make `php artisan db:seed` create 50 repeatable demo accounts and a dense, internally consistent Cultiva dataset.

**Architecture:** Add `DemoDataSeeder`, called from `DatabaseSeeder`, to replace the current single-user stub. Seed 25 producers and 25 retailers, build relationships through existing factories, and clean/recreate only demo-owned records in a transaction so reruns preserve unrelated data.

**Tech Stack:** Laravel 13, Eloquent factories, PostgreSQL/PostGIS, PHPUnit.

**Spec:** `docs/superpowers/specs/2026-11-05-demo-data-seeder-design.md`

## Global Constraints

- Exactly 50 demo users: 25 producer profiles and 25 retailer profiles.
- Generate 8 offers per producer, 10 purchases per retailer, and 20 wishlist entries per retailer.
- Reruns preserve records unrelated to demo identities.
- No migrations, dependencies, or dual-role profile behavior.
- Run PHP, Artisan, Composer, and tests inside the `php` Docker Compose service; tests use `cultiva_test` only.

## Review Focus

- Existing demo users and their dependent rows on rerun: assert row counts stay fixed after two runs.
- Unrelated users and their related records: assert a sentinel user remains after two runs.
- Purchase stock and producer/retailer linkage: assert purchases reference active demo offers, and every producer has a sale.
- Wishlist unique constraint: assert 20 distinct product IDs per retailer.
- Categories missing: rely on migration-provided categories and ensure seeder doesn't create duplicate category names.

---

### Task 1: Add and wire repeatable demo data seeder

**Files:**
- Create: `backend/tests/Feature/Database/Seeders/DemoDataSeederTest.php`
- Create: `backend/database/seeders/DemoDataSeeder.php`
- Modify: `backend/database/seeders/DatabaseSeeder.php`

**Interfaces:**
- `DemoDataSeeder::run(): void` owns cleanup and generation of the full demo dataset.
- `DatabaseSeeder::run(): void` calls `DemoDataSeeder` and no longer creates the one-off `test@example.com` user.

- [ ] **Step 1: Write the failing feature test**

  Test `DatabaseSeeder` creates 25 producers and 25 retailers, 200 offers, 250 purchases, and 500 wishlist items; every producer has at least one purchase, every retailer has ten purchases and twenty wishlist items. Add an unrelated sentinel user, call `DatabaseSeeder` twice, and assert all demo counts remain fixed and the sentinel remains.

- [ ] **Step 2: Run the test to verify it fails**

  Run: `docker compose exec php php artisan test --filter=DemoDataSeederTest`
  Expected: fail because current `DatabaseSeeder` creates only `test@example.com`.

- [ ] **Step 3: Implement `DemoDataSeeder::run(): void`**

  In one transaction, locate existing demo users by the stable `demo.producer.NN@cultiva.test` and `demo.retailer.NN@cultiva.test` identities; delete their purchases, offers, wishlist entries, profile rows, companies, and users in foreign-key-safe order. Recreate the 50 users, companies with addresses, producer/retailer profiles, 8 active produce offers per producer using migration-seeded categories, 10 purchases per retailer linked to those offers with quantities within stock, and 20 distinct wishlist items per retailer. Use a shared local-development password documented in the seeder and block execution in production. Create purchases directly because `PurchaseFactory` creates a separate offer as a side effect. Keep all generated rows connected to these identities.

- [ ] **Step 4: Wire `DatabaseSeeder` to call `DemoDataSeeder`**

  Replace its single `UserFactory` create call with `$this->call(DemoDataSeeder::class)`.

- [ ] **Step 5: Run the feature test to verify it passes**

  Run: `docker compose exec php php artisan test --filter=DemoDataSeederTest`
  Expected: PASS against isolated `cultiva_test`.

- [ ] **Step 6: Review the diff and rerun the targeted test**

  Confirm only the planned files changed and rerun the same command after any fixes.
