# How to add a new parser to an existing parser project

## Parser organization in NOMAD

The NOMAD parsers can be found within the NOMAD gitlab project repository () under
`dependencies/parsers/`. The parsers are organized into the following individual projects
(`dependencies/parsers/<parser_project_name>`) with their own corresponding repositories:

* atomistic - Parsers for output from classical molecular simulations, e.g., from Gromacs, Lammps, etc.
* database - Parsers for various databases, e.g., OpenKim
* eelsdb - Parser for the EELS database (https://eelsdb.eu/).
* electronic - Parsers for output from electronic structure calculations, e.g., from Vasp, Fhiaims, etc.
* nexus - Parsers for combining various instrument output formats and electronic lab notebooks.

Within each project folder you will find a `test/` directory, containing the parser tests
(to be discussed later), and also a directory containing the parsers' source code,
`<parser_project_name>parsers` or `<parser_project_name>parser` dependent on how many
parsers are contained within the project. In the case of multiple parsers, the files discussed
below will then be contained within the corresponding subdirectory for each individual parser.
For example, the VASP parser files are found in `dependencies/parsers/electronic/electronicparsers/vasp/`

## Adding a new parser to an existing parser project

The easiest way to add a parser is within an existing parser project.
In this case, after _cloning and installing a development version of NOMAD_, you will need
to create new branches within *both* the NOMAD project and *also* within the corresponding
parser project. Ideally, this should be done following the _best practices for NOMAD development_.
Here, we briefly outline the procedure:

Create a new issue within the NOMAD project at [NOMAD gitlab](https://gitlab.mpcdf.mpg.de/nomad-lab/nomad-FAIR.git).
On the page of the new issue, in the top right, click the arrow next to the `Create merge request`
button and select `Create branch`. The branch name should be automatically generated with the
corresponding issue number and the title of the issue (copy the branch name to the clipboard for use below),
and the default source branch should be `develop`.
Click the `Create branch` button.

Now, run the following commands in your local NOMAD directory:

`git fetch --all` - sync with remote

`git checkout origin/<new_branch_name> -b <new_branch_name>` - checkout the new branch and create a local copy of the branch

Unless you *just* installed the NOMAD development version, you should rerun `./scripts/setup_dev_env.sh`
within the NOMAD directory to reinstall with the newest development branch.

Now we need to repeat this process for the parser project that we plan to extend.
All parser projects are available at [nomad-coe](https://github.com/nomad-coe). For example, the
atomistic parser project is found at [atomistic-parsers](https://github.com/nomad-coe/atomistic-parsers).

Similar to above, create a new issue at the relevant parser project page. Using the identical
issue title as you did for the NOMAD project above is ideal for clarity.
On the page of the new issue, in the right side-bar under the subsection `Development`, click
`Create a branch`. As above, the branch name should be automatically generated with the
corresponding issue number and the title of the issue, and the default source branch should be `develop`
(which can be seen by clicking `change source branch`).
Under `What's next`, the default option should be `Checkout locally`, which is what we want in this case.
Click `Create branch`, and then copy the provided commands to the clipboard, and run them within the parser
project folder within your local NOMAD repo, i.e., `dependencies/parsers/<parser_project_name>`.