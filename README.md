# Data Analysis Tutorials and Resources

A collection of hands-on tutorials bridging bioinformatics, data science, and the command-line tools that hold a computational workflow together. Each tutorial is self-contained, with runnable code, tables of reference commands, and a resources section for going deeper.

## Tutorials

| Tutorial | Description |
|---|---|
| [UNIX Basics for Bioinformatics and Data Science](UNIX_basic_commands.md) | Linux command line fundamentals — navigation, file operations, `grep`/`sed`/`awk`, SLURM job scheduling, and FASTA/data-file handling for HPC environments |
| [tmux Tutorial](tmux_tutorial.md) | Terminal multiplexing for keeping long-running jobs alive across SSH sessions — sessions, windows, panes, and detach/reattach workflows |
| [ggplot2 for Data Visualization in R](ggplot2_tutorial.md) | The Grammar of Graphics in R — geoms, aesthetics, faceting, themes, and building publication-ready plots from scratch |
| [Benchmarking Variant Calls with hap.py](happy_benchmarking_tutorial.md) | Evaluating variant calling pipelines against GIAB truth sets — installation, the `vcfeval` engine, output interpretation, stratification, and ROC curve plotting |

The tutorials build on each other where relevant — for example, the hap.py tutorial links back to the UNIX/SLURM tutorial for running benchmarks on a cluster, tmux for keeping those jobs alive, and ggplot2 for visualizing the results.

## Getting Started

Each tutorial is a standalone Markdown file — click any link above to read it directly on GitHub. Tutorials that include R code blocks (like the ggplot2 and hap.py plotting sections) are written to be copy-pasted into an R session or R Markdown document; shell examples can be run directly in a terminal.

## Contributing

Suggestions, corrections, and new tutorial ideas are welcome. Feel free to open an issue or pull request.

## About Me

Hello, my name is **Viraj Rajendra Muthye**, and I am passionate about making data analysis and bioinformatics more approachable and practical for learners at all levels. With a background in biology, bioinformatics, and teaching, I have seen firsthand how powerful data-driven approaches can be in answering scientific questions, but also how intimidating the learning curve can feel.

I am creating tutorials on GitHub to share clear, hands-on examples that bridge theory and practice, helping others build confidence in applying tools and techniques to real-world datasets. My goal is to foster an open, collaborative space where knowledge is accessible and where learners can grow their skills step by step.

## Contact

**Viraj Rajendra Muthye** - viraj.muthye@gmail.com
