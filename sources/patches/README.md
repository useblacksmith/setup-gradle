# Patches for External Dependencies

This directory contains patches for external npm dependencies used by the Gradle GitHub Actions. These patches modify the behavior of the dependencies to better suit our needs.

## Current Patches

### 1. `@actions/cache`

**File:** `@actions+cache+3.2.213.patch`

**Purpose:**
- Modifies the return types of `restoreCache` and `saveCache` functions to return a `CacheEntry` object instead of just a string or number
- Adds a `CacheEntry` class that includes both the cache key and size information
- Modifies error handling to propagate errors rather than catching them

**Why needed:**
The Gradle GitHub Actions need more information about cache entries, particularly their size, for better reporting and management. The patch enhances the API to include this information.

### 2. `@azure/logger`

**File:** `@azure+logger+1.1.4.patch`

**Purpose:**
- Disables automatic enabling of logging based on environment variables
- Removes sensitive information from error messages

**Why needed:**
This prevents unwanted verbose logging in production environments and improves security by removing potentially sensitive information from error messages.

## How Patches Are Generated

We use [patch-package](https://github.com/ds300/patch-package) to create and apply patches. Here's how to create or update a patch:

1. Install the dependency you want to patch at a specific version:
   ```bash
   npm install @actions/cache@npm:@useblacksmith/cache@X.Y.Z
   ```

2. Make the necessary changes to the files in `node_modules/@actions/cache/`.

3. Generate the patch:
   ```bash
   npx patch-package @actions/cache
   ```

4. The patch will be created in the `patches` directory with the format `@actions+cache+X.Y.Z.patch`.

## Updating the Cache Dependency

When updating the `@actions/cache` dependency to a new version, follow these steps:

1. Create a temporary directory for patch generation:
   ```bash
   mkdir -p temp-patching && cd temp-patching
   npm init -y
   ```

2. Install the new cache version:
   ```bash
   npm install @actions/cache@npm:@useblacksmith/cache@X.Y.Z patch-package --save-dev
   ```

3. Make the necessary changes to the following files:
   - `node_modules/@actions/cache/lib/cache.d.ts`: Update return types
   - `node_modules/@actions/cache/lib/cache.js`: Update implementation to return `CacheEntry` objects and modify error handling

4. Generate the patch:
   ```bash
   npx patch-package @actions/cache
   ```

5. Copy the new patch file to the main project:
   ```bash
   cp patches/@actions+cache+X.Y.Z.patch ../patches/
   ```

6. Update the project's `package.json` to use the new version:
   ```json
   "@actions/cache": "npm:@useblacksmith/cache@X.Y.Z"
   ```

7. Remove the old patch file (if it exists) and install dependencies:
   ```bash
   rm patches/@actions+cache+[old-version].patch
   npm install
   ```

8. Build and test the project to ensure everything works as expected:
   ```bash
   npm run build
   npm test
   ```

## Key Changes in Cache Patches

The main changes we make to the cache implementation are:

1. In `cache.d.ts`:
   - Change return type of `restoreCache` from `Promise<string | undefined>` to `Promise<CacheEntry | undefined>`
   - Change return type of `saveCache` from `Promise<number>` to `Promise<CacheEntry>`
   - Add `CacheEntry` class definition

2. In `cache.js`:
   - Add `CacheEntry` class implementation
   - Modify `restoreCache` to return `new CacheEntry(cacheKey, archiveFileSize)` instead of just `cacheKey`
   - Modify `saveCache` to return `new CacheEntry(key, archiveFileSize)` instead of just `cacheId`
   - Comment out error handling to allow errors to propagate

## Troubleshooting

If you encounter issues with patches:

1. **Patch fails to apply**: This usually means the dependency has changed significantly. You'll need to create a new patch following the steps above.

2. **Missing functionality after patching**: Check that your patch includes all necessary changes and that it's properly applied.

3. **Runtime errors**: Ensure your code properly handles the modified API from the patched dependencies.