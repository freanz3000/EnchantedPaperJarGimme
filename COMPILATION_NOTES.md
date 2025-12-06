# EnchantedPaper Compilation Notes

## Current Status

**BLOCKED**: Cannot complete full compilation due to network restrictions in the build environment.

## Required Network Access

The EnchantedPaper build requires access to several Maven repositories that are currently blocked:

### Critical Repositories
1. **repo.papermc.io** - Required for Paper build dependencies
   - yarn mappings (net.fabricmc:yarn)
   - Paper-specific artifacts
   - Status: BLOCKED

2. **repo.velocitypowered.com** - Required for Velocity plugin support in Master
   - velocity-api
   - Status: BLOCKED

3. **maven.fabricmc.net** - Alternative source for Fabric dependencies
   - Status: BLOCKED

4. **hub.spigotmc.org** - Required for Spigot submodules in Paper
   - Status: BLOCKED (workaround available)

5. **jitpack.io** - Potential alternative dependency source
   - Status: BLOCKED

## Workarounds Applied

### 1. Git Submodule Bypass (for hub.spigotmc.org)

Created a custom git-submodule wrapper that skips submodule updates:

**Location**: `/tmp/git-exec/git-submodule`

**Usage**:
```bash
GIT_EXEC_PATH=/tmp/git-exec:$(git --exec-path) ./gradlew applyPatches
```

This successfully bypasses the Paper submodule initialization that requires hub.spigotmc.org.

### 2. Master Build Configuration Changes

Modified `EnchantedPaper-Master/build.gradle.kts` to:
- Comment out repo.velocitypowered.com repository
- Comment out Velocity API dependencies (causes compilation errors in velocity plugin support)
- Exclude brigadier transitive dependency from BungeeCord API

**Result**: Master module has 29 compilation errors in `MultiPaperVelocity.java` due to missing Velocity API.

## Successful Builds

The following component compiled successfully:
- **EnchantedPaper-MasterMessagingProtocol**: `EnchantedPaper-MasterMessagingProtocol/build/libs/enchantedpaper-mastermessagingprotocol-2.12.3-1.21.1.jar`

## Build Steps Required (When Network Access is Available)

### 1. Apply Patches
```bash
# Use git wrapper to skip blocked submodules
GIT_EXEC_PATH=/tmp/git-exec:$(git --exec-path) ./gradlew applyPatches
```

### 2. Build Master
```bash
./gradlew :enchantedpaper-master:shadowJar
```

### 3. Build Server
```bash
./gradlew shadowjar createReobfPaperclipJar
```

### 4. Collect Artifacts
- Server JAR: `build/libs/enchantedpaper-paperclip-*.jar`
- Master JAR: `EnchantedPaper-Master/build/libs/enchantedpaper-master-*-all.jar`

## Recommendations

To successfully build EnchantedPaper, one of the following is required:

1. **Whitelist Required Domains**: Request access to the blocked Maven repositories
   - repo.papermc.io
   - repo.velocitypowered.com (or remove Velocity support from Master)

2. **Pre-download Dependencies**: Populate the Gradle cache with required dependencies before building

3. **Alternative Build Environment**: Use a different environment with unrestricted network access

4. **Use Pre-built Upstream**: Consider using pre-built Paper jars and only building the EnchantedPaper patches

## Modified Files

- `EnchantedPaper-Master/build.gradle.kts` - Commented out blocked repositories and Velocity dependencies
- `/tmp/git-exec/git-submodule` - Custom git wrapper (temporary, needs to be recreated per session)
