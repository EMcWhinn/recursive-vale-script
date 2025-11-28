# Recursive Vale Check Script

This script runs the Vale linter on a specified `master.adoc` file and recursively checks all included modules and snippets, ensuring full style coverage of the documentation source tree.

## Prerequisites

- The [Vale command-line tool](https://vale.sh/docs/install) is installed and accessible in your system's `$PATH`
- You have set up the necessary [Vale configuration](https://github.com/jhradilek/asciidoctor-dita-vale?tab=readme-ov-file#asciidocdita) (e.g., `.vale.ini`)
- The system has the standard Bash shell and core utilities (`grep`, `awk`, `dirname`)

## Installation

### Method 1: Clone the Repository (Recommended)

Clone this repository to get the latest version and easily pull updates:

```bash
# Clone the repository
git clone https://github.com/EMcWhinn/recursive-vale-script.git ~/Scripts/recursive-vale-check-repo

# Create a symbolic link to the script
ln -s ~/Scripts/recursive-vale-check-repo/recursive-vale-check ~/Scripts/recursive-vale-check

# Make the script executable
chmod +x ~/Scripts/recursive-vale-check-repo/recursive-vale-check
```

To update the script in the future, simply run:

```bash
cd ~/Scripts/recursive-vale-check-repo && git pull
```

### Method 2: Manual Installation (Alternative)

If you prefer not to clone the repository, you can manually create the script:

#### 1. Create the Script File

Create the script file in a dedicated location, such as `~/Scripts`:

```bash
mkdir -p ~/Scripts
touch ~/Scripts/recursive-vale-check
```

#### 2. Add the Script Content

Open the file and add the following content:

```bash
#!/bin/bash

# Function to find include files in a source file and run vale on them.
check_includes() {
    local source_file="$1"
    local source_dir="$(dirname "$source_file")"

    # Filters for literal 'include::' paths, excluding attribute-based includes (which contain '{')
    grep '^include::[^\{]*$' "$source_file" | \
    awk -F'::|\\[' '{print $2}' | \
    while read included_path; do
        # Construct the absolute path
        local full_path="$source_dir/$included_path"

        # Check if the file exists and is not an attributes file
        if [[ -f "$full_path" && "$included_path" != *"attributes"* ]]; then
            echo "Linting: $full_path"
            vale "$full_path"
            # Recursively check the included file
            check_includes "$full_path"
        fi
    done
}

# Main execution logic
if [[ -z "$1" ]]; then
    echo "Usage: $0 <path/to/master.adoc>"
    exit 1
fi

echo "--- Checking Master File: $1 ---"
vale "$1"

# Begin the recursive check
check_includes "$1"

echo "--- Recursive Check Complete ---"
```

> **Note:** For larger files, the CLI might truncate the content so you miss the first assemblies and modules. To view only Vale warnings and errors without suggestions, see the [alternative script version](#warnings-and-errors-only-version) below.

#### 3. Make the Script Executable

```bash
chmod +x ~/Scripts/recursive-vale-check
```

## Usage

Run the script from the directory containing your `master.adoc` file:

```bash
~/Scripts/recursive-vale-check master.adoc
```

Or with the full path:

```bash
# Linux
/home/username/Scripts/recursive-vale-check master.adoc

# macOS
/Users/username/Scripts/recursive-vale-check master.adoc
```

## Optional: Create a Shell Alias

To simplify execution, define an alias in your shell configuration file (`~/.bashrc` or `~/.zshrc`):

### 1. Open the Configuration File

```bash
# For bash
vim ~/.bashrc

# For zsh
vim ~/.zshrc
```

### 2. Add the Alias

Add a line that points to the script's full path. For example, using the alias `vale-check`:

```bash
alias vale-check='~/Scripts/recursive-vale-check'
```

### 3. Apply the Changes

```bash
# For bash
source ~/.bashrc

# For zsh
source ~/.zshrc
```

You can now run the check using only `vale-check master.adoc` from your documentation directory.

## Warnings and Errors Only Version

For larger files or to reduce output, use this version that shows only warnings and errors (no suggestions):

```bash
#!/bin/bash

# Function to find include files in a source file and run vale on them.
check_includes() {
    local source_file="$1"
    local source_dir="$(dirname "$source_file")"

    # Filters for literal 'include::' paths, excluding attribute-based includes (which contain '{')
    grep '^include::[^\{]*$' "$source_file" | \
    awk -F'::|\\[' '{print $2}' | \
    while read included_path; do
        # Construct the absolute path
        local full_path="$source_dir/$included_path"

        # Check if the file exists and is not an attributes file
        if [[ -f "$full_path" && "$included_path" != *"attributes"* ]]; then
            echo "Linting: $full_path"
            vale --minAlertLevel=warning "$full_path"
            # Recursively check the included file
            check_includes "$full_path"
        fi
    done
}

# Main execution logic
if [[ -z "$1" ]]; then
    echo "Usage: $0 <path/to/master.adoc>"
    exit 1
fi

echo "--- Checking Master File: $1 ---"
vale --minAlertLevel=warning "$1"

# Begin the recursive check
check_includes "$1"

echo "--- Recursive Check Complete ---"
```

## How It Works

The script performs the following actions:

1. **Initial Check:** Runs `vale` on the specified master file (e.g., `master.adoc`)

2. **Recursive Search:** Calls the `check_includes` function which:
   - Uses `grep '^include::[^\{]*$'` to find all literal includes, filtering out any includes that use AsciiDoc attributes (which contain `{`)
   - Uses `awk` to cleanly extract the file path from the `include::path/to/file.adoc[]` syntax
   - For each found file, runs `vale "$full_path"`
   - Recursively calls `check_includes "$full_path"`, ensuring it descends through all levels of inclusion

3. **Completion:** Prints `--- Recursive Check Complete ---` when finished

## Features

- Recursively lints all included AsciiDoc files
- Automatically skips attribute files
- Filters out attribute-based includes (e.g., `{variable}`)
- Supports both full output and warnings/errors-only modes
- Works with nested include hierarchies

## License

MIT License - See LICENSE file for details

## Contributing

Feel free to open issues or submit pull requests to improve this script.
