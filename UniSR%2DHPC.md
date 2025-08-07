# UniSR Cluster Guide

Welcome to the UniSR High Performance Computing (HPC) cluster documentation! This guide provides comprehensive information for using our (tiny 👼🏻) GPU cluster for machine learning research and computational tasks.

  

## Overview

  

The UniSR GPU cluster consists of two workstations, each equipped with two NVIDIA GPUs, designed to support deep learning training, data processing, and computational research. This documentation has been migrated to a DevOps wiki format to provide better organization and accessibility.

  

# Quick Start

  

1. **Connect to VPN** - Ensure you're connected to the provided VPN

2. **SSH Access** - Use `ssh surname.name@hsr.it@10.64.79.72` to connect

3. **Check Resource Availability** - Review the shared spreadsheet for resource booking

4. **Submit Jobs** - Use SLURM workload manager to run your computational tasks

 ---



# Table of Contents

[[_TOSP_]]
  

## Getting Started

- **[Login Guide](/UniSR%2DHPC/Login-instructions)** - SSH configuration and connection setup

- **[Resource Management](/UniSR%2DHPC/Spreadsheet)** - Shared spreadsheet for booking cluster resources

  

## Core Technologies

- **[SLURM Workload Manager](/UniSR%2DHPC/SLURM-%2D-Cheatsheets)** - Job scheduling, partitions, and resource allocation

- **[Apptainer Containers](/UniSR%2DHPC/Apptainer-%2D-Cheatsheet)** - Containerization for reproducible environments

  

## Practical Usage
- **[Useful tools](/UniSR%2DHPC/Useful-tools)** -Find useful tools to make your life easier on the cluster
- **[Shell Aliases](/UniSR%2DHPC/Aliases)** - Useful command shortcuts and productivity tips

  
---

# Key Features

## Hardware Resources

- **2 nodes**: Each with dual NVIDIA GPUs

- HDD storage for code and datasets

- High-speed SSD storage for active training data

- Parallel SSD storage for maximum performance

  

## Software Environment

- **SLURM**: Professional workload management

- **Apptainer**: Container runtime for reproducible environments

- **Pre-built Images**: Ready-to-use containers with popular ML frameworks

- **Jupyter Support**: Interactive development environment

  

## Resource Management

- **Shared Booking System**: Informal resource coordination via spreadsheet

- **Multiple Partitions**: Separate queues for interactive work vs. training jobs

- **Flexible Storage**: Multiple storage options optimized for different use cases


---

# Getting Help

  

This documentation is organized to help you:

  

1. **Get Connected**: Start with the login guide to establish access

2. **Understand Resources**: Learn about SLURM partitions and storage options

3. **Run Workloads**: Use Apptainer containers for your computational tasks

4. **Coordinate Usage**: Book resources through our shared spreadsheet system

  

For detailed technical information, command examples, and troubleshooting, refer to the specific guide sections linked above.

  

# Best Practices

  

- Use the **interactive partition** for development and debugging (12-hour limit)

- Use the **cuda partition** for GPU-intensive training (5-day limit)

- Book resources in advance using the shared spreadsheet

- Store datasets on appropriate storage tiers based on access patterns

- Prefer batch jobs over interactive sessions for long-running training

  

---

  

*This guide is maintained by the UniSR team. For technical support or questions, please reach out through the appropriate channels.*