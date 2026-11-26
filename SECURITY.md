# Security policy

## Supported versions

The latest release of each tool. A fix comes in a new release: older releases are not patched.

## Reporting a vulnerability

Please do not open a public issue. Report it privately, through the **Security** tab of the
repository concerned (**Report a vulnerability**), or by email to the maintainer, Jacques Serizay, at
<jacques.serizay@pasteur.fr>. Say what it affects, how to reproduce it, and which version (`<tool> --version`).

Once it is fixed and released, the advisory is published, crediting you unless you ask otherwise.

## Scope

These tools run on your own machine, or your cluster, and read the files you give them. What matters
most:

- an input file (reads, an assembly, pairs, an alignment, a contact map, a configuration file) that
  makes a tool run code, or read or write outside the paths it was given;
- a release, on PyPI, bioconda or a container registry, that was not built from the tagged source;
- a workflow of a repository that gives a pull request from a fork its secrets, or write access.

A crash or a wrong result on a malformed file is an ordinary bug: open an issue.
