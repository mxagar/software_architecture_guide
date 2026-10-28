# Software Architecture Guide

Personal notes on Software Architecture.

I have organized the notes in the following sections, each of them in a separate folder:

- [`01_Design_Patterns`](./01_Design_Patterns/)
- [`02_SW_Architecture_Patterns`](./02_SW_Architecture_Patterns/)
- [`03_TDD_and_BDD`](./03_TDD_and_BDD/)
- [`04_Fundamentals_SW_Architecture_Book`](./04_Fundamentals_SW_Architecture_Book/)

See also some other related guides of mine:

- [Backend Django Guide](https://github.com/mxagar/backend_django_guide)
- [Web-App Deployment Guide](https://github.com/mxagar/webapp_deployment_guide)
- [DevOps and Software Engineering Guide](https://github.com/mxagar/software_devops_ibm)

## Cloning with submodules

The Design Patterns notes are included as a Git submodule in
[`01_Design_Patterns/design_patterns_notes`](./01_Design_Patterns/design_patterns_notes/).

To clone this guide and initialize its submodules in one step:

```bash
git clone --recurse-submodules https://github.com/mxagar/software_architecture_guide.git
```

If you have already cloned the guide, run the following from the repository root
to download and initialize the submodules at their recorded commits:

```bash
git submodule update --init --recursive
```

## Dependencies

Each folder should contain its own dependencies and instructions for setting them up, if applicable. I usually use `uv`, and/or `pip-tools` in the past.

## Authorship

These notes have been collected after following courses and reading relevant literature on software architecture. All due credit goes to the original authors and sources; check the respective references in each section/folder for more details.

Mikel Sagardia, 2026.  
No guarantees.
