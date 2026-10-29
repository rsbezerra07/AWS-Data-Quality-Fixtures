# AWS Data Quality Fixtures

A repository of synthetic, deterministic, and reproducible datasets designed to
test data quality, data contracts, schema evolution, duplicate processing,
late-arriving data, and incompatible contract versions within the DataTrust AWS
ecosystem.

The purpose of this project is to provide test inputs whose expected behavior is
known before execution.

In other words:

> known input + known contract + known expected result = reproducible test

This repository does not implement AWS infrastructure or the main DataTrust
platform. Its responsibility is to produce and validate reusable fixtures that can
be consumed by the other projects in the DataTrust AWS family.

---

## Objective

`aws-data-quality-fixtures` exists to produce reproducible synthetic datasets with
known valid and defective outcomes.

The main outputs of the project are:

- fixtures;
- checksums;
- expected-result manifests;
- versioned data contracts;
- deterministic data generation;
- privacy and licensing documentation;
- automated tests that verify the integrity of the fixture set.

A fixture set should only be considered complete when it can be regenerated
deterministically and every published fixture has a documented expected outcome.

---

## Scope

### This repository is responsible for

- generating synthetic data from controlled configurations;
- maintaining versioned data contracts;
- generating valid fixtures;
- generating fixtures with invalid schemas;
- generating fixtures with data-quality failures;
- representing duplicate-data scenarios;
- representing late-arriving-data scenarios;
- representing incompatible contract versions;
- recording expected results;
- recording checksums;
- verifying deterministic regeneration;
- verifying artifact integrity;
- documenting the fixture catalog;
- documenting privacy considerations;
- documenting data licensing and provenance.

### Out of scope

This repository should not contain:

- AWS infrastructure;
- the DataTrust platform implementation;
- Athena publication logic;
- Terraform modules for the DataTrust platform;
- quarantine implementation;
- replay implementation;
- cloud observability infrastructure;
- duplicated implementation code from other DataTrust projects.

Those responsibilities belong to the other repositories in the DataTrust AWS
project family.

---

## Repository Structure

    aws-data-quality-fixtures/
    ├── .github/
    │   └── workflows/
    │       └── ci.yml
    │
    ├── fixtures/
    │   ├── valid/
    │   ├── invalid-schema/
    │   ├── invalid-quality/
    │   ├── duplicate/
    │   ├── late-arrival/
    │   └── incompatible-version/
    │
    ├── generators/
    │   ├── __init__.py
    │   ├── generate.py
    │   └── profiles/
    │       ├── base/
    │       └── defects/
    │
    ├── contracts/
    │   └── v1/
    │
    ├── manifests/
    │   ├── expected-results/
    │   └── checksums/
    │
    ├── tests/
    │   ├── test_contracts.py
    │   ├── test_determinism.py
    │   ├── test_expected_results.py
    │   └── test_checksums.py
    │
    ├── scripts/
    │   ├── generate_fixtures.py
    │   ├── verify_fixtures.py
    │   └── generate_checksums.py
    │
    ├── docs/
    │   ├── fixture-catalog.md
    │   ├── privacy.md
    │   └── licensing.md
    │
    ├── .gitignore
    ├── CHANGELOG.md
    ├── pyproject.toml
    └── README.md

---

# Architecture

The repository separates five core concepts:

1. contract;
2. generation profile;
3. fixture;
4. expected result;
5. verification.

This separation makes it possible to distinguish:

    what the data should conform to
                ↓
    how the data is produced
                ↓
    which dataset was produced
                ↓
    what should happen to that dataset
                ↓
    how reproducibility and integrity are verified

---

# Main Flow

The logical flow of the repository is:

    contracts/
        ↓
    generators/profiles/
        ↓
    generators/
        ↓
    fixtures/
        ↓
    manifests/
        ↓
    tests/
        ↓
    CI

---

## 1. Contracts

Versioned contracts are stored under:

    contracts/v1/

Contracts define the expected structure or rules against which the generated
datasets can be interpreted.

Keeping contracts versioned is important because part of the DataTrust ecosystem
is designed to distinguish compatible changes from breaking changes.

---

## 2. Generation Profiles

Generation profiles are stored under:

    generators/profiles/

They describe the different scenarios that can be produced by the generator.

The directory is separated into:

    profiles/
    ├── base/
    └── defects/

### `base/`

Contains the definitions required to produce the baseline or valid form of a
dataset.

### `defects/`

Contains definitions for intentionally introducing defects into generated
datasets.

The purpose of this separation is to distinguish:

    normal dataset generation

from:

    intentional defect injection

This makes failure scenarios explicit rather than embedding them throughout the
generation code.

---

## 3. Generators

Data-generation code is stored under:

    generators/

The primary component is:

    generators/generate.py

It contains the core generation logic used to transform controlled generation
parameters into datasets.

Generation should be deterministic.

Given the same controlled inputs, the repository should be able to reproduce the
same expected fixture.

---

# 4. Fixtures

Generated datasets are stored under:

    fixtures/

