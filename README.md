# AI Skills Repository

A collection of AI-agnostic skills for use with various AI coding assistants and agents.

## Structure

This repository contains reusable skill definitions that can be used across different AI platforms.

### Real Directories (Portable Skills)
- `devops-claude-skills/` - DevOps workflows and best practices
- `frontend-design/` - Frontend design patterns and guidelines  
- `go-development/` - Go programming best practices

### Symlinks (Platform-Specific)
The following are symlinks to platform-specific skill locations:
- `bastion` - Autonomous agent delegation patterns
- `cs-c-level` - C-level advisor skills
- `cs-design-director` - Design director skills
- `cs-marketing` - Marketing skills
- `cs-product` - Product management skills
- `phaser-gamedev` - Phaser game development
- `remotion-best-practices` - Remotion video creation

**Note:** Symlinks are platform-specific and won't work when cloned to other systems. Only the real directories above are truly portable.

## Usage

To use these skills with your AI assistant, copy the relevant skill folder to your AI's skills directory.

## Contributing

When adding new skills:
1. Create a new directory (not a symlink) for portable skills
2. Include a `SKILL.md` file describing the skill's purpose and usage
3. Follow the existing directory naming conventions

## License

[Your License Here]
