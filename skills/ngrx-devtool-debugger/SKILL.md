---
name: ngrx-devtool-debugger
description: Debug and visualize NgRx state management in Angular apps with real-time action monitoring, effect tracking, state diffs, and performance metrics
triggers:
  - debug ngrx state and actions
  - setup ngrx devtool
  - visualize ngrx effects and state changes
  - monitor ngrx actions in real time
  - configure ngrx debugging tool
  - track angular state management with ngrx devtool
  - troubleshoot ngrx performance issues
  - inspect ngrx store state changes
---

# NgRx DevTool Debugger

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

NgRx DevTool is a comprehensive debugging and visualization tool for NgRx state management in Angular applications. It provides real-time action monitoring, effect tracking, state visualization with diff viewer, and performance metrics without requiring browser extensions. The tool runs a separate WebSocket server that your Angular app connects to, displaying all NgRx activity in a dedicated UI.

## Installation

Install the package as a development dependency:

```bash
npm install --save-dev @amadeus-it-group/ngrx-devtool
```

Or with yarn:

```bash
yarn add -D @amadeus-it-group/ngrx-devtool
```

## Basic Setup

### 1. Configure Your Angular Application

Add the DevTool provider and meta-reducer to your application configuration:

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideStore } from '@ngrx/store';
import { provideEffects } from '@ngrx/effects';
import { 
  provideNgrxDevTool, 
  createDevToolMetaReducer 
} from '@amadeus-it-group/ngrx-devtool';
import * as fromRoot from './store/reducers';
import { AppEffects } from './store/effects/app.effects';

export const appConfig: ApplicationConfig = {
  providers: [
    provideStore(
      fromRoot.reducers,
      { 
        metaReducers: [createDevToolMetaReducer()] 
      }
    ),
    provideEffects([AppEffects]),
    provideNgrxDevTool({
      wsUrl: 'ws://localhost:4000',
      trackEffects: true,
      enabled: true
    })
  ]
};
```

### 2. Module-Based Configuration (Legacy)

If using NgModule instead of standalone components:

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { 
  NgRxDevToolModule, 
  createDevToolMetaReducer 
} from '@amadeus-it-group/ngrx-devtool';
import { reducers } from './store/reducers';
import { AppEffects } from './store/effects/app.effects';

@NgModule({
  imports: [
    StoreModule.forRoot(reducers, {
      metaReducers: [createDevToolMetaReducer()]
    }),
    EffectsModule.forRoot([AppEffects]),
    NgRxDevToolModule.forRoot({
      wsUrl: 'ws://localhost:4000',
      trackEffects: true
    })
  ]
})
export class AppModule {}
```

## CLI Commands

### Start the DevTool Server

The primary command to launch both the WebSocket server and UI:

```bash
npx ngrx-devtool
```

This starts:
- WebSocket server on `ws://localhost:4000`
- UI server on `http://localhost:3000`

### Custom Port Configuration

```bash
# Change WebSocket port
npx ngrx-devtool --ws-port 5000

# Change UI port
npx ngrx-devtool --ui-port 8080

# Custom ports for both
npx ngrx-devtool --ws-port 5000 --ui-port 8080
```

### Server Only Mode

Run only the WebSocket server without the UI:

```bash
npx ngrx-devtool --server-only
```

## Configuration Options

### DevTool Configuration Interface

```typescript
interface NgRxDevToolConfig {
  // WebSocket server URL (default: 'ws://localhost:4000')
  wsUrl?: string;
  
  // Enable/disable the devtool (default: true in development)
  enabled?: boolean;
  
  // Track NgRx effects (default: true)
  trackEffects?: boolean;
  
  // Maximum number of actions to store (default: 100)
  maxActions?: number;
  
  // Automatically connect on initialization (default: true)
  autoConnect?: boolean;
  
  // Reconnection attempts (default: 3)
  reconnectAttempts?: number;
  
  // Reconnection delay in ms (default: 1000)
  reconnectDelay?: number;
}
```

### Environment-Based Configuration

