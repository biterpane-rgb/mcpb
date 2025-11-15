# CLAUDE.md - AI Assistant Guide for MCPB

This document provides comprehensive guidance for AI assistants working with the MCPB (MCP Bundles) codebase.

## Project Overview

**MCPB (MCP Bundles)** is a bundling and distribution format for Model Context Protocol (MCP) servers. It enables single-click installation of local MCP servers similar to how browser extensions (`.crx`) or VS Code extensions (`.vsix`) work.

### Key Components

1. **Bundle Specification** ([MANIFEST.md](MANIFEST.md)) - The `.mcpb` manifest format and schema
2. **CLI Tool** ([CLI.md](CLI.md)) - Command-line utilities for creating, signing, and validating bundles
3. **Core Library** (src/) - Code used by Claude for macOS/Windows to load and verify bundles

### Important Context

- **Renamed from DXT**: This project was recently renamed from "DXT (Desktop Extensions)" to "MCPB (MCP Bundles)"
- **Current Manifest Version**: 0.2 (with 0.3 in development)
- **License**: MIT
- **Package**: `@anthropic-ai/mcpb` on npm
- **Current Version**: 1.1.1 (see package.json)

## Repository Structure

```
mcpb/
├── src/                          # Source code (TypeScript)
│   ├── cli/                      # CLI commands implementation
│   │   ├── cli.ts                # Main CLI entry point
│   │   ├── init.ts               # Interactive manifest creation
│   │   ├── pack.ts               # Bundle packing logic
│   │   └── unpack.ts             # Bundle unpacking logic
│   ├── node/                     # Node.js-specific utilities
│   │   ├── files.ts              # File operations
│   │   ├── sign.ts               # Digital signature operations
│   │   └── validate.ts           # Manifest validation
│   ├── schemas/                  # Strict validation schemas (Zod)
│   │   ├── 0.1.ts                # Schema for manifest v0.1
│   │   ├── 0.2.ts                # Schema for manifest v0.2
│   │   ├── 0.3.ts                # Schema for manifest v0.3 (WIP)
│   │   ├── latest.ts             # Current latest schema
│   │   └── index.ts              # Schema exports
│   ├── schemas_loose/            # Lenient schemas for parsing
│   │   └── ...                   # Mirror of schemas/ with loose validation
│   ├── shared/                   # Shared utilities
│   │   ├── config.ts             # Configuration handling
│   │   ├── constants.ts          # Project constants
│   │   └── log.ts                # Logging utilities
│   ├── types.ts                  # TypeScript type definitions
│   ├── index.ts                  # Main export (Node.js + all features)
│   ├── browser.ts                # Browser-specific exports
│   ├── node.ts                   # Node-specific exports
│   └── cli.ts                    # CLI-specific exports
├── test/                         # Jest test files
├── examples/                     # Example MCPB bundles
│   ├── hello-world-node/         # Reference implementation (Node.js)
│   ├── file-system-node/         # File system MCP server
│   ├── file-manager-python/      # Python example
│   └── chrome-applescript/       # macOS-specific example
├── scripts/                      # Build and utility scripts
├── .github/workflows/            # GitHub Actions CI/CD
├── dist/                         # Compiled output (generated)
├── README.md                     # User-facing documentation
├── MANIFEST.md                   # Manifest specification
├── CLI.md                        # CLI documentation
├── CONTRIBUTING.md               # Contribution guidelines
└── package.json                  # Package configuration

```

## Development Setup

### Prerequisites

- **Node.js**: 16.0.0 or higher (tested on 20.19.x and 22.17.x)
- **Yarn**: 4.10.3+ (Berry/Modern Yarn)
- **TypeScript**: 5.6.3+

### Initial Setup

```bash
# Clone the repository
git clone https://github.com/anthropics/mcpb.git
cd mcpb

# Install dependencies (uses Yarn Berry)
yarn install

# Build the project
yarn build

# Run tests
yarn test
```

### Available Scripts