Fixtures are categorized according to the scenario they represent.

---

## `fixtures/valid/`

Contains datasets that conform to their corresponding data contracts.

These fixtures represent the positive path.

Conceptually:

    valid fixture
        ↓
    contract validation
        ↓
    ACCEPT

---

## `fixtures/invalid-schema/`

Contains datasets intentionally designed to violate the expected schema.

These fixtures allow consuming systems to test whether structural violations are
detected correctly.

Conceptually:

    invalid schema
        ↓
    schema validation
        ↓
    REJECT / QUARANTINE

---

## `fixtures/invalid-quality/`

Contains datasets whose structure may be valid but whose values intentionally
violate data-quality expectations.

This distinction is important:

    schema problem

is not necessarily the same as:

    data-quality problem

Conceptually:

    valid structure
        +
    invalid data quality
        ↓
    quality validation
        ↓
    QUARANTINE

---

## `fixtures/duplicate/`

Contains datasets or scenarios intended to represent duplicate processing.

These fixtures provide deterministic inputs for systems that must prove
idempotent behavior.

Within the broader DataTrust project, duplicate scenarios are relevant to proving
that duplicate events do not result in duplicate trusted publications.

---

## `fixtures/late-arrival/`

Contains fixtures representing data that arrives later than the normal sequence
assumed by a scenario.

The category keeps late-arriving-data tests separate from:

- duplicate processing;
- schema violations;
- data-quality failures;
- incompatible versions.

---

## `fixtures/incompatible-version/`

Contains datasets intentionally associated with incompatible versions of a
contract.

These fixtures are particularly useful for schema-evolution experiments and for
the `aws-schema-drift-lab` repository.

---

# 5. Manifests

Machine-readable metadata related to the fixtures is stored under:

    manifests/

---

## Expected Results

Expected results are stored under:

    manifests/expected-results/

They define what should happen when a particular fixture is evaluated.

The fundamental relationship is:

    fixture
        +
    expected result
        =
    known test case

A consuming system can therefore compare:

    expected_result

against:

    observed_result

The expected result acts as the test oracle.

The observed result is produced later by the system being tested.

Keeping those concepts separate prevents the expected behavior from being hidden
inside the implementation of the test itself.

---

## Checksums

Checksum information is stored under:

    manifests/checksums/

Checksums provide an integrity reference for published fixture artifacts.

Conceptually:

    fixture
        ↓
    known checksum
        ↓
    regeneration
        ↓
    observed checksum
        ↓
    comparison

Unexpected fixture changes should therefore be detectable before a modified
artifact is considered equivalent to the previous version.

---

# 6. Tests

Automated repository-level tests are stored under:

    tests/

These tests validate the fixture repository itself rather than the DataTrust
platform.

---

## `test_contracts.py`

Validates the consistency of the contracts used by the repository.

Its purpose is to prevent an invalid or inconsistent contract from becoming the
reference for fixture generation.

---

## `test_determinism.py`

Verifies one of the core properties of the repository:

> The same controlled generation should reproduce the same expected data.

Conceptually:

    seed/configuration
            ↓
      generation A
            ↓
        fixture A

    seed/configuration
            ↓
      generation B
            ↓
        fixture B

For a deterministic scenario:

    fixture A == fixture B

---

## `test_expected_results.py`

Validates the relationship between fixtures and their expected-result manifests.

It prevents situations such as:

    fixture without expected result

or:

    expected result without corresponding fixture

Every published fixture should have a documented expected behavior.

---

## `test_checksums.py`

Validates fixture integrity against the known checksum metadata.

Its purpose is to detect unexpected changes between the artifact that should exist
and the artifact currently present in the repository.

---

# 7. Scripts

Operational entry points are stored under:

    scripts/

The intended separation is:

    generators/ = reusable implementation

    scripts/ = operational entry points

This prevents command-line tooling from becoming a second implementation of the
generation logic.

---

## `generate_fixtures.py`

Operational entry point for generating fixtures.

The underlying generation logic should remain in `generators/`.

---

## `generate_checksums.py`

Generates integrity metadata for the published fixture artifacts.

---

## `verify_fixtures.py`

Provides an operational verification entry point for the complete fixture set.

It can act as the high-level verification command used before a fixture package is
considered ready for consumption or release.

---

# 8. Documentation

Project-specific documentation is stored under:

    docs/

---

## `fixture-catalog.md`

Catalog of the available fixtures.

It should make it possible for consumers to identify:

- which scenarios exist;
- where a fixture is stored;
- which contract it relates to;
- what scenario it represents;
- what result is expected.

---

## `privacy.md`

Documents privacy considerations for the datasets.

Public datasets used by this project should be synthetic, sanitized, or otherwise
appropriate for public use.

The goal is to make privacy an explicit property of the fixture package rather
than an assumption.

---

## `licensing.md`

Documents licensing and data provenance considerations.

It complements the repository-level `LICENSE` file.

The distinction is:

    LICENSE
        → repository/software license

    docs/licensing.md
        → dataset licensing and provenance information

