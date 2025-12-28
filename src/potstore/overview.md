# HoneyBee Potstore Overview

The HoneyBee Potstore is a repository of pre-configured honeypots ready for deployment via HoneyBee nodes.

## What is the Potstore?

The Potstore is a GitHub repository containing:

- **Pre-configured Honeypots**: Ready-to-deploy honeypot implementations
- **HoneyBee Integration**: All honeypots configured to forward events to nodes
- **Installation Scripts**: Automated setup for each honeypot
- **Standardized Format**: Consistent structure across all honeypots

## Repository

**GitHub**: [https://github.com/H0neyBe/honeybee_potstore](https://github.com/H0neyBe/honeybee_potstore)

## How It Works

1. **Core sends InstallPot command** to a node
2. **Node clones Potstore** from GitHub
3. **Node installs specified honeypot** with dependencies
4. **Node configures HoneyBee integration** automatically
5. **Honeypot forwards events** to node (port 9100)

## Available Honeypots

### Currently Available

- **Cowrie** - SSH and Telnet honeypot (Python/Twisted)
- **HonnyPotter** - WordPress login honeypot (PHP)

### Coming Soon

- Dionaea - Multi-protocol honeypot
- Heralding - Credential honeypot
- Elasticpot - Elasticsearch honeypot
- Mailoney - SMTP honeypot

## Potstore Manifest

The `potstore.json` file contains metadata for all honeypots:

- Honeypot type and version
- Requirements (Python, PHP, etc.)
- Default ports
- Installation instructions
- HoneyBee integration details

## Directory Structure

```
honeybee_potstore/
├── README.md
├── LICENSE
├── potstore.json        # Honeypot manifest
├── cowrie/              # Cowrie SSH/Telnet honeypot
│   ├── requirements.txt
│   ├── src/
│   │   └── cowrie/
│   │       └── output/
│   │           └── honeybee.py  # HoneyBee output plugin
│   └── ...
├── HonnyPotter/         # WordPress login honeypot
│   ├── standalone.php
│   ├── honeybee-forwarder.php
│   ├── install.sh
│   └── ...
└── [future-pots]/       # Future honeypots
```

## Next Steps

- [Available Honeypots](./honeypots.md) - List of all honeypots
- [Using Honeypots](./using.md) - How to use honeypots
- [Adding New Honeypots](./adding.md) - Add your own honeypots
- [Integration Guide](./integration.md) - HoneyBee integration details

