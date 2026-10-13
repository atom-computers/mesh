# Mesh

Open-source cloud framework that replaces AWS/GCP/Azure with a stack you can run on your own infrastructure and eliminate vendor lock-in from your codebase.

```typescript
// Define a PostgreSQL table
const products = table('Products', Product, {
  id: index()
})

// Define an IAM role
export const readProducts = role('ReadProducts', {
  policies: [allow('table/Products:read')],
})

// Define a GraphQL query
export const getProduct = query((id: string) => 
  products.get({ id }), 
  { role: readProducts }
)

// Define the API
api({
  schema: 'schema.gql',
  types: { Product }
  queries: { getProduct }
})
```

Mesh supports all the usual cloud primitives:

### Features

- State machines
- Functions
- Object storage
- Queues
- Streaming
- GraphQL
- Scheduler
- IAM

Example (Queue + Cron + IAM):

```typescript
// Define a queue
const Tasks = queue('Tasks', Task, {
  id: deduplicate()
}, {
  retry: { attempts: 3, backoff: "5m" },
  deadLetter: 'TasksDead'
})

// Define a cron
cron('IncrementCount', () => {
  // Enqueue a task
  await Tasks.enqueue({
    id: randomUUID(),
    body: { fn: 'incrementCount' }
  })
}, {
  expression: '0 5 * * *', role: taskEnqueuer
})

// Define a role
const taskEnqueuer = role('TaskEnqueuer', {
  policies: [allow('queue/Tasks:send')]
})

const onTask = (task: Task) => {
  if(task.body.fn == 'incrementCount'){
    incrementCount()
  }
}

Tasks.addTrigger(onTask)
```

