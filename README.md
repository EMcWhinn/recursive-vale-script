# Recursive Vale check script

This script runs the Vale linter on a specified `master.adoc` file and recursively checks all included modules and snippets, ensuring full style coverage of the documentation source tree.

### **Prerequisites**

* The [Vale command-line tool](https://vale.sh/docs/install) is installed and accessible in your system's `$PATH`.  
* You have set up the necessary [Vale configuration](https://github.com/jhradilek/asciidoctor-dita-vale?tab=readme-ov-file#asciidocdita) (e.g., `.vale.ini`).  
* The system has the standard Bash shell and core utilities (`grep`, `awk`, `dirname`).

### **Procedure**

1. Create the script file in a dedicated location , such as `<SCRIPTS_DIR>`.

   **Example (Linux/macOS):**

```
mkdir -p ~/Scripts
touch ~/Scripts/recursive-vale-script
# Open the file and paste the following script content:
```

2. **Script content (`recursive-vale-script`):**

```
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

**IMPORTANT**  
For larger files, the CLI might truncate the content so you miss the first assemblies and modules. In order to view just Vale **warnings** and **errors** without suggestions see the [updated script example](#bookmark=id.vgud15gygkra).

3. Grant executable permissions:

```
chmod +x ~/Scripts/recursive-vale-script
```

   

   **NOTE:** 

   Executable permission is required to run the file directly as a command.

4. Run the script from the directory containing your `master.adoc` file, using the full path to the script.

   **Example syntax:**

```
<PATH_TO_SCRIPT>/recursive-vale-script <MASTER_FILE_NAME>
```

   **Example (Linux/macOS):**

```
Linux: /home/emcwhinn/Scripts/recursive-vale-script master.adoc
mac/OS: /Users/sayee/Scripts/recursive-vale-script master.adoc
```

### **Optional: Create a Shell alias**

To simplify execution, define an alias in your shell configuration file (`~/.bashrc`, or `~/.zshrc`)

1. **Open the configuration file:**

```
code ~/.bashrc
```

2. Add a line that points to the script's full path. For example, using the alias `vs` (Vale script):

```
Linux: alias vs='/home/emcwhinn/Scripts/recursive-vale-script master.adoc'
mac/OS: alias vs='/Users/sayee/Scripts/recursive-vale-script master.adoc'

```

3. Apply the new alias without restarting the terminal session:

```
source ~/.bashrc
```

   You can now run the check using only `vs` from your documentation directory.

### **Optional: Script for larger files and viewing Vale errors and warnings only** 

```
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

### **Script logic and output**

The script performs the following actions:

1. Initial Check: Runs `vale` on `master.adoc`.  
2. Recursive Search: Calls `check_includes master.adoc`.  
   * It uses `grep '^include::[^\{]*$'` to find all literal includes, filtering out any includes that use AsciiDoc attributes (which start with an `{`).  
   * It uses `awk` to cleanly extract the file path from the `include::path/to/file.adoc[]` syntax.  
   * For each found file, it runs `vale "$full_path"`.  
   * It then **recursively** calls `check_includes "$full_path"`, ensuring it descends through all levels of inclusion.  
3. **Completion:** Prints `--- Recursive Check Complete ---`.  
     
   
