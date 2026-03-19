# Import Rules

## The Golden Rule
**Import ONLY from layers below.**
**NEVER import from the SAME layer.**

## Allowed Imports

| Layer | Can Import From |
|-------|-----------------|
| `app` | `screens`, `widgets`, `features`, `entities`, `shared` |
| `screens` | `widgets`, `features`, `entities`, `shared` |
| `widgets` | `features`, `entities`, `shared` |
| `features` | `entities`, `shared` |
| `entities` | `shared` (via segment barrels `@/shared/api`, or direct for `shadcn/`/`ui/`) |
| `shared` | (No internal imports) |

## Cross-Slice Access
If an Entity needs another Entity (e.g., `Post` needs `User`):
1. **Prefer Props**: Pass data from parent (Screen/Widget).
2. **Interface**: Define required interface in consumer.
3. **Slots**: Pass component slot from parent.
