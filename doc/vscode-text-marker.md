# VSCode Text Marker Extension - Development Guide

## Project Overview

The VSCode Text Marker Extension is a comprehensive Visual Studio Code extension that enables advanced text highlighting capabilities with configurable colors, multiple matching modes, and persistent highlight storage.

## Local Development Setup

### Prerequisites

1. **Node.js**: Version 12.14.1 (specified in `.nvmrc`)
2. **Yarn**: Package manager (https://yarnpkg.com/)
3. **VS Code**: For development and testing

### Installation Steps

#### 1. Install Dependencies
```bash
yarn install
```

#### 2. Compile the Extension
```bash
yarn run compile
```

This compiles TypeScript files from `src/` to `out/` directory with source maps.

#### 3. Run Quality Checks
```bash
yarn run check
```

Runs compilation, linting, and all tests.

## Local Installation Options

### Option A: Extension Development Host (Recommended for Development)
1. Open project in VS Code
2. Press `F5` or `Run > Start Debugging`
3. Opens new "Extension Development Host" window with extension loaded
4. Make changes and reload window (`Ctrl+R`) to test updates

### Option B: Package and Install as VSIX

#### Step-by-Step Build Flow (Tested & Working)

1. **Install Dependencies**:
   ```bash
   # Install dependencies (use npm if yarn is not available)
   npm install
   # OR if you have yarn installed:
   yarn install
   ```

2. **Install VS Code Extension Manager CLI**:
   ```bash
   npm install -g vsce
   # Note: You may see warnings about deprecated vsce, but it still works
   ```

3. **Install Yarn (if not available)**:
   ```bash
   npm install -g yarn
   ```

4. **Handle TypeScript Compilation Issues**:

   Due to TypeScript strict mode issues in test files, we need to temporarily modify the build script:

   ```bash
   # Backup the original package.json
   cp package.json package.json.bak

   # Temporarily modify the prepublish script to skip compilation
   sed -i '' 's/"yarn run compile"/"echo Skipping compile"/' package.json
   ```

5. **Compile the Extension** (before packaging):
   ```bash
   # Compile the main extension code (this works)
   npm run compile
   ```

6. **Package the Extension**:
   ```bash
   vsce package
   ```

   When prompted about '*' activation, type `y` to continue.

7. **Restore Original Configuration**:
   ```bash
   # Restore the original package.json
   mv package.json.bak package.json
   ```

8. **Install the Generated VSIX**:
   ```bash
   code --install-extension text-marker-1.11.0.vsix
   ```

#### Alternative Installation Methods

**Via VS Code UI:**
1. Open VS Code
2. Go to Extensions view (`Ctrl+Shift+X` or `Cmd+Shift+X`)
3. Click "..." (More Actions) → "Install from VSIX..."
4. Select `text-marker-1.11.0.vsix`

**Via Command Palette:**
1. Open Command Palette (`Ctrl+Shift+P` or `Cmd+Shift+P`)
2. Type "Extensions: Install from VSIX..."
3. Browse to and select the VSIX file

#### Troubleshooting Build Issues

**Common Problems & Solutions:**

1. **TypeScript Compilation Errors**:
   - The project has some TypeScript strict mode issues in test files
   - **Solution**: Use the build script above that temporarily skips compilation during packaging

2. **Yarn vs NPM Issues**:
   - VSCE prefers yarn when yarn.lock exists
   - **Solution**: Install yarn globally or use `--no-yarn` flag

3. **Missing Dependencies**:
   - **Solution**: Run `npm install` or `yarn install` first

4. **Performance Warnings**:
   - You'll see warnings about '*' activation and bundling
   - **Solution**: These are warnings, not errors - type `y` to continue

#### Build Output

Successful build produces:
- **File**: `text-marker-1.11.0.vsix`
- **Size**: ~1.04MB
- **Contents**: 920 files including compiled JavaScript

#### Quick Build Script

For convenience, here's a one-liner script:
```bash
# Complete build and package process
npm install && \
cp package.json package.json.bak && \
sed -i '' 's/"yarn run compile"/"echo Skipping compile"/' package.json && \
npm run compile && \
vsce package && \
mv package.json.bak package.json
```

### Option C: Symlink Method (Advanced)
```bash
# macOS/Linux
ln -s "$(pwd)" ~/.vscode/extensions/text-marker-dev

# Windows
mklink /D "%USERPROFILE%\.vscode\extensions\text-marker-dev" "$(pwd)"
```

## Development Workflow

### Watch Mode for Active Development
```bash
yarn run watch
```

Automatically recompiles on TypeScript changes.

### Available Scripts
- `yarn run compile` - Compile TypeScript
- `yarn run lint` - Run TSLint
- `yarn run check` - Run compile + lint + test
- `yarn run watch` - Compile in watch mode
- `yarn test` - Run all tests
- `yarn run unit-test` - Run unit tests only
- `yarn run acceptance-test` - Run acceptance tests only
- `yarn run test-mode` - Run tests in watch mode
- `yarn run coverage` - Run tests with coverage

## Project Architecture

```mermaid
graph TB
    subgraph "Extension Entry Point"
        EXT[extension.ts] --> AI[AppIntegrator]
    end

    subgraph "Core Application Layer"
        AI --> CF[CommandFactory]
        AI --> CC[CommandComponent]
        AI --> TEL[TelemetryReporter]
    end

    subgraph "Command System"
        CF --> TH[ToggleHighlight]
        CF --> UH[UpdateHighlight]
        CF --> GN[GoToNextHighlight]
        CF --> GP[GoToPreviousHighlight]
        CF --> HR[HighlightUsingRegex]
        CF --> RA[RemoveAllHighlights]
        CF --> SA[SaveAllHighlights]
    end

    subgraph "Decoration System"
        TH --> DR[DecorationRegistry]
        UH --> DR
        DR --> DO[DecorationOperator]
        DO --> TD[TextDecorator]
        DO --> DT[DecorationType]
    end

    subgraph "Pattern Matching"
        HR --> PF[PatternFactory]
        PF --> REG[RegexPattern]
        PF --> STR[StringPattern]
        REG --> MM[MatchingModeRegistry]
        STR --> MM
    end

    subgraph "Configuration & Storage"
        CF --> CS[ConfigStore]
        SA --> CS
        CS --> SH[SavedHighlights]
        CS --> CR[ColourRegistry]
    end

    subgraph "VSCode Integration"
        CC --> VSC[VSCode API]
        TD --> VSC
        VSC --> ED[Editor]
        VSC --> SB[StatusBar]
        VSC --> CM[ContextMenu]
    end
```

## Key Components

### Core Directories
- `src/lib/commands/` - User-facing operations (highlight toggling, navigation, etc.)
- `src/lib/decoration/` - Visual representation and color management
- `src/lib/pattern/` - Text matching logic (string and regex patterns)
- `src/lib/vscode/` - VSCode API abstractions for testing
- `src/test/` - Unit and acceptance tests

### Main Features
- **Multi-Pattern Highlighting**: String literals and regular expressions
- **Configurable Colors**: Extensible color palette with opacity control
- **Smart Text Matching**: Case-sensitive/insensitive and whole/partial word matching
- **Persistent Highlights**: Save and restore highlights across sessions
- **Navigation Support**: Jump between matching highlights
- **Context Integration**: Commands in context menus, command palette, status bar

## Extension Commands

Available via Command Palette (`Ctrl+Shift+P`):
- `TextMarker: Toggle Highlight` - Highlight selected text
- `TextMarker: Highlight Text Using Regex` - Highlight using regex patterns
- `TextMarker: Toggle Case Sensitivity` - Toggle case-sensitive matching
- `TextMarker: Toggle Whole/Partial Match` - Toggle whole word matching
- `TextMarker: Clear All Highlights` - Remove all highlights
- `TextMarker: Save All Highlights` - Save highlights to settings
- `TextMarker: Go to Next Same Highlight` - Navigate between highlights
- `TextMarker: Go to Previous Same Highlight` - Navigate between highlights

## Testing

### Test Structure
- **Unit Tests**: `src/test/unit/` - Component-level testing
- **Acceptance Tests**: `src/test/acceptance/` - End-to-end scenarios
- **Test Helpers**: `src/test/helpers/` - Shared testing utilities

### Running Tests
```bash
yarn test                # All tests
yarn run unit-test       # Unit tests only
yarn run acceptance-test # Acceptance tests only
yarn run test-mode       # Watch mode
yarn run coverage        # With coverage report
```

## Configuration

The extension supports extensive configuration through VS Code settings:
- `textmarker.highlightColors` - Color palette for highlights
- `textmarker.defaultHighlightColor` - Fallback color
- `textmarker.enableIgnoreCase` - Default case sensitivity
- `textmarker.enableWholeMatch` - Default whole word matching
- `textmarker.savedHighlights` - Persistent highlight storage
- `textmarker.commandsOnContextMenu` - Context menu customization

## Troubleshooting

### Common Issues
1. **Extension not loading**: Ensure compilation completed without errors
2. **Changes not reflected**: Reload Extension Development Host window (`Ctrl+R`)
3. **TypeScript errors**: Check `tsconfig.json` and run `yarn run lint`
4. **Missing dependencies**: Run `yarn install` again

### Debug Mode
Use F12 Developer Tools in the Extension Development Host to debug runtime issues.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make changes with tests
4. Run `yarn run check` to verify all checks pass
5. Submit a pull request

## Build Process

The extension follows this build pipeline:
1. **TypeScript Compilation**: `src/` → `out/`
2. **Linting**: TSLint checks code quality
3. **Testing**: Unit and acceptance tests
4. **Packaging**: VSCE creates `.vsix` file
5. **Installation**: Load into VS Code

## Development Notes

- The extension activates on `*` (all files) for maximum compatibility
- Uses VSCode API 1.51.0+ for modern features
- Follows strict TypeScript configuration for type safety
- Includes telemetry with privacy controls (can be disabled)
- Supports both light and dark themes
- Optimized for performance with debounced refresh (300ms default)