# Demo data seeder

## Intent and scope

Replace the single-user `DatabaseSeeder` stub with a repeatable demo dataset for local development. Create 50 accounts split evenly into producer and retailer profiles, plus their companies and addresses, offers, purchases/sales, and substantial wishlists. Existing domain models make producer and retailer separate profiles, so purchases by retailers represent sales for producers; one account cannot hold both roles.

## Approach

- **Recommended:** create 25 producers and 25 retailers; connect all offers, purchases, and wishlist items to these accounts. Scope reruns to recognizable demo accounts so unrelated database records remain intact.
- Use 50 producers and 50 retailers to give each side more data, but this exceeds the requested user count.
- Truncate or reset the database before seeding; this is repeatable but can erase unrelated development data.

Choose the first approach for the requested 50 accounts and safer reruns.

## Proposed dataset and behavior

- Add a dedicated demo seeder and call it from `DatabaseSeeder`.
- Create 25 producer and 25 retailer users, each with company and address.
- Create 8 offers per producer, 10 purchases per retailer, and 20 wishlist entries per retailer (200 offers, 250 purchases, 500 wishlist entries). Ensure every producer has at least one sale and every retailer has purchases.
- Use the existing product categories, Brazilian produce names, and valid relations. Keep seeded accounts identifiable and remove/rebuild only their demo-owned records on rerun.
- No migrations, new packages, or dual-role account behavior.

## Calls left to implementation

Use existing factories where they preserve intended relationships; create purchases against the seeded offers and retailers rather than the standalone purchase factory, which creates unrelated records. Keep the dataset local-development oriented and give generated accounts one documented shared password for easy login.
