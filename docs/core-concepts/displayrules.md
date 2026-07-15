# Display Rules and Conditional Rendering

WebComponents can specify when they should appear using display rules. Each WebComponent can have multiple display rules, and if **ANY** rule matches, the component is shown.

Within a single display rule, there are three matcher lists that work together with **AND** logic:

```yaml
displayRules:
  # Rule 1: Show in navigation context for users with specific roles
  - allOf:
      - context-name: navigation
    anyOf:
      - role: user
      - role: admin
    noneOf:
      - path: "^/public/.*"
  
  # Rule 2: Show in drawer context on specific paths
  - allOf:
      - context-name: drawer
      - path: "^/dashboard/.*"
```

## How Display Rules Are Evaluated

The controller evaluates display rules using this logic:

**For each display rule (OR between rules):**

1. **ALL of `allOf` matchers must match** (AND logic)
2. **At least ONE of `anyOf` matchers must match** (OR logic) - if `anyOf` is present
3. **NONE of `noneOf` matchers can match** (NOT logic)

If all three conditions are satisfied, that display rule matches and the component is included.

## Matcher Types

Each matcher can check three properties:

**1. Context Name** - Where the `polyfea-context` element is located
```yaml
allOf:
  - context-name: navigation  # Exact match
```

**2. Path** - URL path as a regular expression
```yaml
allOf:
  - path: "^/dashboard/.*"  # Regex pattern
```

**3. Role** - User role from HTTP headers (e.g., `x-auth-request-roles`)
```yaml
anyOf:
  - role: admin
  - role: editor
```

## Examples

**Example 1: Admin-only component in navigation**
```yaml
displayRules:
  - allOf:
      - context-name: navigation
      - role: admin
```
Shows only if context is "navigation" AND user has "admin" role.

**Example 2: Multi-role component with path restriction**
```yaml
displayRules:
  - allOf:
      - context-name: toolbar
    anyOf:
      - role: admin
      - role: editor
    noneOf:
      - path: "^/public/.*"
```
Shows if context is "toolbar" AND (user is admin OR editor) AND path doesn't start with "/public/".

**Example 3: Multiple rules for different contexts**
```yaml
displayRules:
  # Show in desktop navigation for all users
  - allOf:
      - context-name: navigation
  
  # Or show in mobile menu for authenticated users
  - allOf:
      - context-name: mobile-menu
    anyOf:
      - role: user
      - role: admin
```
Shows if EITHER rule matches (context is "navigation" OR context is "mobile-menu" with a user role).

**Example 4: Path-based conditional rendering**
```yaml
displayRules:
  # Show settings only on settings pages
  - allOf:
      - context-name: sidebar
      - path: "^/settings/.*"
```

Shows only when in "sidebar" context AND on a path starting with "/settings/".