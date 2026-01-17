# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).


## [0.2.0] - 2026-01-17

### Added
- **Inspect & Edit Configurations**: New `inspect_config` feature in TUI to view, edit, and export experiment configurations
  - View full YAML config with syntax highlighting
  - Edit individual parameters using dot notation (e.g., `model.lr`)
  - Flat view showing all parameters with types
  - Export configurations to YAML files
- **Bulk Delete Experiments**: Delete multiple experiments at once using filter criteria
  - Support for filter operators: `=`, `>`, `<`, `>=`, `<=`, `!=`
  - Filter by experiment status
  - Preview matching experiments before deletion
- **Rules-Based Validation**: New `prism.rules.yaml` file for advanced configuration filtering
  - Define rules with conditions and actions (`error`, `skip`, `skip-warning`, `include-warning`)
  - Support for operators: `$gt`, `$gte`, `$lt`, `$lte`, `$ne`, `$in`, `$nin`, `$regex`, `$exists`
  - Automatic rule discovery and application during sweep generation
  - Example rules file in `examples/prism.rules.yaml`

### Changed
- **Manager API**: Added new methods for configuration manipulation
  - `update_experiment_config()`: Update a single parameter in an experiment
  - `update_experiment_configs_bulk()`: Update multiple parameters in multiple experiments
  - `find_experiments_by_filter()`: Find experiments matching filter criteria
  - `delete_experiments()`: Delete experiments by keys
  - `delete_experiments_by_filter()`: Delete experiments matching filters
- **TUI**: Replaced `show_config` with `inspect_config` for interactive configuration management
- **expand_configs()**: Now automatically finds and applies rules from `prism.rules.yaml`

### Improved
- Better table layouts in TUI with adaptive column widths
- Numbers and experiment names always visible regardless of terminal size

## [0.1.4] - 2026-01-05
### Added
- Special `@` prefix syntax for string values to force list values to be treated as scalar (non-sweep) values. Use `param: "@[1, 2, 3]"` when you want to pass the entire list as a single value instead of creating a sweep with 3 experiments.

## [0.1.3] - 2026-01-05
### Fixed
- Fixed scalar parameters from prism files being ignored in multi-file cartesian product mode. When using multiple prism files where some have nominal parameters (`$key`) and others only have scalar parameters, the scalars were not being applied to generated experiments. Now all scalar parameters are correctly applied before nominal/positional parameter substitution.

## [0.1.2] - 2026-01-03

### Added
- Command `reload config` in TUI and CLI to reload the configuration without resetting the experiments.

### Fixed
- Training is executed with the same Python interpreter that launched PRISM (`sys.executable`), ensuring the active environment is respected in both CLI and TUI

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
[0.1.1]: https://github.com/FrancescoCorrenti/prism-sweep/releases/tag/v0.1.1
[0.1.2]: https://github.com/FrancescoCorrenti/prism-sweep/releases/tag/v0.1.2
[0.1.3]: https://github.com/FrancescoCorrenti/prism-sweep/releases/tag/v0.1.3
[0.1.4]: https://github.com/FrancescoCorrenti/prism-sweep/releases/tag/v0.1.4
[0.2.0]: https://github.com/FrancescoCorrenti/prism-sweep/releases/tag/v0.2.0
