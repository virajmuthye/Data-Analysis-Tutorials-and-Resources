# Complete Guide to Linux Command Line for Bioinformatics and Data Science

**Author:** Viraj Rajendra Muthye  
**Email:** viraj.muthye@gmail.com
**Date:** `r Sys.Date()`  

---

## Table of Contents

1. [Introduction](#introduction)
2. [Linux File System Basics](#linux-file-system-basics)
3. [Essential Navigation Commands](#essential-navigation-commands)
4. [File and Directory Operations](#file-and-directory-operations)
5. [File Content Management](#file-content-management)
6. [Advanced File Operations](#advanced-file-operations)
7. [Working with SLURM Job Scheduler](#working-with-slurm-job-scheduler)
8. [Bioinformatics-Specific Commands](#bioinformatics-specific-commands)
9. [Best Practices and Tips](#best-practices-and-tips)
10. [Practice Exercises](#practice-exercises)
11. [Resources and References](#resources-and-references)

---

## Introduction

This tutorial provides a comprehensive introduction to Linux command line operations, with a focus on applications in bioinformatics and data science. Whether you're working on high-performance computing clusters like ARC (Advanced Research Computing) or your local Linux machine, these commands form the foundation of efficient computational work.

### Why Learn Linux Command Line?

- **Efficiency**: Command line operations are often faster than GUI alternatives
- **Automation**: Essential for scripting and batch processing
- **Remote Access**: Most servers and clusters use Linux without GUI
- **Bioinformatics**: Many bioinformatics tools are command-line based
- **Data Science**: Powerful tools for data manipulation and analysis

### Prerequisites

- Basic understanding of file systems and directories
- Access to a Linux terminal (local machine, server, or online simulator)
- No prior Linux experience required

---

## Linux File System Basics

### Understanding the File System

In Linux, the file system is organized as a hierarchical tree structure:

- **Directory**: What Windows users call a "folder"
- **Path**: The location of a file or directory in the file system
- **Root Directory**: The top-level directory, represented by `/`

### File System Structure Example

```
/work/
├── wasmuth_lab/
│   ├── viraj/
│   │   ├── mimicry/
│   │   ├── projects/
│   │   └── data/
│   └── shared/
└── other_users/
```

### Understanding Paths

#### Absolute Paths
- Start from the root directory (`/`)
- Always begin with a forward slash
- Work from anywhere in the system

**Example:** `/work/wasmuth_lab/viraj/mimicry`

#### Relative Paths
- Start from your current location
- Do not begin with a forward slash
- Depend on your current working directory

**Example:** If you're in `/work/wasmuth_lab/`, you can reach the mimicry folder with `viraj/mimicry`

### Special Path Symbols

| Symbol | Meaning |
|--------|---------|
| `/` | Root directory |
| `~` | Home directory |
| `.` | Current directory |
| `..` | Parent directory |
| `-` | Previous directory |

---

## Essential Navigation Commands

### 1. Finding Your Location: `pwd`

The `pwd` (print working directory) command shows your current location in the file system.

```bash
pwd
# Output example: /home/viraj/projects
```

### 2. Changing Directories: `cd`

The `cd` (change directory) command is your primary navigation tool.

```bash
# Navigate to home directory
cd

# Navigate to a specific directory
cd /work/wasmuth_lab/viraj

# Move up one level
cd ..

# Move up two levels
cd ../..

# Go to previous directory
cd -

# Navigate using relative path
cd projects/data

# Navigate using absolute path
cd /work/wasmuth_lab/viraj/mimicry
```

### 3. Listing Directory Contents: `ls`

The `ls` command displays the contents of directories.

```bash
# Basic listing
ls

# Detailed listing with permissions, dates, sizes
ls -l

# List all files (including hidden files starting with .)
ls -a

# List in human-readable format
ls -lh

# List contents of a specific directory
ls /work/wasmuth_lab/

# Combine options
ls -lah

# List files with specific extension
ls *.fasta
```

**Understanding `ls -l` Output:**
```
-rw-r--r-- 1 viraj users 1.2K Oct 15 14:30 sequences.fasta
drwxr-xr-x 2 viraj users 4.0K Oct 15 15:45 analysis/
```

| Column | Meaning |
|--------|---------|
| `-rw-r--r--` | File permissions |
| `1` | Number of links |
| `viraj` | Owner |
| `users` | Group |
| `1.2K` | File size |
| `Oct 15 14:30` | Last modified date |
| `sequences.fasta` | File name |

---

## File and Directory Operations

### 1. Creating Directories: `mkdir`

```bash
# Create a single directory
mkdir analysis

# Create multiple directories
mkdir data results scripts

# Create nested directories
mkdir -p projects/2023/analysis/output

# Create directory with specific permissions
mkdir -m 755 public_data
```

### 2. Copying Files and Directories: `cp`

```bash
# Copy a file
cp source.txt destination.txt

# Copy file to another directory
cp data.csv /work/analysis/

# Copy multiple files
cp file1.txt file2.txt file3.txt destination_directory/

# Copy all files with specific extension
cp *.fasta sequences/

# Copy directory and all contents (recursive)
cp -r source_directory/ destination_directory/

# Copy with interactive confirmation
cp -i important_file.txt backup/

# Preserve file attributes and timestamps
cp -p original.txt copy.txt

# Copy and show progress for large files
cp -v large_file.dat backup/
```

### 3. Moving and Renaming: `mv`

```bash
# Rename a file
mv old_name.txt new_name.txt

# Move file to another directory
mv data.csv analysis/

# Move multiple files
mv *.log logs/

# Move and rename simultaneously
mv temp_results.txt analysis/final_results.txt

# Move directory
mv old_project_name/ new_project_name/

# Move with interactive confirmation
mv -i important_file.txt archive/
```

### 4. Removing Files and Directories: `rm` and `rmdir`

> ⚠️ **WARNING**: The `rm` command permanently deletes files. There is no "Trash" or "Recycle Bin" in Linux!

```bash
# Remove a file
rm unwanted_file.txt

# Remove multiple files
rm file1.txt file2.txt file3.txt

# Remove all files with specific extension
rm *.tmp

# Remove with confirmation prompt
rm -i important_file.txt

# Remove directory and all contents (DANGEROUS!)
rm -r directory_name/

# Force removal (use with extreme caution!)
rm -rf stubborn_directory/

# Remove empty directory
rmdir empty_directory/
```

**Safety Tips:**
- Always double-check your `rm` commands
- Use `ls` to verify what you're about to delete
- Consider using `rm -i` for interactive confirmation
- Make backups of important files
- The infamous "rm and Toy Story" incident: Pixar accidentally deleted most of Toy Story 2 with an `rm -rf` command!

---

## File Content Management

### 1. Viewing File Contents: `cat`, `less`, `more`

```bash
# Display entire file content
cat sequences.fasta

# Display multiple files
cat file1.txt file2.txt

# Display with line numbers
cat -n sequences.fasta

# View large files page by page (preferred for large files)
less large_file.txt
# Use arrow keys, Page Up/Down, 'q' to quit

# Alternative pager
more large_file.txt
```

### 2. Viewing Parts of Files: `head` and `tail`

```bash
# Show first 10 lines (default)
head sequences.fasta

# Show first 20 lines
head -20 sequences.fasta

# Show first 5 lines
head -5 sequences.fasta

# Show last 10 lines (default)
tail log_file.txt

# Show last 20 lines
tail -20 log_file.txt

# Follow a growing file (useful for logs)
tail -f analysis.log

# Show specific line range using combination
head -50 file.txt | tail -10  # Lines 41-50
```

### 3. Concatenating Files: `cat` with Redirection

```bash
# Combine multiple files into one
cat file1.fasta file2.fasta > combined.fasta

# Append to existing file
cat new_sequences.fasta >> all_sequences.fasta

# Combine all FASTA files
cat *.fasta > all_sequences.fasta

# Create a new file with content
cat > new_file.txt
# Type content, then Ctrl+D to save and exit
```

### 4. Searching in Files: `grep`

```bash
# Search for a pattern
grep "pattern" file.txt

# Case-insensitive search
grep -i "Pattern" file.txt

# Search recursively in directories
grep -r "function" scripts/

# Count matches
grep -c ">" sequences.fasta

# Show line numbers
grep -n "error" log_file.txt

# Search for lines NOT containing pattern
grep -v "comment" data.txt

# Use regular expressions
grep "^>" sequences.fasta  # Lines starting with >
```

---

## Advanced File Operations

### 1. File Permissions and Ownership

```bash
# View detailed file permissions
ls -l

# Change file permissions
chmod 755 script.sh        # rwxr-xr-x
chmod u+x script.sh        # Add execute permission for user
chmod g-w file.txt         # Remove write permission for group
chmod o+r data.csv         # Add read permission for others

# Change ownership
chown viraj:users file.txt
chown viraj file.txt       # Change owner only
chgrp users file.txt       # Change group only
```

### 2. File Compression and Archives

```bash
# Create tar archive
tar -cvf archive.tar directory/

# Create compressed tar archive
tar -czvf archive.tar.gz directory/

# Extract tar archive
tar -xvf archive.tar
tar -xzvf archive.tar.gz

# Create zip archive
zip -r archive.zip directory/

# Extract zip archive
unzip archive.zip

# View archive contents without extracting
tar -tvf archive.tar.gz
```

### 3. Finding Files and Directories: `find`

```bash
# Find files by name
find . -name "*.fasta"

# Find files by type
find . -type f -name "*.py"    # Files only
find . -type d -name "data*"   # Directories only

# Find files by size
find . -size +100M             # Files larger than 100MB
find . -size -1K               # Files smaller than 1KB

# Find files by modification time
find . -mtime -7               # Modified in last 7 days
find . -mtime +30              # Modified more than 30 days ago

# Execute command on found files
find . -name "*.tmp" -delete   # Delete all .tmp files
find . -name "*.py" -exec wc -l {} \;  # Count lines in Python files
```

### 4. Text Processing Tools

```bash
# Count words, lines, characters
wc file.txt                    # All counts
wc -l file.txt                 # Line count only
wc -w file.txt                 # Word count only
wc -c file.txt                 # Character count only

# Sort lines
sort file.txt                  # Alphabetical sort
sort -n numbers.txt            # Numerical sort
sort -r file.txt               # Reverse sort
sort -u file.txt               # Sort and remove duplicates

# Remove duplicate lines
uniq file.txt                  # Remove consecutive duplicates
sort file.txt | uniq           # Remove all duplicates

# Cut columns from files
cut -d',' -f1,3 data.csv       # Extract columns 1 and 3 from CSV
cut -c1-10 file.txt            # Extract characters 1-10

# Replace text
sed 's/old/new/g' file.txt     # Replace all occurrences
sed 's/old/new/' file.txt      # Replace first occurrence per line
```

---

## Working with SLURM Job Scheduler

SLURM (Simple Linux Utility for Resource Management) is commonly used on high-performance computing clusters.

### 1. Basic SLURM Commands

```bash
# Submit a job
sbatch job_script.slurm

# Check job queue for your jobs
squeue -u your_username

# Check detailed job information
scontrol show job job_id

# Cancel a job
scancel job_id

# Cancel all your jobs
scancel -u your_username

# Check cluster information
sinfo

# Check your account usage
sacct
```

### 2. Sample SLURM Job Script

Create a file named `analysis.slurm`:

```bash
#!/bin/bash
#SBATCH --job-name=my_analysis
#SBATCH --output=analysis_%j.out
#SBATCH --error=analysis_%j.err
#SBATCH --time=02:00:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --mem=8GB
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=your.email@university.edu

# Load necessary modules
module load python/3.9

# Change to working directory
cd /work/your_project/

# Run your analysis
python analysis_script.py

# Additional commands
echo "Job completed at $(date)"
```

### 3. Job Monitoring

```bash
# Monitor job progress
tail -f analysis_12345.out

# Check job efficiency after completion
seff job_id

# View job history
sacct -j job_id --format=JobID,JobName,MaxRSS,Elapsed
```

---

## Bioinformatics-Specific Commands

### 1. Working with FASTA Files

```bash
# Count sequences in FASTA file
grep -c ">" sequences.fasta

# Extract sequence headers
grep ">" sequences.fasta

# Extract sequence headers with line numbers
grep -n ">" sequences.fasta

# Get sequence statistics
grep ">" sequences.fasta | wc -l  # Number of sequences

# Extract sequences by pattern
grep -A 1 "species_name" sequences.fasta

# Convert multiline FASTA to single-line
awk '/^>/ {if (seq) print seq; print; seq=""; next} {seq = seq $0} END {print seq}' input.fasta > output.fasta
```

### 2. Working with Data Files

```bash
# Quick data exploration
head -5 data.csv                    # First few rows
tail -5 data.csv                    # Last few rows
wc -l data.csv                      # Number of rows

# Column operations
cut -d',' -f1 data.csv | head       # First column
cut -d',' -f1,3,5 data.csv          # Multiple columns

# Search for patterns in data
grep "pattern" data.txt
grep -i "case_insensitive" data.txt
grep -v "exclude_this" data.txt

# Simple statistics
sort -n values.txt | uniq -c        # Frequency count
sort -n values.txt | head -1        # Minimum value
sort -n values.txt | tail -1        # Maximum value
```

### 3. File Format Conversions

```bash
# Convert line endings (Windows to Unix)
dos2unix file.txt

# Convert line endings (Unix to Windows)
unix2dos file.txt

# Remove blank lines
grep -v "^$" file.txt > clean_file.txt

# Add line numbers
nl file.txt

# Remove line numbers
sed 's/^[[:space:]]*[0-9]*[[:space:]]*//' file.txt
```

---

## Best Practices and Tips

### 1. File Naming Conventions

```bash
# Good practices:
my_analysis_2023.txt
results_v1.2.csv
protein_sequences.fasta

# Avoid:
"my file with spaces.txt"    # Spaces cause problems
file.txt.txt                 # Confusing extensions
MyFile.TXT                   # Inconsistent capitalization
```

### 2. Safety and Backup Strategies

```bash
# Always backup important files
cp important_file.txt important_file.txt.backup
cp important_file.txt important_file_$(date +%Y%m%d).txt

# Use version control for scripts
git init
git add script.py
git commit -m "Initial version"

# Test commands safely
ls file*                     # Check what will be affected
echo rm file*               # See what would be deleted without doing it
```

### 3. Efficient Command Usage

```bash
# Use tab completion
cd /work/was[TAB]           # Auto-complete directory names

# Use command history
history                     # View command history
!123                        # Run command number 123 from history
!!                          # Run last command
!grep                       # Run last command starting with "grep"

# Use aliases for common commands
alias ll='ls -lah'
alias ..='cd ..'
alias la='ls -la'

# Chain commands
cd analysis && ls -la       # Run second command only if first succeeds
cd analysis || echo "Directory not found"  # Run second if first fails
cd analysis; ls -la         # Always run both commands
```

### 4. Working with Large Files

```bash
# Process files without loading entirely into memory
head -1000000 large_file.txt > sample.txt  # Create sample
split -l 10000 large_file.txt chunk_       # Split into chunks
wc -l large_file.txt                       # Count lines efficiently

# Monitor disk usage
du -h directory/                           # Directory size
df -h                                      # Disk space usage
```

---

## Practice Exercises

### Exercise 1: Basic Navigation
```bash
# 1. Find your current location
pwd

# 2. Go to your home directory
cd

# 3. Create a directory called "linux_practice"
mkdir linux_practice

# 4. Navigate to this directory
cd linux_practice

# 5. Create three subdirectories: data, scripts, results
mkdir data scripts results

# 6. List the contents to verify
ls -la
```

### Exercise 2: File Operations
```bash
# 1. Create a sample file
echo "This is a test file" > data/sample.txt

# 2. Copy it to the results directory
cp data/sample.txt results/

# 3. Rename the copy
mv results/sample.txt results/processed_sample.txt

# 4. View the file contents
cat results/processed_sample.txt

# 5. Count the words in the file
wc -w results/processed_sample.txt
```

### Exercise 3: Working with Multiple Files
```bash
# 1. Create several files
echo "Sequence 1: ATCGATCG" > data/seq1.txt
echo "Sequence 2: GCTAGCTA" > data/seq2.txt
echo "Sequence 3: TTAATTAA" > data/seq3.txt

# 2. Combine all sequence files
cat data/seq*.txt > results/all_sequences.txt

# 3. Search for a specific pattern
grep "GC" results/all_sequences.txt

# 4. Count the number of sequences
grep -c "Sequence" results/all_sequences.txt
```

---

## Troubleshooting Common Issues

### Permission Denied Errors
```bash
# Problem: Permission denied when trying to execute a script
# Solution: Make the script executable
chmod +x script.sh

# Problem: Cannot write to a directory
# Solution: Check permissions and ownership
ls -ld directory_name/
```

### File Not Found Errors
```bash
# Problem: File not found
# Solution: Check if file exists and verify path
ls -la filename
pwd  # Confirm current directory
```

### Command Not Found Errors
```bash
# Problem: Command not found
# Solution: Check if command is installed or load appropriate module
which command_name
module avail  # On cluster systems
```

---

## Resources and References

### Online Practice Platforms
- [Tutorial Point Bash Online](https://www.tutorialspoint.com/execute_bash_online.php)
- [JSLinux](https://bellard.org/jslinux/) - Linux emulator in browser
- [Katacoda Linux Scenarios](https://www.katacoda.com/courses/linux)

### Comprehensive Tutorials
- [Unix Tutorial for Beginners](http://www.ee.surrey.ac.uk/Teaching/Unix/)
- [Iowa State HPC Unix Introduction](https://www.hpc.iastate.edu/guides/unix-introduction/unix-tutorial-1)
- [Linux Command Line Basics](https://ubuntu.com/tutorials/command-line-for-beginners)

### Command References
- [Linux Command Library](https://linuxcommandlibrary.com/)
- [SS64 Linux Commands](https://ss64.com/bash/)
- [ExplainShell](https://explainshell.com/) - Explains shell commands

### SLURM Resources
- [SLURM Documentation](https://slurm.schedmd.com/documentation.html)
- [SLURM Job Script Generator](https://www.hpc.iastate.edu/guides/condo-2017/slurm-job-script-generator-for-condo)

### Books
- "The Linux Command Line" by William Shotts
- "Unix and Linux System Administration Handbook" by Evi Nemeth
- "Learning the bash Shell" by Cameron Newham

---

## Quick Reference Card

### Essential Commands
```bash
pwd              # Print working directory
ls -la           # List files with details
cd directory     # Change directory
mkdir name       # Create directory
cp src dest      # Copy files
mv src dest      # Move/rename files
rm file          # Delete file
cat file         # View file content
grep pattern     # Search in files
chmod +x file    # Make executable
```

### File Permissions
```
r = read (4)     w = write (2)     x = execute (1)
user  group  other
rwx   rwx    rwx
421   421    421
```

### Common Shortcuts
```bash
~        # Home directory
.        # Current directory
..       # Parent directory
-        # Previous directory
Ctrl+C   # Cancel current command
Ctrl+Z   # Suspend current command
Ctrl+D   # End of file/logout
```

---

## Conclusion

This tutorial provides a solid foundation for working with Linux command line in bioinformatics and data science contexts. Practice these commands regularly, and gradually incorporate more advanced techniques as you become comfortable with the basics.

Remember:
- Start with simple commands and build complexity gradually
- Always double-check destructive operations (especially `rm`)
- Use `man command_name` to get detailed help for any command
- Practice in a safe environment before working with important data
- Don't hesitate to ask for help in online communities

### Session Information

```{r session-info, echo=FALSE}
# This section would typically show R session info
# For a Linux tutorial, we'll show system information instead
```

For questions, suggestions, or contributions, please contact:
**Viraj Rajendra Muthye** - viraj.muthye@gmail.com

---

*Happy computing! 🐧*
