# Interline Transitland <!-- omit in toc -->

`transitland-lib` is a library and command-line tool for reading, writing, and processing transit data in [GTFS](http://gtfs.org) and related formats. The library is structured as a set of data sources, filters, and transformations that can be mixed together in a variety of ways to create processing pipelines. The library supports the [DMFR](https://github.com/transitland/distributed-mobility-feed-registry) format to describe feed resources.

![Test & Release](https://github.com/interline-io/transitland-lib/workflows/Test%20&%20Release/badge.svg) [![GoDoc](https://godoc.org/github.com/interline-io/transitland-lib/tl?status.svg)](https://godoc.org/github.com/interline-io/transitland-lib/tl) ![Go Report Card](https://goreportcard.com/badge/github.com/interline-io/transitland-lib)

## Key Features

- Modular readers and writers for GTFS, CSV, SQLite and PostgreSQL
- Pipelines for fetching, validating and importing feeds
- CLI utilities for diffing, merging and real-time conversions
- Extensible filters to customize feed processing
- Usable as a Go library for bespoke data workflows

## Table of Contents <!-- omit in toc -->
<!-- to update use https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one -->
- [Key Features](#key-features)
- [Installation](#installation)
        - [Download prebuilt binary](#download-prebuilt-binary)
        - [Install using homebrew](#install-using-homebrew)
        - [Install binary from source](#install-binary-from-source)
- [Usage as a CLI tool](#usage-as-a-cli-tool)
	- [Breaking changes](#breaking-changes)
- [Usage as a library](#usage-as-a-library)
- [Database migrations](#database-migrations)
- [Usage as a Web Service](#usage-as-a-web-service)
- [Included Readers and Writers](#included-readers-and-writers)
- [Repository layout](#repository-layout)
- [Development](#development)
        - [Contributing](#contributing)
        - [Releases](#releases)
- [Licenses](#licenses)

## Installation

### Download prebuilt binary

The `transitland` binaries for Linux and macOS are attached to each [release](https://github.com/interline-io/transitland-lib/releases).

### Install using homebrew

The `transitland` binary can be installed using homebrew. The executable is code-signed and notarized.

```bash
brew install interline-io/transitland-lib/transitland-lib
```

### Install binary from source

```bash
go get github.com/interline-io/transitland-lib/cmd/transitland
```

This package uses Go Modules and will also install required dependencies.

Main dependencies:
- `twpayne/go-geom`
- `jmoiron/sqlx`
- `Masterminds/squirrel`
- `jackc/pgx`
- `mattn/go-sqlite3` (requires CGO)

## Usage as a CLI tool

The main subcommands are:
* [transitland copy](doc/cli/transitland_copy.md)	 - Copy performs a basic copy from a reader to a writer.
* [transitland diff](doc/cli/transitland_diff.md)	 - Calculate difference between two feeds, writing output in a GTFS-like format
* [transitland dmfr-format](doc/cli/transitland_dmfr-format.md)	 - Lint DMFR files
* [transitland dmfr-lint](doc/cli/transitland_dmfr-lint.md)	 - Format a DMFR file
* [transitland extract](doc/cli/transitland_extract.md)	 - Extract a subset of a GTFS feed
* [transitland fetch](doc/cli/transitland_fetch.md)	 - Fetch GTFS data and create feed versions
* [transitland import](doc/cli/transitland_import.md)	 - Import feed versions
* [transitland merge](doc/cli/transitland_merge.md)	 - Merge multiple GTFS feeds
* [transitland sync](doc/cli/transitland_sync.md)	 - Sync DMFR files to database
* [transitland unimport](doc/cli/transitland_unimport.md)	 - Unimport feed versions
* [transitland validate](doc/cli/transitland_validate.md)	 - Validate a GTFS feed
* [transitland rt-convert](doc/cli/transitland_rt-convert.md)	 - Convert GTFS-RealTime to JSON

See the [full list of subcommands](doc/cli/transitland.md)

### Breaking changes

Note: as of v0.17, we moved from Go standard library `flags` to Cobra's `pflags`; this is a breaking change in that single-dash (`-flag`) command flags are no longer supported, only double-dash (`--flag`).

## Usage as a library

See [library examples](doc/library-example.md).

## Database migrations

Migrations are supported for PostgreSQL, using the schema files in `internal/schema/postgres/migrations`. These files can be read and applied using [golang-migrate](https://github.com/golang-migrate/migrate), which will store the most recently applied migration version in `schema_migrations`. See the `bootstrap.sh` script in that directory for an example, as well as details on how to import Natural Earth data files for associating agencies with places.

SQLite database are intended to be short-lived. They can be created on an as needed basis by passing the `-create` flag to some commands that accept a writer. They use a single executable schema, defined in `internal/schema/sqlite.sql`.

## Usage as a Web Service

See [transitland-server](https://github.com/interline-io/transitland-server) documentation.

## Included Readers and Writers

| Target                   | Module  | Supports Read | Supports Write |
| ------------------------ | ------- | ------------- | -------------- |
| CSV                      | `tlcsv` | ✅             | ✅              |
| SQLite                   | `tldb`  | ✅             | ✅              |
| PostgreSQL (with PostGIS)  | `tldb`  | ✅             | ✅              |

We welcome the addition of more readers and writers.

## Repository layout

The top level of this repository is organized into Go packages that make up the
library and supporting directories used for documentation and tests.

### Go packages

```
adapters/   - reusable readers and writers
causes/     - validation result helpers
cmd/        - main entrypoint for the `transitland` binary
cmds/       - individual CLI subcommands
copier/     - utilities for copying feed data
diff/       - diff command implementation
dmfr/       - DMFR feed metadata types
ext/        - GTFS extensions
extract/    - extracting subsets of feeds
fetch/      - fetch GTFS and realtime data
filters/    - data filtering utilities
gtfs/       - core GTFS models
importer/   - import pipeline
internal/   - shared packages
request/    - HTTP/FTP/Azure/S3 helpers
rt/         - GTFS-RealTime support
rules/      - validation rules
schema/     - database schemas
service/    - service layer utilities
stats/      - feed statistics
sync/       - DMFR sync helpers
tlcli/      - CLI helper utilities
tlcsv/      - CSV helpers
tldb/       - database helpers
tlxy/       - geometry helpers
tt/         - typed data primitives
validator/  - feed validation library
```

### Infrastructure & tests

```
doc/        - generated documentation
testdata/   - sample feeds and other fixtures for tests
```

## Development

`transitland-lib` follows Go coding conventions.

GitHub Actions runs all tests, stores code coverage reports as artifacts, and prepares releases.

### Contributing

1. Fork the repository and create a branch for your work.
2. Run `go mod tidy` and `go generate ./...` to update dependencies and generated files.
3. Run `go test ./...` and ensure all tests pass.
4. Format code with `gofmt -s -w` before committing.
5. Open a pull request with a clear description of your changes.

### Releases

Releases follow [Semantic Versioning](https://semver.org/) conventions.

To cut a new release:

1. Run `go generate ./...` to update auto-generated documentation.
2. Create a GitHub release. This will create a tag and GitHub Actions will create &amp; attach code-signed binaries.
3. Download the files from the release, and update the [homebrew formula](https://github.com/interline-io/homebrew-transitland-lib/blob/master/transitland-lib.rb) with the updated sha256 hashes and version tag.

## Licenses

`transitland-lib` is released under a "dual license" model:

- open-source for use by all under the [GPLv3](LICENSE) license
- also available under a flexible commercial license from [Interline](mailto:info@interline.io)

