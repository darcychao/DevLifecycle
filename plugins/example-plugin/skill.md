# Example Plugin Skill

## Purpose
Demonstrates how a plugin can define a custom skill that agents can invoke during the development lifecycle.

## When to Use
This skill is invoked by the Dev Agent before and after Phase 5 (Development Coding).

## Pre-Development Checklist (dev-pre)
- Verify all prerequisite documents are approved
- Confirm development environment is ready
- Check for pending challenges

## Post-Development Quality Check (dev-post)
- Run linter, formatter, type checker
- Run test suite
- Generate change summary

## Usage
This plugin is automatically activated if present in `plugins/example-plugin/`. No manual invocation needed — the orchestrator picks it up via `plugin.json`.
