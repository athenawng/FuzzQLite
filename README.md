# FuzzQLite

## Overview

FuzzQLite is an automated testing tool developed as part of the Automated Software Testing course at ETH Zurich. It evaluates the reliability of SQLite database engines by employing various testing techniques such as mutation and generation-based fuzzing to detect crashes and logic bugs.

## Features

-   **Multiple Fuzzing Strategies**: Uses both mutation and generation-based fuzzing
-   **Differential Testing**: Compares behavior between SQLite versions to detect logic bugs
-   **Coverage Optimization**: Optimizes for path coverage or statement coverage in SQLite source code
-   **Grammar Coverage**: Can optimize for SQLite grammar coverage
-   **Bug Reproducers**: Automatically saves queries that trigger bugs for later analysis

## Requirements

-   Docker and Docker Compose
-   Internet connection (for initial base Docker image download)

## Installation

Clone the repository and navigate to the project directory:

```bash
git clone https://github.com/athenawng/FuzzQLite
cd FuzzQLite
```

## Usage

### Running with Docker Compose

FuzzQLite can be run using Docker Compose:

```bash
docker compose run --rm fuzzqlite
```

This command builds the Docker image and starts the fuzzer with default settings.

### Command Line Options

FuzzQLite supports several command line options:

```
usage: main.py [-h] [--seed SEED] [--trials TRIALS] [--path-coverage] [--grammar-coverage]

FuzzQLite - SQLite Fuzzer

options:
  -h, --help                show this help message and exit
  --seed SEED               Random seed for reproducibility
  --trials TRIALS           Number of fuzzing trials to run (default: 1000)
  --path-coverage           Maximize source code path coverage (if not set, statement coverage is used)
  --grammar-coverage        Maximize SQLite grammar coverage (if not set, grammar coverage is not used)
```

### Examples

Run 10,000 fuzzing trials with statement coverage optimization:

```bash
docker compose run --rm fuzzqlite --trials 10000
```

> **Note:** This is the fastest option.

Run 10,000 fuzzing trials with path coverage and grammar coverage optimization:

```bash
docker compose run --rm fuzzqlite --trials 10000 --path-coverage --grammar-coverage
```

> **Note:** This is the slowest option and may take a long time to complete.

### Interactive Shell Access

For debugging or examining the system directly, you can access an interactive shell:

```bash
docker compose run --rm shell
```

## How It Works

FuzzQLite uses a combination of strategies to find bugs in SQLite. The big picture is as follows:

1. **Database Generation**: Randomly creates SQLite databases with various tables and data types
2. **Seed Query Generation**: Creates schema-based SQL queries using the previously generated databases
3. **Mutation & Generation Cycle**: Alternates randomly between:
    - **Mutation**: Modifies existing queries in the population to create new variants
    - **Grammar-based Query Generation**: Generates new SQL queries based on SQLite grammar rules
4. **Coverage Tracking**: Monitors source code coverage to guide fuzzing towards more promising areas. If a query increases code coverage, it's added to the population for future mutations.
5. **Logic Bug Detection**: Using differential testing, it compares the behavior of different SQLite versions to identify logic bugs
6. **Crash Detection**: Monitors for crashes and hangs during query execution
7. **Reference Error Detection**: Detects if the reference SQLite version crashes or hangs during differential testing

When a bug is detected, a bug reproducer is automatically generated and saved to `bug_reproducers/`.

### Bug Reproducers

Each bug reproducer is stored in its own directory with a standardized structure:

```
bug_reproducers/
└── [SQLite version]/
    ├── crashes/
    │   └── crash_[SQLite version]_[timestamp]/
    │       ├── README.md                       # Description of the bug
    │       ├── original_test.sql               # Original SQL query that triggered the bug
    │       ├── reduced_test.sql                # Minimized SQL query that still triggers the bug
    │       ├── test.db                         # Database file before the query was executed
    │       └── version.txt                     # SQLite version where the bug was found
    ├── logic_bugs/
    │   └── logic_bug_[SQLite version]_[timestamp]/
    │       ├── README.md
    │       ├── original_test.sql
    │       ├── reduced_test.sql
    │       ├── test.db
    │       └── version.txt
    └── reference_errors/
        └── reference_error_[SQLite version]_[timestamp]/
            ├── README.md
            ├── original_test.sql
            ├── reduced_test.sql
            ├── test.db
            └── version.txt
```

## License

[MIT](https://opensource.org/license/MIT)
