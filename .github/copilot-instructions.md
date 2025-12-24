# GitHub Desktop AI Coding Guidelines

## Project Overview
GitHub Desktop is an Electron-based desktop application written in TypeScript and React. It provides a GUI for Git and GitHub operations, focusing on making version control workflows approachable and frustration-free.

## Architecture
- **Main Process**: Handles Electron lifecycle, IPC, file system operations (`app/src/main-process/`)
- **Renderer Process**: React-based UI components (`app/src/ui/`)
- **Shared Libraries**: Common utilities and models (`app/src/lib/`, `app/src/models/`)
- **IPC Communication**: Typed channels defined in `app/src/lib/ipc-shared.ts` for secure main-renderer communication

## Development Workflow
- **Setup**: `yarn` to install dependencies
- **Build**: `yarn build:dev` for development, `yarn build:prod` for production
- **Run**: `yarn start` launches the app with hot reload (Ctrl/Cmd+Alt+R to refresh)
- **Main Process Changes**: Require `yarn build:dev` then `yarn start`
- **Test**: `yarn test` (unit tests), `yarn test:script` (script tests)
- **Lint**: `yarn lint` (includes Prettier formatting)
- **Debug**: Toggle Developer Tools from View menu; React DevTools auto-installs in dev mode

## Key Patterns & Conventions
- **Error Handling**: Use `fatalError()` for unrecoverable errors, `assertNever()` for exhaustive checks, `forceUnwrap()` for null assertions with rationale
- **IPC**: Define channels in `ipc-shared.ts` with strong typing; use `ipcMain.on/handle` in main, `ipcRenderer.invoke/send` in renderer
- **State Management**: Actions dispatched through `app/src/ui/dispatcher/dispatcher.ts`
- **Disposables**: Use `event-kit` for cleanup (e.g., `Disposable.fromSubscription()`)
- **Logging**: Custom logging system in `main-process/log.ts`
- **File Paths**: Use `path.join()` and absolute paths; handle platform differences
- **Async**: Prefer async/await over Promises; use `try/catch` with specific error types
- **TypeScript**: Strict typing required; avoid `any`; use interfaces for complex objects
- **React**: Functional components with hooks; avoid class components
- **Testing**: Unit tests in `app/test/unit/`; use fixtures from `app/test/fixtures/`; mock IPC and external deps
- **Webpack**: Separate configs for dev/prod; handles TypeScript, SCSS, assets

## Code Examples
### IPC Usage
```typescript
// In ipc-shared.ts
export type RequestResponseChannels = {
  'get-repositories': () => Promise<ReadonlyArray<IRepository>>
}

// In main process
handle('get-repositories', async () => {
  return getRepositories()
})

// In renderer
const repos = await ipcRenderer.invoke('get-repositories')
```

### Error Handling
```typescript
import { fatalError, forceUnwrap } from '../lib/fatal-error'

function processRepo(repo: IRepository | null) {
  const validRepo = forceUnwrap('Repository must exist', repo)
  // proceed with validRepo
}
```

### Dispatcher Action
```typescript
// In dispatcher.ts
export async function selectRepository(
  repository: Repository
): Promise<Repository | null> {
  // validation and state updates
  this.emitUpdate()
  return repository
}
```

## Dependencies & Integrations
- **Electron**: Cross-platform desktop framework
- **React**: UI rendering with custom hooks
- **TypeScript**: Type safety throughout
- **Webpack**: Bundling with hot reload
- **SCSS**: Styling with custom variables
- **Git Integration**: Direct libgit2 usage via custom bindings
- **GitHub API**: REST/GraphQL for repository operations
- **Keytar**: Secure credential storage
- **Desktop Notifications**: Native OS notifications

## Security Considerations
- IPC channels validate sender trustworthiness
- Certificate validation for HTTPS requests
- Secure storage for tokens via keytar
- Input sanitization for user-provided paths
- Avoid exposing sensitive data in renderer process

## Performance Notes
- Lazy load UI components to reduce bundle size
- Use virtualization for large lists (e.g., commit history)
- Debounce user input (e.g., search, typing)
- Background processing for Git operations
- Memory management for large repositories

## Testing Strategy
- Unit tests for pure functions and utilities
- Integration tests for Git operations
- Mock external APIs and file system
- Use test fixtures for consistent data
- CI runs full test suite on multiple platforms

## Common Pitfalls
- Forgetting to rebuild after main process changes
- Not handling platform-specific path separators
- Blocking main thread with synchronous operations
- Leaking event listeners (use disposables)
- Assuming renderer has access to Node APIs
- Not awaiting async operations in IPC handlers