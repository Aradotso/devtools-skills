---
name: blatui-laravel-blade-components
description: CLI that installs shadcn/ui-style Blade components for the BLAT stack (Blade, Laravel, Alpine, Tailwind) into your Laravel project
triggers:
  - add blade UI components to my Laravel app
  - install BlatUI components
  - set up shadcn-style components in Laravel
  - add accessible Blade components with Alpine
  - use BlatUI to scaffold UI
  - install BLAT stack components
  - add Tailwind Blade components
  - set up BlatUI in Laravel
---

# BlatUI Laravel Blade Components

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

BlatUI is a CLI for Laravel that brings shadcn/ui's component philosophy to the **BLAT stack** (Blade · Laravel · Alpine · Tailwind). It copies accessible, themeable UI components directly into your project — you own the code and can customize it freely. Components are built with Blade, styled with Tailwind CSS v4, and use Alpine.js for interactivity.

**Key Features:**
- 156+ accessible components (WCAG AA compliant)
- Components are copied, not installed — you own and edit them
- Full design token system with light/dark mode
- Faithful port of shadcn/ui's design language
- 70 chart types, 64 pre-built blocks

## Requirements

- PHP 8.2+
- Laravel 11, 12, or 13
- Tailwind CSS v4 (required)
- Alpine.js 3
- Node 18+

## Installation

### New Project Setup

```bash
# 1. Install the CLI package
composer require anousss007/blatui

# 2. Install peer dependencies
composer require gehrisandro/tailwind-merge-laravel mallardduck/blade-lucide-icons
npm install -D alpinejs @alpinejs/anchor @alpinejs/collapse @alpinejs/focus

# 3. Publish foundations (theme tokens, Alpine engine, CSS)
php artisan vendor:publish --tag=blatui-foundations

# 4. Verify setup
php artisan blatui:init
```

**Update your Vite entrypoints** (replace contents):

```css
/* resources/css/app.css */
@import "./blatui.css";
```

```js
// resources/js/app.js
import "./blatui.js";
```

The published `blatui.css` includes Tailwind v4, oklch design tokens, and theme presets. The `blatui.js` file bootstraps Alpine with required plugins.

### Existing Project Setup

BlatUI is fully additive. Key considerations:

**Tailwind v4 Migration:**
```bash
# Check your version
php artisan blatui:init

# Migrate from v3 to v4 if needed
npx @tailwindcss/upgrade
```

**CSS Integration:**
```css
/* resources/css/app.css */
@import "tailwindcss";
@import "./blatui.css";  /* Add below Tailwind import */
```

**Alpine.js Integration (if already using Alpine):**

Don't import `blatui.js` — it would create a second Alpine instance. Use `blatui-core.js` instead:

```js
// resources/js/app.js
import Alpine from 'alpinejs'
import { registerBlatUI } from './blatui-core.js'

registerBlatUI(Alpine)  // Registers plugins, theme store, chart/calendar engines
window.Alpine = Alpine
Alpine.start()
```

## CLI Commands

### Initialize and Verify

```bash
# Check setup: packages, Tailwind v4, tokens, Alpine wiring
php artisan blatui:init
```

This command validates your entire setup and provides actionable fixes if something is missing.

### List Components

```bash
# List all available components, blocks, and charts
php artisan blatui:list

# Get details about a specific component
php artisan blatui:list select
php artisan blatui:list dialog
php artisan blatui:list button
```

### Add Components

```bash
# Add one component
php artisan blatui:add button

# Add multiple components (dependencies resolved automatically)
php artisan blatui:add button card input select

# Install all components at once
php artisan blatui:add --all
```

Components are copied to `resources/views/components/ui/`. The CLI will notify you of any additional npm/composer packages needed (e.g., charts require `apexcharts`).

## Component Usage

Components are used as Blade anonymous components under the `x-ui.*` namespace.

### Basic Card Example

```blade
<x-ui.card class="max-w-md">
    <x-ui.card-header>
        <x-ui.card-title>Account Settings</x-ui.card-title>
        <x-ui.card-description>
            Manage your account preferences
        </x-ui.card-description>
    </x-ui.card-header>
    <x-ui.card-content>
        <p>Card content goes here</p>
    </x-ui.card-content>
    <x-ui.card-footer>
        <x-ui.button>Save Changes</x-ui.button>
    </x-ui.card-footer>
</x-ui.card>
```

### Button Variants

