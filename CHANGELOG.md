# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.1] - 2025-12-31

### Added
- Live demo gif in README showcasing TUI features and workflow
- Examples in the `examples/` directory for common use cases
### Fixed
- Standardized parameter naming conventions in documentation

## [0.1.0] - 2025-12-30

### Added
- Initial release of PRISM
- Interactive TUI (Terminal User Interface) for experiment management
- CLI for automation and scripting
- Parameter sweep functionality with nominal and positional parameters
- Custom configuration validator support
- Real-time experiment monitoring
- Retry functionality for failed experiments
- YAML-based configuration system
- Support for choice, range, and uniform sweep types
- Metric tracking from stdout JSON or log files
- Project configuration via `prism.project.yaml`

### Features
- `prism_tui`: Interactive terminal interface
- `prism`: Command-line interface for automation
- Create and manage studies from base configs and sweep definitions
- Run experiments in parallel or sequentially
- Track experiment progress and status
- Automatic config validation before execution
- Support for Python 3.8+

[0.1.0]: https://github.com/FrancescoCorrenti/prism-sweep/releases/tag/v0.1.0