```typescript
// app.config.ts
import { isDevMode } from '@angular/core';
import { provideNgrxDevTool, createDevToolMetaReducer } from '@amadeus-it-group/ngrx-devtool';

export const appConfig: ApplicationConfig = {
  providers: [
    provideStore(
      reducers,
      { 
        metaReducers: isDevMode() ? [createDevToolMetaReducer()] : []
      }
    ),
    provideNgrxDevTool({
      wsUrl: `ws://${window.location.hostname}:4000`,
      enabled: isDevMode(),
      trackEffects: true,
      maxActions: 200,
      reconnectAttempts: 5
    })
  ]
};
```

### Production-Safe Configuration

```typescript
// app.config.ts
import { environment } from './environments/environment';

export const appConfig: ApplicationConfig = {
  providers: [
    provideStore(
      reducers,
      { 
        metaReducers: environment.devToolEnabled 
          ? [createDevToolMetaReducer()] 
          : []
      }
    ),
    provideNgrxDevTool({
      wsUrl: environment.devToolWsUrl,
      enabled: environment.devToolEnabled,
      trackEffects: environment.devToolEnabled
    })
  ]
};
```

```typescript
// environments/environment.ts
export const environment = {
  production: false,
  devToolEnabled: true,
  devToolWsUrl: 'ws://localhost:4000'
};

// environments/environment.prod.ts
export const environment = {
  production: true,
  devToolEnabled: false,
  devToolWsUrl: ''
};
```

## Usage Patterns

### Monitoring Actions

Once configured, all dispatched actions are automatically tracked:

```typescript
// user.actions.ts
import { createAction, props } from '@ngrx/store';

export const loadUsers = createAction('[User List] Load Users');
export const loadUsersSuccess = createAction(
  '[User API] Load Users Success',
  props<{ users: User[] }>()
);
export const loadUsersFailure = createAction(
  '[User API] Load Users Failure',
  props<{ error: string }>()
);

// Component dispatch - automatically tracked
this.store.dispatch(loadUsers());
```

### Effect Tracking

Effects are automatically tracked when `trackEffects: true`:

```typescript
// user.effects.ts
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { catchError, map, switchMap } from 'rxjs/operators';
import { of } from 'rxjs';
import * as UserActions from './user.actions';
import { UserService } from '../services/user.service';

@Injectable()
export class UserEffects {
  // This effect will be tracked by NgRx DevTool
  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UserActions.loadUsers),
      switchMap(() =>
        this.userService.getUsers().pipe(
          map(users => UserActions.loadUsersSuccess({ users })),
          catchError(error => of(UserActions.loadUsersFailure({ 
            error: error.message 
          })))
        )
      )
    )
  );

  constructor(
    private actions$: Actions,
    private userService: UserService
  ) {}
}
```

### State Visualization

The DevTool automatically captures state before and after each action, showing diffs:

```typescript
// counter.reducer.ts
import { createReducer, on } from '@ngrx/store';
import * as CounterActions from './counter.actions';

export interface CounterState {
  count: number;
  lastUpdated: Date | null;
}

export const initialState: CounterState = {
  count: 0,
  lastUpdated: null
};

export const counterReducer = createReducer(
  initialState,
  on(CounterActions.increment, state => ({
    ...state,
    count: state.count + 1,
    lastUpdated: new Date()
  })),
  on(CounterActions.decrement, state => ({
    ...state,
    count: state.count - 1,
    lastUpdated: new Date()
  })),
  on(CounterActions.reset, () => initialState)
);
```

### Custom Action Metadata

Add metadata to actions for better debugging:

```typescript
// product.actions.ts
import { createAction, props } from '@ngrx/store';

export const addToCart = createAction(
  '[Product] Add to Cart',
  props<{ 
    productId: string; 
    quantity: number;
    metadata?: { source: string; timestamp: number }
  }>()
);

// Dispatch with metadata
this.store.dispatch(addToCart({
  productId: 'prod-123',
  quantity: 2,
  metadata: {
    source: 'product-detail-page',
    timestamp: Date.now()
  }
}));
```

## Advanced Configuration

### Selective Action Tracking

Filter which actions to track by customizing the meta-reducer:

```typescript
// app.config.ts
import { ActionReducer, Action } from '@ngrx/store';
import { createDevToolMetaReducer } from '@amadeus-it-group/ngrx-devtool';

