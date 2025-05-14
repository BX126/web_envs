# Web Environments - Quick Start Guide

This guide will help you set up and host all required websites quickly and efficiently.

## Prerequisites
- Ensure you have `bash` and all necessary permissions to execute scripts in this directory.
- **Node.js** (version 18.x or higher recommended). You can check your version with `node -v`.
  - If you do not have Node.js installed, visit [nodejs.org](https://nodejs.org/) and follow the installation instructions for your operating system.

## Setup Instructions

### 1. Install Required Resources and Packages
Run the following command to download all necessary resources and install required packages. *This step only needs to be performed once*:

```bash
bash build.sh
```

Change the HOME_URL in .env file to your home url.

### 2. Host Websites Enviroment
After the initial setup, you can host all websites by running:

```bash
bash host.sh
```