```blade
{{-- Default --}}
<x-ui.button>Click Me</x-ui.button>

{{-- Variants --}}
<x-ui.button variant="destructive">Delete</x-ui.button>
<x-ui.button variant="outline">Cancel</x-ui.button>
<x-ui.button variant="secondary">Secondary</x-ui.button>
<x-ui.button variant="ghost">Ghost</x-ui.button>
<x-ui.button variant="link">Link Style</x-ui.button>

{{-- Sizes --}}
<x-ui.button size="sm">Small</x-ui.button>
<x-ui.button size="lg">Large</x-ui.button>
<x-ui.button size="icon">
    <x-lucide-search class="h-4 w-4" />
</x-ui.button>

{{-- With icons --}}
<x-ui.button>
    <x-lucide-mail class="mr-2 h-4 w-4" />
    Send Email
</x-ui.button>
```

### Form Components

```blade
<form method="POST" action="/profile" class="space-y-4">
    @csrf
    
    {{-- Input --}}
    <div class="space-y-2">
        <x-ui.label for="email">Email</x-ui.label>
        <x-ui.input 
            id="email" 
            type="email" 
            name="email" 
            placeholder="m@example.com"
            value="{{ old('email', $user->email) }}"
            required
        />
        @error('email')
            <p class="text-sm text-destructive">{{ $message }}</p>
        @enderror
    </div>

    {{-- Textarea --}}
    <div class="space-y-2">
        <x-ui.label for="bio">Bio</x-ui.label>
        <x-ui.textarea 
            id="bio" 
            name="bio" 
            rows="4"
            placeholder="Tell us about yourself"
        >{{ old('bio', $user->bio) }}</x-ui.textarea>
    </div>

    {{-- Select --}}
    <div class="space-y-2">
        <x-ui.label for="role">Role</x-ui.label>
        <x-ui.select id="role" name="role">
            <option value="user">User</option>
            <option value="admin">Admin</option>
            <option value="moderator">Moderator</option>
        </x-ui.select>
    </div>

    {{-- Checkbox --}}
    <div class="flex items-center space-x-2">
        <x-ui.checkbox id="terms" name="terms" />
        <x-ui.label for="terms" class="font-normal">
            I agree to the terms and conditions
        </x-ui.label>
    </div>

    <x-ui.button type="submit">Save Changes</x-ui.button>
</form>
```

### Dialog Component

```blade
{{-- Dialog trigger and content --}}
<div x-data="{ open: false }">
    <x-ui.button @click="open = true">
        Open Dialog
    </x-ui.button>

    <x-ui.dialog x-model="open">
        <x-ui.dialog-content>
            <x-ui.dialog-header>
                <x-ui.dialog-title>Confirm Action</x-ui.dialog-title>
                <x-ui.dialog-description>
                    Are you sure you want to proceed?
                </x-ui.dialog-description>
            </x-ui.dialog-header>
            <x-ui.dialog-footer>
                <x-ui.button variant="outline" @click="open = false">
                    Cancel
                </x-ui.button>
                <x-ui.button @click="open = false">
                    Confirm
                </x-ui.button>
            </x-ui.dialog-footer>
        </x-ui.dialog-content>
    </x-ui.dialog>
</div>
```

### Table Component

```blade
<x-ui.table>
    <x-ui.table-header>
        <x-ui.table-row>
            <x-ui.table-head>Name</x-ui.table-head>
            <x-ui.table-head>Email</x-ui.table-head>
            <x-ui.table-head>Role</x-ui.table-head>
            <x-ui.table-head class="text-right">Actions</x-ui.table-head>
        </x-ui.table-row>
    </x-ui.table-header>
    <x-ui.table-body>
        @foreach($users as $user)
            <x-ui.table-row>
                <x-ui.table-cell class="font-medium">
                    {{ $user->name }}
                </x-ui.table-cell>
                <x-ui.table-cell>{{ $user->email }}</x-ui.table-cell>
                <x-ui.table-cell>{{ $user->role }}</x-ui.table-cell>
                <x-ui.table-cell class="text-right">
                    <x-ui.button variant="ghost" size="sm">Edit</x-ui.button>
                </x-ui.table-cell>
            </x-ui.table-row>
        @endforeach
    </x-ui.table-body>
</x-ui.table>
```

### Alert Component

```blade
{{-- Default alert --}}
<x-ui.alert>
    <x-lucide-info class="h-4 w-4" />
    <x-ui.alert-title>Information</x-ui.alert-title>
    <x-ui.alert-description>
        Your session will expire in 5 minutes.
    </x-ui.alert-description>
</x-ui.alert>

{{-- Destructive variant --}}
<x-ui.alert variant="destructive">
    <x-lucide-alert-circle class="h-4 w-4" />
    <x-ui.alert-title>Error</x-ui.alert-title>
    <x-ui.alert-description>
        Failed to save changes. Please try again.
    </x-ui.alert-description>
</x-ui.alert>
```

### Badge Component