- `yarn build` - Compile TypeScript to JavaScript (output to `dist/`)
- `yarn build:code` - Same as `yarn build`
- `yarn build:schema` - Generate JSON schema from TypeScript schemas
- `yarn dev` - Watch mode for development (TypeScript compiler in watch mode)
- `yarn test` - Run Jest test suite
- `yarn test:watch` - Run tests in watch mode
- `yarn lint` - Run TypeScript compiler + ESLint checks
- `yarn fix` - Auto-fix linting and formatting issues
- `yarn dev-version` - Create a development version of the package

## Code Conventions and Standards

### TypeScript Configuration

- **Target**: ES2024
- **Module**: Node18 (Node16 resolution)
- **Strict Mode**: Enabled
- **Declaration Files**: Generated (`.d.ts`)
- **Source Directory**: `src/`
- **Output Directory**: `dist/`

### Code Style and Linting

The project uses **ESLint** with TypeScript support and **Prettier** for formatting.

#### Key ESLint Rules

1. **Import Organization** (simple-import-sort):
   - Side effects first
   - Node.js built-ins (prefixed with `node:`)
   - External packages
   - Absolute imports (`@/`)
   - Relative imports (`.` or `..`)

2. **Type Safety**:
   - Prefer type imports: `import type { Foo } from './foo.js'`
   - No explicit `any` types (`@typescript-eslint/no-explicit-any`)
   - No floating promises (`@typescript-eslint/no-floating-promises`)
   - No unused variables (with `_` prefix exception)

3. **Import Rules**:
   - No extraneous dependencies
   - No cyclical dependencies
   - Imports must be first in file
   - No duplicate imports

4. **Code Quality**:
   - No empty functions (unless commented)
   - No switch-case fall-through
   - Spaced comments (`// Comment` not `//Comment`)
   - Consistent type imports

### File Extensions

- **Import paths must include `.js` extension** even for TypeScript files
  - Example: `import { foo } from './bar.js'` (not `./bar` or `./bar.ts`)
  - This is due to ES modules in Node.js

### Testing

- **Framework**: Jest with ts-jest
- **Location**: `test/` directory
- **Pattern**: `*.test.ts` files
- **Config**: `jest.config.js` and `tsconfig.test.json`

#### Test Guidelines

```typescript
// Test files should match **/*.test.ts pattern
// Example: test/mcpbignore.test.ts

describe('Feature Name', () => {
  it('should do something specific', () => {
    // Test implementation
  });
});
```

### Commit Standards

**IMPORTANT**: All commits must be signed with GPG/SSH signatures.

- Configure commit signing: https://docs.github.com/en/authentication/managing-commit-signature-verification
- Commits without signatures will be rejected

## Key Technical Concepts

### Manifest Versions

The project supports multiple manifest schema versions:

- **v0.1**: Initial version
- **v0.2**: Current stable version (default)
- **v0.3**: Work in progress (see `src/schemas/0.3.ts`)

Each version has both:
- **Strict schema** (`src/schemas/*.ts`) - For validation
- **Loose schema** (`src/schemas_loose/*.ts`) - For parsing with defaults

### Dual Export Structure

The package provides multiple entry points:

1. **Main export** (`@anthropic-ai/mcpb`): Full Node.js functionality
2. **Browser export** (`@anthropic-ai/mcpb/browser`): Browser-compatible code
3. **Node export** (`@anthropic-ai/mcpb/node`): Node-specific utilities
4. **CLI export** (`@anthropic-ai/mcpb/cli`): CLI commands

### Bundle Format

- **Container**: ZIP archive with `.mcpb` extension
- **Compression**: Maximum compression level
- **Signature**: Optional PKCS#7 digital signature appended to file
- **Signature Format**: `[ZIP content]MCPB_SIG_V1[base64 PKCS#7]MCPB_SIG_END`

### Variable Substitution

Manifests support runtime variable substitution:

- `${__dirname}`: Bundle installation directory
- `${HOME}`, `${DESKTOP}`, `${DOCUMENTS}`, `${DOWNLOADS}`: User directories
- `${pathSeparator}` or `${/}`: Platform path separator
- `${user_config.KEY}`: User-configured values

