# Layering

In our projects we follow the pattern of **Container/Content** controlled by MobX state to achieve clear separation of concerns, better testability, and scalable architecture.
To follow this approach we single out independent sections on the page which have their own state and network calls.

Below is a typical folder structure for this approach. In this case _auth_ is the only section on the page, so we don't need to create a _sections_ folder:

```bash
├── pages
│   ├── auth
│   │   └── state
│   │       └── AuthState.cy.ts
│   │       └── AuthState.ts
│   │       └── AuthStateContext.ts
│   │   └── AuthContainer.tsx
│   │   └── AuthContent.tsx
│   │   └── AuthPage.scss
│   │   └── AuthPage.tsx
```

> Page.tsx file only wraps the Container in context provider, e.g.:

```javascript
AuthPage.tsx

export function AuthPage() {
  const authState = useMemo(
    () => new AuthState(),
    [],
  )

  return (
    <AuthStateContext.Provider value={authState}>
      <AuthContainer />
    </AuthStateContext.Provider>
  )
}
```

In case there are several sections on the page, create a _sections_ folder:
```bash
├── pages
│   ├── todos
│   │   ├── sections
│   │   │   ├── list
│   │   │   │   └── state
│   │   │   │   │   └── ToDosState.cy.ts
│   │   │   │   │   └── ToDosState.ts
│   │   │   │   │   └── ToDosStateContext.ts
│   │   │   │   └── ToDosContainer.tsx
│   │   │   │   └── ToDosContent.tsx
│   │   │   │   └── ToDosContents.scss
│   │   │   └── new-to-do
│   │   │       └── state
│   │   │       │   └── NewToDoState.cy.ts
│   │   │       │   └── NewToDoState.ts
│   │   │       │   └── NewToDoStateContext.ts
│   │   │       └── NewToDoContainer.tsx
│   │   │       └── NewToDoContent.tsx
│   │   │       └── NewToDoContents.scss
│   │   └── ToDosPage.tsx
```

In case the content gets too large, we add a components directory and extract separate sections there:

```bash
├── pages
│   ├── add-book
│   │   └── components
│   │   │   └── radio-group
│   │   │   │   └── RadioGroup.tsx
│   │   │   │   └── RadioGroup.scss
│   │   │   └── counter-input
│   │   │       └── CounterInput.tsx
│   │   │       └── CounterInput.scss
│   │   └── state
│   │   │   └── AddBookState.cy.ts
│   │   │   └── AddBookState.ts
│   │   │   └── AddBookStateContext.ts
│   │   └── AddBookContainer.tsx
│   │   └── AddBookContent.tsx
│   │   └── AddBookPage.scss
│   │   └── AddBookPage.tsx
```


## Pros of Container/Content Pattern with MobX

1. Separation of concerns
- Container handles business logic and API calls
- Content handles presentation, rendering on UI 
2. Testability
- Container can be tested with integration tests (API mocking)
- Content can be tested with unit tests (props, rendering)
- State can be tested independently with Cypress
3. State management
- Centralized state logic
- Automatic re-rendering when state changes
4. Maintainability 
- Easier to debug and refactor

## Cons of Container/Content Pattern with MobX

1. Boilerplate code
2. More files to navigate
3. Simple components may become over-engineered

## When to Use
This approach is worth following for complex components with state and API calls. Also if testing is of a high priority on the project, the layering approach is quite useful to separate different testing types. 
In case of simple static components this approach might be an overkill. 

## Alternatives
[Feature-Sliced Design](https://fsd.how/docs/get-started/overview/) is an architectural methodology for scaffolding front-end applications, based on layers, slices, and segments:

![Layers, slices, and segments in FSD](/frontend/images/fsd.webp)

**Problems of FSD:**
1. Cross-imports of slices are prohibited
2. Related functionality is spread across different layers leadgin to high coupling
3. Difficult to understand where the data comes from
4. Lack of a clear understanding of what goes on which layer
5. Problems with documentation

Comprared to our approach, FSD seems excessively complex due to the following:
1. As a project grows, the relationships between entities become significantly more complex, making the code harder to maintain. 
2. FSD's mental model can be interpreted differently by teams and team members.
3. A strict layer hierarchy forces the logic of a single business entity to be spread across entities, features, and widgets, thereby losing transparency. 
4. Cross-imports between slices are inevitable, and traversing them through @x files only adds cognitive noise without solving the coupling problem.

Our Container/Content + MobX approach, however, offers a simple and transparent model: all page logic is collected in one place, state is separated from presentation, and business logic is concentrated in the state. This gives us structure and predictability in the code without the need to spend time creating excessive abstractions.
