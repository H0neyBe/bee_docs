# Contributing to HoneyBee

Thank you for your interest in contributing to HoneyBee!

## How to Contribute

### Reporting Issues

- Use GitHub Issues for bug reports
- Include steps to reproduce
- Provide logs and error messages
- Specify component (Core, Node, Potstore)

### Submitting Code

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

### Code Style

- **Rust (Core)**: Follow Rust standard formatting (`rustfmt`)
- **Go (Node)**: Follow Go standard formatting (`gofmt`)
- **Documentation**: Use Markdown, follow existing style

## Component-Specific Guidelines

### HoneyBee Core

- Follow Rust best practices
- Use async/await (Tokio)
- Add tests for new features
- Update documentation

### HoneyBee Node

- Follow Go best practices
- Use structured logging
- Add error handling
- Update configuration docs

### HoneyBee Potstore

- Follow existing honeypot structure
- Include installation scripts
- Add HoneyBee integration
- Update potstore.json

## Testing

- Run existing tests before submitting
- Add tests for new features
- Test on multiple platforms if applicable

## Documentation

- Update relevant documentation
- Add examples for new features
- Keep documentation in sync with code

## Next Steps

- [Building from Source](./building.md) - Build components
- [Creating Custom Nodes](./custom-nodes.md) - Build nodes
- [Testing](./testing.md) - Testing guide