---

# Continuous Integration

Repository automation is defined under:

    .github/workflows/ci.yml

CI acts as an automated quality gate.

Its purpose is to ensure that changes to generators, contracts, manifests, or
fixtures do not silently invalidate the fundamental properties of the repository.

Conceptually:

    pull request / commit
            ↓
            CI
            ↓
    contract validation
            ↓
    determinism validation
            ↓
    expected-result validation
            ↓
    checksum validation
            ↓
           PASS

Only a consistent fixture set should become a release candidate.

---

# Governance Files

## `CHANGELOG.md`

Records meaningful changes between fixture releases.

Changes to datasets, contracts, generators, or expected results should be visible
to consumers who depend on a specific release.

---

## `.gitignore`

Prevents local or generated artifacts that do not belong to the published project
from being committed.

---

## `pyproject.toml`

Centralizes Python project configuration and metadata.

---

# End-to-End Flow

The complete logical flow can be summarized as:

    Contract
       │
       ▼
    Generation Profile
       │
       ▼
    Seeded Generator
       │
       ▼
    Synthetic Dataset
       │
       ▼
    Fixture Classification
       │
       ├── valid
       ├── invalid-schema
       ├── invalid-quality
       ├── duplicate
       ├── late-arrival
       └── incompatible-version
       │
       ▼
    Expected-Result Manifest
       │
       ▼
    Checksum
       │
       ▼
    Automated Tests
       │
       ▼
    CI Validation
       │
       ▼
    Versioned Fixture Release

---

# Relationship with DataTrust AWS

This repository is one component of the DataTrust AWS project family.

The overall dependency sequence is:

    aws-data-quality-fixtures
                ↓
    aws-datatrust-platform
                ↓
    terraform-aws-datatrust
                ↓
    aws-schema-drift-lab

Fixtures come first because platform validation requires deterministic test inputs.

---

## `aws-data-quality-fixtures`

Responsible for:

    known inputs

It provides reproducible datasets and expected outcomes.

---

## `aws-datatrust-platform`

Responsible for:

    making dataset trust decisions

The platform consumes datasets, contract versions, metadata, and replay requests.

Its outputs include trusted or quarantined objects, execution state, evidence
manifests, and publication that can be verified through Athena.

This repository must not duplicate that runtime logic.

---

## `terraform-aws-datatrust`

Responsible for:

    reusable infrastructure

Validated infrastructure from the working platform can later be extracted into
reusable Terraform modules.

That infrastructure remains outside the fixture repository.

---

## `aws-schema-drift-lab`

Responsible for:

    contract and schema evolution experiments

It consumes released fixtures and the other released DataTrust components to
demonstrate scenarios such as:

- contract baselines;
- compatible evolution;
- breaking changes;
- migration;
- quarantine;
- replay.

The `incompatible-version/` fixtures are therefore useful inputs for these kinds
of experiments.

---

# Project Principles

## 1. Determinism

Fixtures must be reproducible.

When the same controlled parameters are used, the repository should be able to
determine whether the generated artifact matches the expected artifact.

---

## 2. Expected Results Exist Before Execution

The expected outcome should exist independently of the system being tested.

Therefore:

    expected_result != observed_result

`expected_result` represents the test oracle.

`observed_result` represents what actually happened during execution.

---

## 3. Verifiable Integrity

Published fixtures must be identifiable and protected against silent changes.

Checksums and automated validation provide the integrity layer.

---

## 4. Versioning

Contracts and releases must preserve the evolution of the test set.

Consumers should be able to reproduce an experiment against a specific released
version rather than an uncontrolled moving target.

---

## 5. Privacy by Design

This project should not require personal or confidential data to demonstrate
data-quality scenarios.

Public datasets should follow the privacy and licensing requirements documented
by the repository.

---

## 6. Clear Repository Boundary

This repository produces test data.

It does not implement the system under test.

The intended separation is:

    fixtures
       ↓
    reusable test inputs

rather than:

    fixtures
       +
    cloud platform
       +
    Terraform
       +
    runtime infrastructure

Keeping that boundary clear allows the same fixture package to be reused across
multiple experiments without coupling the datasets to a specific platform
implementation.

---

# Definition of Done

A fixture set should only be considered complete when, at minimum:

- it can be regenerated deterministically;
- it is associated with the appropriate versioned contract;
- every fixture has a documented expected result;
- artifact integrity can be verified;
- the relevant automated tests pass;
- the fixture's purpose is sufficiently documented;
- applicable privacy requirements are documented;
- applicable licensing or provenance requirements are documented.

The existence of code or generated data alone is not sufficient evidence of
validation.

---

# Final Outcome

The product of this repository is not simply a directory containing sample data.

The product is a reusable combination of:

    reproducible inputs
           +
    versioned contracts
           +
    expected outcomes
           +
    integrity metadata
           +
    automated verification

Together, these components make it possible to ask a reproducible question:

> Given this dataset and this contract, what decision should a trusted data
> admission system make?