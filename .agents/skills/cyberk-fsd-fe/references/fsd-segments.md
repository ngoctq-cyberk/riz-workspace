# Code Patterns & Segments

## Query Pattern (Entities)
Location: `entities/{name}/api/{name}.queries.ts`

```typescript
import { orpc } from "@/shared/api";

type ListInputType = Exclude<Parameters<typeof orpc.posts.list.queryOptions>["0"]["input"], symbol>;

export const postQueries = {
  all: () => ["posts"] as const,

  // Standard: Explicit queryKey hierarchy with helpers
  lists: () => [...postQueries.all(), "list"] as const,
  list: (filter: ListInputType) => orpc.posts.list.queryOptions({ 
    input: filter,
    queryKey: [...postQueries.lists(), filter] 
  }),

  details: () => [...postQueries.all(), "detail"] as const,
  detail: (id: string) => orpc.posts.get.queryOptions({ 
    input: { id },
    queryKey: [...postQueries.details(), id]
  }),
};
```

## Mutation Pattern (Features)
Location: `features/{action}/api/use-{action}.ts`

```typescript
import { useMutation } from "@tanstack/react-query";
import { orpc, queryClient } from "@/shared/api";
import { postQueries } from "@/entities/post"; 

export function useCreatePost() {
  return useMutation(
    orpc.posts.create.mutationOptions({
      onSuccess: () => {
        // Invalidate ONLY lists, keeping cache for existing details if safe
        queryClient.invalidateQueries({ 
          queryKey: postQueries.lists() 
        });
      },
    })
  );
}
```

## File Segments
| Segment | Purpose | Example |
|---------|---------|---------|
| `ui/` | React Components | `ui/post-card.tsx` |
| `api/` | Data Fetching | `api/post.queries.ts`, `api/use-create-post.ts` |
| `model/` | Types, Stores, Schemas | `model/types.ts` |
| `lib/` | Helpers | `lib/post-utils.ts` |
