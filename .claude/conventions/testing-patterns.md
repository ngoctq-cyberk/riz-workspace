# Testing Patterns

## Conventions

- Framework: Jest
- Pattern: Arrange-Act-Assert
- Naming: `inputX`, `mockX`, `actualX`, `expectedX`
- Write unit tests for public functions
- Write e2e tests for each API module
- Test file pattern: `*.spec.ts`

## E2E Test Pattern

Use `TestContext` and `testHelper` from `@app/spec/test.helper`:

```typescript
describe('TodoSpec', () => {
  let tc: TestContext
  let uc: UserContextTestType

  beforeAll(async () => {
    tc = await testHelper.createContext({ imports: [TodoModule] })
    uc = await tc.generateAcount()  // Generate test user with auth
  })
  afterAll(async () => await tc?.clean())

  test('Create:Success', async () => {
    const res = await uc.request((r) => r.post('/todo')).send({ title: 'Test' } as CreateTodoDto)
    expect(res).toBeCreated()  // Custom matcher
  })
})
```

## Custom Matchers

`toBeOK()`, `toBeCreated()`, `toBeBad(pattern?)`, `toBe404()`