function createFilteredDevToolMetaReducer() {
  const devToolMetaReducer = createDevToolMetaReducer();
  
  return (reducer: ActionReducer<any>) => {
    const wrappedReducer = devToolMetaReducer(reducer);
    
    return (state: any, action: Action) => {
      // Skip tracking for high-frequency actions
      if (action.type.includes('[Router]') || 
          action.type.includes('[Internal]')) {
        return reducer(state, action);
      }
      return wrappedReducer(state, action);
    };
  };
}

export const appConfig: ApplicationConfig = {
  providers: [
    provideStore(reducers, {
      metaReducers: [createFilteredDevToolMetaReducer()]
    }),
    provideNgrxDevTool()
  ]
};
```

### Multiple Environment Setup

```typescript
// app.config.ts
const devToolConfig = (() => {
  const hostname = window.location.hostname;
  
  if (hostname === 'localhost') {
    return { wsUrl: 'ws://localhost:4000', enabled: true };
  } else if (hostname.includes('staging')) {
    return { wsUrl: 'ws://staging-devtool.example.com:4000', enabled: true };
  }
  return { enabled: false };
})();

export const appConfig: ApplicationConfig = {
  providers: [
    provideNgrxDevTool(devToolConfig)
  ]
};
```

## Troubleshooting

### Connection Issues

**Problem**: DevTool UI shows "Disconnected" status

**Solutions**:

1. Verify the DevTool server is running:
```bash
npx ngrx-devtool
```

2. Check WebSocket URL matches server configuration:
```typescript
provideNgrxDevTool({
  wsUrl: 'ws://localhost:4000' // Must match server port
})
```

3. Check browser console for connection errors:
```
WebSocket connection to 'ws://localhost:4000' failed: Connection refused
```

4. Verify no firewall blocking WebSocket connections

### Actions Not Appearing

**Problem**: Actions are dispatched but not showing in DevTool

**Solutions**:

1. Ensure meta-reducer is registered:
```typescript
provideStore(reducers, {
  metaReducers: [createDevToolMetaReducer()] // Must be present
})
```

2. Verify DevTool is enabled:
```typescript
provideNgrxDevTool({
  enabled: true // Check this is true
})
```

3. Check for action filtering that might be excluding actions

### Effects Not Tracked

**Problem**: Effects execute but don't appear in DevTool

**Solutions**:

1. Enable effect tracking:
```typescript
provideNgrxDevTool({
  trackEffects: true // Must be true
})
```

2. Ensure effects are properly registered:
```typescript
provideEffects([UserEffects, ProductEffects])
```

3. Verify effects use the `createEffect()` function

### Performance Issues

**Problem**: Application slows down with DevTool enabled

**Solutions**:

1. Reduce action history limit:
```typescript
provideNgrxDevTool({
  maxActions: 50 // Default is 100
})
```

2. Filter high-frequency actions:
```typescript
// Use custom meta-reducer to skip router/polling actions
```

3. Disable in production builds:
```typescript
provideNgrxDevTool({
  enabled: !environment.production
})
```

### Port Conflicts

**Problem**: Port 4000 or 3000 already in use

**Solutions**:

1. Use custom ports:
```bash
npx ngrx-devtool --ws-port 5000 --ui-port 8080
```

2. Update configuration to match:
```typescript
provideNgrxDevTool({
  wsUrl: 'ws://localhost:5000'
})
```

### State Diff Not Showing

**Problem**: State changes occur but diff viewer is empty

**Solutions**:

1. Ensure reducer returns new state reference:
```typescript
// ❌ Bad - mutates state
on(updateUser, (state, { user }) => {
  state.user = user; // Don't do this
  return state;
})

// ✅ Good - returns new state
on(updateUser, (state, { user }) => ({
  ...state,
  user
}))
```

2. Check state serialization for circular references

### Browser Compatibility

The DevTool requires WebSocket support. All modern browsers support WebSockets, but if you encounter issues:

1. Ensure browser is up to date
2. Check corporate proxy/firewall settings
3. Try different browser to isolate issue
4. Check browser console for specific WebSocket errors
