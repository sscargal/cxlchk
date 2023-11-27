# CXLChk: A System Configuration Validation and Optimization Checker for Compute Express Link (CXL) Environments

## Project Overview
This project provides a command-line tool to analyze Linux server configurations that have Compute Express Link (CXL) devices. It checks for configuration issues, hardware failures, and provides optimization recommendations. The tool consists of two main components: a Data Collector and an Analyzer. The Data Collector collects data from a server. The Analyzer processes the collected data using rules. The tool can process data both online (directly from a live server) and offline (from collected data).

## Installation
### Prerequisites
cxlchk has the following requirements:
- Python 3.8 or higher
- A Linux server with Compute Express Link (CXL) devices
  - Linux Mainline Kernel 5.19 or higher is recommended
  - Linux Distributions such as Ubuntu 22.04.01, Fedora 36, or later are recommended

### Installing
Clone the repository:
```bash
git clone https://github.com/sscargal/cxlchk
```
Navigate to the project directory:
```bash
cd cxlchk
```
Install the required dependencies:
```bash
pip install -r requirements.txt
```

## Usage

### Basic Usage

Data Collection Mode:

This command will execute the collector on a live server and save the data for offline analysis.
```bash
cxlchk -C
```

Data Analysis Mode:

Use this command to execute the analyzer on a previously collected dataset.
```bash
cxlchk -A [path/to/dataset.tar.gz]
```

List Rule Groups:

Lists all available rule groups.
```bash
cxlchk -l
```

### Advanced Usage
Including Specific Rule Groups:

Analyze the dataset using only specific rule groups (e.g.: group1 and group2).
```bash
cxlchk -A [path/to/dataset.tar.gz] -i group1 group2
```

Excluding Specific Rule Groups:

Analyze the dataset excluding specific rule groups (e.g.: group1 and group2).
```bash
cxlchk -A [path/to/dataset.tar.gz] -e group1 group2
```

Verbose Output:

Run the collector with verbose output. Use -vv or -vvv for more detailed output.
```bash
cxlchk -C -v
```

### Examples

Collect data with verbose output:
```bash
cxlchk -C -vv
```

Analyze a specific dataset, including only rules from network and security groups:
```bash
cxlchk -A /path/to/data.tar.gz -i network security
```

Analyze a dataset excluding the hardware rule group:
```bash
cxlchk -A /path/to/data.tar.gz -e hardware
```

List all available rule groups:

```bash
cxlchk -l
```

## Contributing
Contributions to this project are welcome. Please make sure your code follows the project's coding standards and includes the appropriate tests described in CONTRIBUTING.md.