```blade
<x-ui.badge>Default</x-ui.badge>
<x-ui.badge variant="secondary">Secondary</x-ui.badge>
<x-ui.badge variant="destructive">Destructive</x-ui.badge>
<x-ui.badge variant="outline">Outline</x-ui.badge>

{{-- With icons --}}
<x-ui.badge>
    <x-lucide-check class="mr-1 h-3 w-3" />
    Verified
</x-ui.badge>
```

## Theming and Customization

### CSS Variables

All design tokens live in `resources/css/blatui.css` as CSS custom properties (oklch color space):

```css
@theme inline {
  --color-background: oklch(100% 0 0);
  --color-foreground: oklch(9% 0 0);
  --color-primary: oklch(47% 0.2 264);
  --color-primary-foreground: oklch(98% 0 0);
  /* ... more tokens */
}

.dark {
  --color-background: oklch(9% 0 0);
  --color-foreground: oklch(98% 0 0);
  /* ... dark mode overrides */
}
```

**To customize your theme:**

1. Edit `resources/css/blatui.css`
2. Change the oklch values for any token
3. Changes apply to all components instantly

### Dark Mode

BlatUI uses class-based dark mode. Add the `dark` class to your root element:

```blade
{{-- In your layout --}}
<html class="dark">
    {{-- Dark mode active --}}
</html>

{{-- Toggle with Alpine --}}
<div x-data="{ dark: false }" :class="{ 'dark': dark }">
    <x-ui.button @click="dark = !dark">
        Toggle Dark Mode
    </x-ui.button>
</div>
```

### Component Customization

Since components are copied to your project, you can edit them directly:

```bash
# Components live here — edit freely
ls resources/views/components/ui/
```

Example: customize the button component:

```blade
{{-- resources/views/components/ui/button.blade.php --}}
@props([
    'variant' => 'default',
    'size' => 'default',
    'type' => 'button',
])

@php
$classes = \TailwindMerge\Laravel\Facades\TailwindMerge::merge([
    'inline-flex items-center justify-center rounded-md font-medium transition-colors',
    'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring',
    'disabled:pointer-events-none disabled:opacity-50',
    
    // Add your custom variant
    match($variant) {
        'brand' => 'bg-purple-600 text-white hover:bg-purple-700',
        default => '...',
    },
    
    $attributes->get('class'),
]);
@endphp

<button {{ $attributes->merge(['type' => $type, 'class' => $classes]) }}>
    {{ $slot }}
</button>
```

## Blocks and Charts

Blocks (dashboards, sidebars, auth pages) and charts are **copy-paste from the demo site**, not installed via CLI. Browse them at the live demo and copy the exact Blade code you need.

**Live Demo:** https://blatui.remix-it.com/blocks

Common blocks:
- Authentication layouts
- Dashboard shells
- Sidebar navigation
- Data tables with filters
- 70+ chart types (bar, line, pie, area, radar, etc.)

Charts require `apexcharts`:

```bash
npm install -D apexcharts
```

## Common Patterns

### Full Login Page

```blade
<div class="flex min-h-screen items-center justify-center bg-muted">
    <x-ui.card class="w-full max-w-md">
        <x-ui.card-header class="space-y-1">
            <x-ui.card-title class="text-2xl font-bold">
                Welcome back
            </x-ui.card-title>
            <x-ui.card-description>
                Enter your credentials to sign in
            </x-ui.card-description>
        </x-ui.card-header>
        <x-ui.card-content>
            <form method="POST" action="{{ route('login') }}" class="space-y-4">
                @csrf
                
                <div class="space-y-2">
                    <x-ui.label for="email">Email</x-ui.label>
                    <x-ui.input 
                        id="email" 
                        type="email" 
                        name="email" 
                        placeholder="m@example.com"
                        required
                    />
                </div>

                <div class="space-y-2">
                    <x-ui.label for="password">Password</x-ui.label>
                    <x-ui.input 
                        id="password" 
                        type="password" 
                        name="password"
                        required
                    />
                </div>

                <div class="flex items-center justify-between">
                    <div class="flex items-center space-x-2">
                        <x-ui.checkbox id="remember" name="remember" />
                        <x-ui.label for="remember" class="font-normal">
                            Remember me
                        </x-ui.label>
                    </div>
                    <a href="{{ route('password.request') }}" class="text-sm text-primary hover:underline">
                        Forgot password?
                    </a>
                </div>

                <x-ui.button type="submit" class="w-full">
                    Sign in
                </x-ui.button>
            </form>
        </x-ui.card-content>
        <x-ui.card-footer>
            <p class="text-center text-sm text-muted-foreground">
                Don't have an account?
                <a href="{{ route('register') }}" class="text-primary hover:underline">
                    Sign up
                </a>
            </p>
        </x-ui.card-footer>
    </x-ui.card>
</div>
```

