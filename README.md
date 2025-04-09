# Golden Helix Workflow Registry

This repository serves as a catalog for workflows, tasks, and applications that can be imported into the Golden Helix server and executed on the workflow and application runtime engine.

## Overview

This registry contains a collection of pre-configured workflows and applications designed to work with Golden Helix's genomic analysis platform. These components can be imported into your Golden Helix server to extend its functionality and automate various genomic analysis tasks.

## About Golden Helix

Golden Helix provides powerful genomic analysis software solutions, with VarSeq being their flagship product for variant analysis and interpretation. For more information about Golden Helix and their products, visit:

- [Golden Helix Website](https://www.goldenhelix.com/)
- [VarSeq Product Page](https://www.goldenhelix.com/products/VarSeq/)

## Repository Structure

The repository is organized around a central `workflow-repository.json` file that contains metadata for workflows available through the Golden Helix VSWarehouse server. This file is accessed when adding workflows via Workflows > Actions > Add Workflow from Repository in the VSWarehouse interface.

The repository contains:

- `workflow-repository.json` - Central manifest file containing metadata about available workflows including:
  - Name and description
  - Git repository URL
  - Workflow tags and categories
  - Last updated timestamp
  - Organization and contact information

- Individual workflow repositories referenced in the manifest, each containing:
  - Workflow definition files
  - Documentation
  - Example configurations
  - Test datasets where applicable
