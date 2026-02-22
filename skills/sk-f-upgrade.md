This skill belongs to the Flint shard. Ensure you have [[init-f]] in context before continuing.

# Skill: Upgrade Flint

Upgrade this Flint workspace to the latest framework version by reading and applying migration documents.

# Actions

## Step 1: Check Version Status

Run `flint upgrade --status` to see:
- Current `flintVersion`
- Latest available version
- Available migrations

If already up to date, inform the user and stop.

## Step 2: Download Migrations

Run `flint upgrade` to download migration documents to `.flint/flint/migrations/`.

## Step 3: Read Migration Documents

For each migration (in order from oldest to newest):

1. Read `.flint/flint/migrations/<from>-to-<to>/migration.md`
2. Understand:
   - What changed
   - What actions are required
   - What files are affected

## Step 4: Apply Changes

For each migration document:

1. Follow the **Breaking Changes** section — these are required
2. Apply each action described
3. Run any suggested verification commands
4. If optional scripts are provided and helpful, run them

## Step 5: Update Version

After all migrations are applied:

1. Edit `.flint/flint.config.yaml`
2. Update `flintVersion` to the new version
3. Save the file

## Step 6: Verify

Run these checks:
- `flint list` — should show this Flint
- `flint shard list` — should work without errors
- Check that any breaking changes were resolved

## Step 7: Report

Tell the user:
- Previous version
- New version
- Summary of changes applied
- Any manual steps they may need to take

# Output

- Flint upgraded to latest version
- All migrations applied
- Version number updated in config