## Common Development Tasks

### Adding a New Manifest Field

1. **Update Schema** (`src/schemas/latest.ts` or version-specific file):
   ```typescript
   export const ManifestSchema = z.object({
     // ... existing fields
     new_field: z.string().optional(),
   });
   ```

2. **Update Types** (`src/types.ts`):
   ```typescript
   export type Manifest = z.infer<typeof ManifestSchema>;
   ```

3. **Update Documentation** (`MANIFEST.md`):
   - Add field definition
   - Provide examples
   - Specify if required/optional

4. **Add Tests** (`test/`):
   - Test validation
   - Test default values
   - Test error cases

5. **Rebuild Schema**: `yarn build:schema`

### Adding a New CLI Command

1. **Create Command File** (`src/cli/your-command.ts`):
   ```typescript
   import { Command } from 'commander';

   export function registerYourCommand(program: Command): void {
     program
       .command('your-command')
       .description('Command description')
       .action(async (options) => {
         // Implementation
       });
   }
   ```

2. **Register in CLI** (`src/cli/cli.ts`):
   ```typescript
   import { registerYourCommand } from './your-command.js';

   // In main setup:
   registerYourCommand(program);
   ```

3. **Export from Index** (`src/index.ts`):
   ```typescript
   export * from './cli/your-command.js';
   ```

4. **Document in CLI.md**: Add usage examples and documentation

### Working with Schemas

The project uses **Zod** for schema validation. There are two types:

1. **Strict schemas** (`src/schemas/`): Validate user input, reject invalid data
2. **Loose schemas** (`src/schemas_loose/`): Parse with defaults, more forgiving

When updating schemas:
- Update both strict and loose versions
- Maintain backward compatibility when possible
- Version bump if breaking changes

### Running Integration Tests

```bash
# Run all tests
yarn test

# Run specific test file
yarn test test/mcpbignore.test.ts

# Run with coverage
yarn test --coverage

# Watch mode
yarn test:watch
```

## CI/CD and GitHub Actions

### Test Workflow (`.github/workflows/test.yml`)

Runs on:
- **Triggers**: Push/PR to `main` branch
- **Node versions**: 20.19.x, 22.17.x
- **OS**: macOS, Ubuntu, Windows
- **Steps**:
  1. Checkout code
  2. Setup Node.js with Yarn cache
  3. Install dependencies (`yarn install --immutable`)
  4. Build (`yarn build`)
  5. Lint and test (`yarn lint && yarn test`)

### Release Process

Per CONTRIBUTING.md:

1. Update version in `package.json`
2. Create a pull request with version bump
3. After merge, create a GitHub release
4. Package automatically published to npm

## Important Files for AI Assistants

### Critical Files to Understand

1. **MANIFEST.md**: Complete specification of the manifest format - READ THIS FIRST when working with manifests
2. **README.md**: User-facing documentation, project overview
3. **CONTRIBUTING.md**: Development workflow, contribution guidelines
4. **CLI.md**: CLI command documentation and examples
5. **package.json**: Dependencies, scripts, package metadata
6. **src/schemas/latest.ts**: Current manifest schema definition

### Example Files

Located in `examples/`, these demonstrate best practices:

