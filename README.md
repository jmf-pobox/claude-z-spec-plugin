# Z Specification Plugin for Claude Code

Create, validate, and test formal Z specifications for stateful systems using fuzz and ProB.

## Features

- **Generate Z specs** from codebase analysis or system descriptions
- **Type-check** with fuzz
- **Animate and model-check** with probcli (ProB)
- **Elaborate** specs with narrative from design documentation
- **ProB-compatible** output (avoids B keyword conflicts, bounded integers, flat schemas)

## Installation

### Prerequisites

**fuzz** - Z type-checker
```bash
# Install from https://github.com/Spivoxity/fuzz
# Verify:
fuzz -version
```

**probcli** - ProB command-line interface
```bash
# Install from https://prob.hhu.de/w/index.php/Download
# Add to shell profile:
export PROBCLI="$HOME/Applications/ProB/probcli"
# Verify:
$PROBCLI -version
```

### Plugin Installation

#### Option 1: Local Marketplace (Recommended)

```bash
# Create local marketplace if it doesn't exist
mkdir -p ~/.claude/plugins/local-plugins/.claude-plugin
mkdir -p ~/.claude/plugins/local-plugins/plugins

# Clone this repo
git clone https://github.com/jmf-pobox/claude-z-spec-plugin.git \
    ~/.claude/plugins/local-plugins/plugins/z-spec

# Create marketplace.json
cat > ~/.claude/plugins/local-plugins/.claude-plugin/marketplace.json << 'EOF'
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "local",
  "description": "Local plugins",
  "owner": { "name": "local", "email": "local@example.com" },
  "plugins": [{
    "name": "z-spec",
    "description": "Create, validate, and test formal Z specifications",
    "version": "1.0.0",
    "source": "./plugins/z-spec",
    "category": "development"
  }]
}
EOF

# Register marketplace and install
claude plugin marketplace add ~/.claude/plugins/local-plugins
claude plugin install z-spec@local
```

#### Option 2: Direct Copy

```bash
# Clone directly to plugins directory
git clone https://github.com/jmf-pobox/claude-z-spec-plugin.git \
    ~/.claude/plugins/z-spec
```

## Commands

| Command | Description |
|---------|-------------|
| `/z create [focus]` | Generate a Z specification from codebase or description |
| `/z check [file]` | Type-check a specification with fuzz |
| `/z test [file]` | Validate and animate with probcli |
| `/z update [file] [changes]` | Modify an existing specification |
| `/z elaborate [spec] [design]` | Enhance spec with narrative from design docs |
| `/z help` | Show quick reference |

## Quick Start

### Create a Specification

```
/z create the user authentication system
```

Analyzes the codebase and generates a formal Z specification in `docs/`.

### Validate

```
/z check docs/auth.tex      # Type-check with fuzz
/z test docs/auth.tex       # Animate with probcli
```

### Elaborate with Design Narrative

```
/z elaborate docs/auth.tex DESIGN.md
```

Transforms a minimal spec into polished documentation with:
- Table of contents
- Design philosophy
- Semantic explanations
- Algorithm summaries
- Validation status

### Update

```
/z update docs/auth.tex add a logout operation
```

## Specification Structure

Generated specs follow this structure:

1. **Basic Types** - Given sets (`[USERID, TIMESTAMP]`)
2. **Free Types** - Enumerations (`Status ::= active | inactive`)
3. **Global Constants** - Configuration values
4. **State Schemas** - Entities with invariants
5. **Initialization** - Valid initial states
6. **Operations** - State transitions
7. **System Invariants** - Key properties summary

## ProB Compatibility

The plugin generates specs that work with both fuzz and probcli:

| Issue | Solution |
|-------|----------|
| B keyword conflict | Use `ZBOOL ::= ztrue \| zfalse` |
| Abstract functions | Provide concrete mappings |
| Unbounded integers | Add bounds in invariants |
| Nested schemas | Flatten into single State schema |
| Unbounded inputs | Add upper bounds to inputs |

## Reference Files

- `reference/z-notation.md` - Z notation cheat sheet
- `reference/schema-patterns.md` - Common patterns and ProB tips
- `reference/probcli-guide.md` - probcli command reference

## Example Output

```latex
\begin{schema}{State}
level : \nat \\
attempts : \nat \\
correct : \nat
\where
level \geq 1 \\
level \leq 26 \\
correct \leq attempts \\
attempts \leq 10000
\end{schema}

\begin{schema}{AdvanceLevel}
\Delta State \\
accuracy? : \nat
\where
accuracy? \geq 90 \\
accuracy? \leq 100 \\
level < 26 \\
level' = level + 1 \\
attempts' = attempts \\
correct' = correct
\end{schema}
```

## License

MIT License - see [LICENSE](LICENSE)
