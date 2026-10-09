---
name: ios-agent
description: Drive an iOS Simulator or a real iPhone through the ios-mcp MCP server. Use when asked to check a change in a running iOS app, change a setting on a device, read an answer out of an app with no API, or walk a flow on a phone.
---

# Driving an iPhone with ios-mcp

ios-mcp is an MCP server that gives an agent an iOS Simulator or a real
iPhone, through Apple's XCUIAutomation via WebDriverAgent. It runs on the
user's own Mac: it needs macOS, Xcode 16.3+, an iOS runtime and Python 3.12+.

Full documentation: https://emazaheri.github.io/ios-agent/

## Set up

```bash
uvx ios-mcp prepare-wda simulator   # builds WebDriverAgent, about 20 seconds, once
uvx ios-mcp doctor                  # says what, if anything, is missing
```

Register it with the client:

```json
{ "mcpServers": { "ios": { "command": "uvx", "args": ["ios-mcp"] } } }
```

For Claude Code: `claude mcp add ios -s user -- uvx ios-mcp`.

A physical iPhone needs more: go-ios, Developer Mode and a signing identity.
Follow https://emazaheri.github.io/ios-agent/guides/physical-device/.

## The loop

1. `ios_open_session` boots or verifies the device and returns the first
   screen as a digest: a few hundred tokens naming each element by a ref such
   as `e2`.
2. Act by ref, never by coordinates: `ios_tap`, `ios_type`, `ios_set_value`,
   `ios_scroll`. Every action returns the screen it produced and
   `screen_changed`, so do not observe again after acting; read the result.
3. If `screen_changed` is false, the action did nothing. Do not report success.
4. Prefer `ios_open_url` with a deep link to reach a screen in one step.
5. Prefer `ios_set_value` over tapping a switch: it checks the current state.
6. When the digest seems to be missing something, `ios_find` searches the
   whole accessibility tree before compaction.
7. `ios_close_session` when done.

The server ships an `ios_operator` prompt with the full rules.

## Safety

Anything that sends, pays, deletes or reaches another person asks the user
first, through MCP elicitation or a signature handed back for the retry. Do not
try to route around it. Never type a real credential with `ios_type`; use
`ios_type_secret`, which takes a keychain reference rather than a value.

## When something fails

Call `ios_doctor`: each failing check comes with a remedy. A leftover
WebDriverAgent holding the device is cleared with `uvx ios-mcp reset -y`. See
https://emazaheri.github.io/ios-agent/guides/troubleshooting/.

Tool reference: https://emazaheri.github.io/ios-agent/reference/tools/
