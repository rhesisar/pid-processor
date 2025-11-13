# P&ID SVG Processor - case study (code private)

This document describes a private Go project that processes Piping and Instrumentation Diagram (P&ID) drawings exported as SVG from Microsoft Visio. It parses large, inconsistent SVGs, normalizes them, detects sub-areas and clickable assets, injects simple click handlers, and writes a stable tree of output SVG files for a downstream UI.

## At a glance

- Inputs: Microsoft Visio SVG exports with heterogeneous structure and transform stacks
- Outputs: a tree of rewritten SVGs (root plus sliced sub-areas), each with consistent viewBox and minimal interactivity
- Core idea: color based detection of sub-PID areas and clickable assets, followed by normalization and write out
- Why it matters: turns static drawings into navigable artifacts a web UI can use immediately
- Role: owned parsing and normalization, designed the small CLI level usage and outputs, wrote tests, coordinated milestones in a 4 person team
- Tech: Go on Linux and macOS, deterministic processing, unit tests and a small fuzz harness during development

## Context and constraints

- Typical file size: 5KB to 1MB
- Typical node count: about 100 to 100k SVG elements
- Sources: multiple Visio templates and export settings leading to different tag usage, attributes, and nested transforms
- Requirements: memory safety, predictable runtime, stable outputs across runs and machines, CLI first for batch processing

## What it produces

1) Rewritten root SVG and child SVGs  
   A stable naming scheme is used for the output files, making it easy for a UI to resolve and display sub diagrams.

2) Clickable assets in place  
   Target shapes are marked clickable with a CSS class and a small onclick handler string. The original `<desc>` is preserved so a UI can display metadata.

3) Asset location mapping (conceptual)  
   During processing, a mapping from asset identifiers or labels to their location in the sub diagram tree is built and used to decide where to inject interactivity.

## How it works

Pipeline overview:

- Streaming parse  
  Read the SVG and build only the minimal structure needed to bound memory usage. Avoid full in memory DOMs when possible.

- Normalization  
  Unify coordinate frames and transforms, canonicalize ids and classes, and map Visio based symbols and text to domain entities.

- Detection and indexing  
  Identify sub areas and clickable assets using a small set of configured colors, then prepare lightweight indexes by id, tokenized text, symbol category, and region.

- Output  
  Write a compact set of SVGs for the root and sub areas, with consistent viewBox and small onclick strings injected for interactive elements.

Detection strategy:

- Sub areas  
  A small set of fill colors is treated as markers for nested P&ID regions. Colors are processed in order from outermost to innermost to resolve overlaps.

- Clickable assets  
  Shapes with a configured color or class are treated as clickable assets. Their `<desc>` content is preserved so the UI can show a tooltip or open a details view.

- Normalization details  
  Elements outside the viewBox are dropped. Common nested group transforms are flattened where needed. Basic path parsing covers the common primitives necessary to place and bound shapes reliably.

- Interactivity  
  Clickable elements receive a `clickable` class and an `onclick` attribute provided by configuration. The strings are kept small and safe.

## Representative results on a local Apple M1

- Median processing time per Visio diagram: about 15.6 ms
- Throughput on typical files: about 100k nodes per second
- Peak memory: under 7 MB during steady state

These vary with export settings and diagram complexity. Processing stayed within budget on the internal dataset.

## Reliability and testing

- Deterministic outputs for a given input and config
- Defensive parsing with clear error messages that include element ids or positions
- Unit tests across transforms, areas, assets, and write out paths
- A small fuzz harness on attribute and transform handlers to uncover edge cases during development

## Limits and notes

- Detection relies on color conventions. If Visio templates or styles change, configuration updates are required.
- Only a subset of SVG features is handled. Deep transform stacks, exotic filters, or embedded fonts may require small normalization tweaks.
- Text extraction depends on the Visio export. Rotated or embedded fonts can be lossy in some exporters.

## What I would add next

- Parallel batch processing across many diagrams
- Optional JSON index emission for external search if needed
- Incremental reprocessing keyed by content hashing

## Team and role

- Team: 4 people
- Scope: parser and normalization design, output scheme and interactivity injection, tests and fuzz inputs, milestone demos and notes
- Collaboration: daily syncs, short written design notes

## Code privacy and related public work

This repository is a case study. Production code and drawings are private due to company IP.

## Contact

robin.holden.rh@gmail.com