- **hello-world-node/**: Reference implementation showing all manifest features
- **file-system-node/**: Practical example of a file system MCP server
- **file-manager-python/**: Python-based MCP server example
- **chrome-applescript/**: Platform-specific (macOS) implementation

## Best Practices for AI Assistants

### When Creating or Modifying Manifests

1. **Always reference MANIFEST.md** for the canonical specification
2. **Use the current manifest version** (check `src/schemas/latest.ts`)
3. **Include required fields**: `manifest_version`, `name`, `version`, `description`, `author`, `server`
4. **Follow naming conventions**: Machine-readable names (lowercase, hyphens)
5. **Validate with CLI**: Use `mcpb validate` before packing

### When Writing Code

1. **Follow TypeScript strict mode**: No `any`, proper typing
2. **Use type imports**: `import type { Foo } from './bar.js'`
3. **Include `.js` extensions**: All import paths must end in `.js`
4. **Organize imports**: Follow the import-sort rules
5. **Handle promises**: All promises must be awaited or .catch()
6. **Add tests**: Include Jest tests for new functionality
7. **Document public APIs**: Use JSDoc comments for exported functions

### When Making Changes

1. **Run tests first**: `yarn test` to ensure nothing breaks
2. **Run linting**: `yarn lint` before committing
3. **Auto-fix formatting**: Use `yarn fix` to automatically fix style issues
4. **Update documentation**: Keep README.md, MANIFEST.md, CLI.md in sync
5. **Add examples**: Update or add examples if changing manifest format
6. **Check all manifest versions**: Consider impact on v0.1, v0.2, v0.3

### Security Considerations

1. **Validate all inputs**: Use Zod schemas for validation
2. **Sanitize file paths**: Prevent directory traversal attacks
3. **Verify signatures**: Check PKCS#7 signatures properly
4. **Handle secrets carefully**: Mark sensitive config as `sensitive: true`
5. **No arbitrary code execution**: Validate server entry points

## Common Gotchas

### 1. File Extension in Imports

❌ **Wrong**:
```typescript
import { foo } from './bar';
import { baz } from './qux.ts';
```

✅ **Correct**:
```typescript
import { foo } from './bar.js';
import { baz } from './qux.js';
```

### 2. Import Organization

Imports must be ordered: side effects → Node.js built-ins → packages → absolute → relative

### 3. Manifest Version Compatibility

When updating schemas, maintain backward compatibility or bump manifest version.

### 4. Platform-Specific Paths

Use `${pathSeparator}` or `${/}` for cross-platform paths in manifests.

### 5. Yarn Berry (Modern Yarn)

This project uses Yarn 4.x (Berry), not Classic Yarn. Commands differ slightly:
- Use `yarn install --immutable` in CI (not `yarn install --frozen-lockfile`)
- `.yarnrc.yml` configures Yarn, not `.yarnrc`

## Debugging Tips

1. **Enable verbose logging**: Check `src/shared/log.ts` for logging utilities
2. **Validate manifests**: Use `mcpb validate <path>` to check manifest syntax
3. **Inspect packed bundles**: Use `mcpb info <file>` to see bundle contents
4. **Test signatures**: Use `mcpb verify <file>` to validate digital signatures
5. **Check TypeScript output**: Look in `dist/` after building

## Resources and References

### Internal Documentation

- [MANIFEST.md](MANIFEST.md) - Complete manifest specification
- [CLI.md](CLI.md) - CLI tool documentation
- [README.md](README.md) - User guide and project overview
- [CONTRIBUTING.md](CONTRIBUTING.md) - Contribution guidelines

### External Resources

- MCP Specification: https://modelcontextprotocol.io
- Zod Documentation: https://zod.dev
- TypeScript Handbook: https://www.typescriptlang.org/docs/
- Jest Documentation: https://jestjs.io/docs/getting-started

## Quick Reference Commands

```bash
# Development
yarn install          # Install dependencies
yarn build           # Compile TypeScript
yarn dev             # Watch mode
yarn test            # Run tests
yarn lint            # Check code quality
yarn fix             # Auto-fix issues

# CLI Usage
mcpb init            # Create manifest interactively
mcpb validate .      # Validate manifest.json
mcpb pack .          # Create .mcpb bundle
mcpb sign file.mcpb  # Sign bundle
mcpb verify file.mcpb # Verify signature
mcpb info file.mcpb  # Show bundle info

# Git
git commit -S -m "msg"  # Signed commit (required)
```

## Version Information

- **Project Version**: 1.1.1
- **Manifest Version**: 0.2 (latest stable)
- **Node.js**: >=16.0.0
- **TypeScript**: 5.6.3
- **Package Manager**: Yarn 4.10.3

---

**Last Updated**: 2025-11-15
**Maintained By**: Anthropic
**Repository**: https://github.com/anthropics/mcpb
