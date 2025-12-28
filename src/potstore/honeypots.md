# Available Honeypots

Complete list of honeypots available in the HoneyBee Potstore.

## Cowrie

**Type**: SSH/Telnet Honeypot  
**Language**: Python  
**Version**: 2.9.0  
**Status**: ✅ Stable

### Description

Medium to high interaction SSH and Telnet honeypot that logs brute force attacks and shell interaction.

### Features

- SSH honeypot
- Telnet honeypot
- Command logging
- File download tracking
- Session recording
- Custom output plugins

### Requirements

- Python 3.7+
- Linux, Windows, or macOS

### Default Ports

- SSH: 2222
- Telnet: 2223

### Installation

Automatically installed by HoneyBee Node when Core sends `InstallPot` command.

### Configuration

```json
{
  "pot_id": "cowrie-01",
  "honeypot_type": "cowrie",
  "config": {
    "ssh_port": "2222",
    "telnet_port": "2223"
  }
}
```

### HoneyBee Integration

- Output plugin: `src/cowrie/output/honeybee.py`
- Events forwarded to: `localhost:9100`
- Format: JSON (one event per line)

## HonnyPotter

**Type**: WordPress Login Honeypot  
**Language**: PHP  
**Version**: 1.2.0  
**Status**: ✅ Stable

### Description

WordPress login honeypot that captures brute-force attacks and credential stuffing attempts.

### Features

- WordPress login emulation
- XML-RPC endpoint support
- Brute-force attack detection
- Credential logging
- Low resource usage
- No database required
- Standalone or WordPress plugin mode

### Requirements

- PHP 7.4+
- PHP extensions: json, sockets
- Linux, Windows, or macOS

### Default Ports

- HTTP: 80
- HTTPS: 443

### Installation

Automatically installed by HoneyBee Node when Core sends `InstallPot` command.

### Configuration

```json
{
  "pot_id": "honnypotter-01",
  "honeypot_type": "honnypotter",
  "config": {
    "http_port": "8080"
  }
}
```

### HoneyBee Integration

- Forwarder: `honeybee-forwarder.php`
- Events forwarded to: `localhost:9100`
- Format: JSON

### Usage Modes

1. **Standalone**: Run `standalone.php` directly
2. **WordPress Plugin**: Install as WordPress plugin

## Coming Soon

### Dionaea

Multi-protocol honeypot supporting FTP, HTTP, SMB, MySQL, and more.

### Heralding

Credential honeypot that captures authentication attempts.

### Elasticpot

Elasticsearch honeypot for detecting Elasticsearch attacks.

### Mailoney

SMTP honeypot for capturing email-based attacks.

## Honeypot Comparison

| Honeypot | Type | Protocols | Language | Status |
|----------|------|-----------|----------|--------|
| Cowrie | SSH/Telnet | SSH, Telnet | Python | ✅ Stable |
| HonnyPotter | Web | HTTP, HTTPS | PHP | ✅ Stable |
| Dionaea | Multi | FTP, HTTP, SMB, MySQL | C | 🚧 Planned |
| Heralding | Credential | Multiple | Python | 🚧 Planned |
| Elasticpot | Search | HTTP | Python | 🚧 Planned |
| Mailoney | Email | SMTP | Python | 🚧 Planned |

## Next Steps

- [Using Honeypots](./using.md) - How to deploy and use honeypots
- [Adding New Honeypots](./adding.md) - Contribute new honeypots
- [Integration Guide](./integration.md) - Technical integration details