### Data Table with Actions

```blade
<div class="space-y-4">
    <div class="flex items-center justify-between">
        <x-ui.input 
            type="search" 
            placeholder="Search users..."
            class="max-w-sm"
        />
        <x-ui.button>
            <x-lucide-plus class="mr-2 h-4 w-4" />
            Add User
        </x-ui.button>
    </div>

    <x-ui.table>
        <x-ui.table-header>
            <x-ui.table-row>
                <x-ui.table-head>Name</x-ui.table-head>
                <x-ui.table-head>Status</x-ui.table-head>
                <x-ui.table-head>Role</x-ui.table-head>
                <x-ui.table-head class="text-right">Actions</x-ui.table-head>
            </x-ui.table-row>
        </x-ui.table-header>
        <x-ui.table-body>
            @foreach($users as $user)
                <x-ui.table-row>
                    <x-ui.table-cell class="font-medium">
                        {{ $user->name }}
                    </x-ui.table-cell>
                    <x-ui.table-cell>
                        <x-ui.badge variant="{{ $user->active ? 'default' : 'secondary' }}">
                            {{ $user->active ? 'Active' : 'Inactive' }}
                        </x-ui.badge>
                    </x-ui.table-cell>
                    <x-ui.table-cell>{{ $user->role }}</x-ui.table-cell>
                    <x-ui.table-cell class="text-right">
                        <x-ui.button variant="ghost" size="sm">
                            <x-lucide-pencil class="h-4 w-4" />
                        </x-ui.button>
                        <x-ui.button variant="ghost" size="sm">
                            <x-lucide-trash class="h-4 w-4" />
                        </x-ui.button>
                    </x-ui.table-cell>
                </x-ui.table-row>
            @endforeach
        </x-ui.table-body>
    </x-ui.table>

    {{ $users->links() }}
</div>
```

### Toast Notifications

```blade
{{-- In your layout, add the toast container --}}
<div x-data="toastStore()" @toast.window="show($event.detail)">
    <template x-for="toast in toasts" :key="toast.id">
        <x-ui.toast :toast="toast" />
    </template>
</div>

{{-- Trigger from Alpine --}}
<x-ui.button @click="$dispatch('toast', { message: 'Settings saved!', variant: 'success' })">
    Save
</x-ui.button>

{{-- Trigger from Livewire --}}
$this->dispatch('toast', message: 'User created', variant: 'success');
```

## Troubleshooting

### `blatui:init` reports missing packages

```bash
# Install missing dependencies
composer require gehrisandro/tailwind-merge-laravel mallardduck/blade-lucide-icons
npm install -D alpinejs @alpinejs/anchor @alpinejs/collapse @alpinejs/focus
```

### Tailwind v3 detected

BlatUI requires Tailwind CSS v4. Migrate first:

```bash
npx @tailwindcss/upgrade
```

### Components not styled correctly

Ensure `blatui.css` is imported in your CSS:

```css
/* resources/css/app.css */
@import "./blatui.css";
```

And that Vite is processing it:

```js
// vite.config.js
export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
});
```

### Alpine not working

Verify Alpine is loaded. If you see "Alpine is not defined":

```js
// resources/js/app.js
import "./blatui.js";  // Boots Alpine for you
```

Or if already using Alpine:

```js
import Alpine from 'alpinejs'
import { registerBlatUI } from './blatui-core.js'

registerBlatUI(Alpine)
window.Alpine = Alpine
Alpine.start()
```

### Charts not rendering

Charts need `apexcharts`:

```bash
npm install -D apexcharts
```

The chart engine is lazy-loaded from `resources/js/blatui.js` — ensure it's imported.

### Dark mode not working

Add the `dark` class to your `<html>` or root element:

```blade
<html class="dark">
```

Or toggle dynamically with Alpine:

```blade
<div x-data="{ dark: localStorage.getItem('theme') === 'dark' }" 
     :class="{ 'dark': dark }"
     @toggle-dark.window="dark = !dark; localStorage.setItem('theme', dark ? 'dark' : 'light')">
    {{-- Your app --}}
</div>
```

### Component dependencies not installed

When you add a component, the CLI tells you if it needs extra packages:

```bash
php artisan blatui:add calendar
# Output: Install required package: npm install -D @fullcalendar/core
```

Follow the instructions and re-run your build.

## Resources

- **Documentation & Demo:** https://blatui.remix-it.com
- **GitHub:** https://github.com/anousss007/blatui
- **Packagist:** https://packagist.org/packages/anousss007/blatui
