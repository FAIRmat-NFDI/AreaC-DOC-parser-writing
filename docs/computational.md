# Parser structure for computation

## Overview of metadata organization for computation

NOMAD stores all processed data in a well defined, structured, and machine readable format, known as the `archive`.
The schema that defines the organization of (meta)data within the archive is known as the `MetaInfo`.
More information can be found in the NOMAD docs: [An Introduction to Schemas and Structured Data in NOMAD](https://nomad-lab.eu/prod/v1/docs/schema/introduction.html).
The following diagram is an overarching visualization of the most important archive sections for computational data:

```
archive
├── run
│    ├── method
│    │      ├── atom_parameters
│    │      ├── dft
│    │      ├── forcefield
│    │      └── ...
│    ├── system
│    │      ├── atoms
│    │      │     ├── positions
│    │      │     ├── lattice_vectors
│    │      │     └── ...
│    │      └── ...
│    └── calculation
│           ├── energy
│           ├── forces
│           └── ...
└── workflow
     ├── method
     ├── inputs
     ├── tasks
     ├── outputs
     └── results
```

The most important section of the archive for computational data is the `run` section, which is
divided into three main subsections: `method`, `system`, and `calculation`. `method` stores
information about the computational model used to perform the calculation. `system` stores
attributes of the atoms involved in the calculation, e.g., atom types, positions, lattice vectors, etc.
`calculation` stores the output of the calculation, e.g., energy, forces, etc.

The `workflow` section of the archive then stores information about the series of tasks performed
to accumulate the (meta)data in the run section. The relevant input parameters for the workflow are
stored in `method`, while the `results` section stores output from the workflow beyond observables
of single configurations. For example, any ensemble-averaged quantity from a molecular dynamics
simulation would be stored under `workflow/results`. Then, the `inputs`, `outputs`, and `tasks` sections
are used to define the specifics of the workflow. For some [standard workflows](references/standard_workflows.md), e.g., geometry optimization and molecular
dynamics, the NOMAD [normalizers]() <!-- TODO Link to normalizer docs  --> will automatically populate these specifics. The parser must only create the
appropriate workflow section. <!-- TODO Should give an example somewhere -->
For non-standard workflows, the parser (or more appropriately the corresponding normalizer) must
populate these sections accordingly.
More information about the structure of the workflow section, as well as instructions on how to upload custom workflows to link individual Entries
in NOMAD, can be found in the [Workflow DOCS](https://fairmat-nfdi.github.io/AreaC-DOC-workflows/)

## Recommended parser layout

The following represents the recommended core structure for a computational parser, <!-- TODO general comp. or just atomistic?? -->
typically implemented within `<parserproject>/parser.py`.

### Imports

The imports typically include the necessary generic python modules, the required MetaInfo
classes from nomad, and additional nomad utilities, e.g., from `nomad.atomutils`.

```python
<license>
import os
import numpy as np
import logging

from nomad.units import ureg
from nomad.parsing.file_parser import FileParser
from nomad.datamodel.metainfo.simulation.run import Run, Program
from nomad.datamodel.metainfo.simulation.method import (
    Method, ForceField, Model, Interaction, AtomParameters
)
from nomad.datamodel.metainfo.simulation.system import (
    System, Atoms, AtomsGroup
)
from nomad.datamodel.metainfo.simulation.calculation import (
    Calculation
)
from nomad.datamodel.metainfo.workflow import (
    Workflow, MolecularDynamics
)
from nomad.datamodel.metainfo.simulation import workflow as workflow2
from nomad.atomutils import get_molecules_from_bond_list, is_same_molecule, get_composition
```


### Parser Classes

```python
class <Parsername><Mainfiletype>Parser(FileParser):
    def __init__(self):
        super().__init__(None)

    @property
    def file<mainfiletype>(self):
        if self._file_handler is None:
            try:
                self._file_handler = <openfilefunction>(self.mainfile, 'rb')
            except Exception:
                self.logger.error('Error reading <mainfiletype> file.')

        return self._file_handler


class <Parsername>Parser:
    def __init__(self):
        self.<mainfiletype>_parser = <Parsername><Mainfiletype>Parser()
```

The main class for your parser, `<Parsername>Parser`, will contain the bulk of the parsing routine,
described further below. It may be useful to create a distinct class, `<Parsername><Mainfiletype>Parser`,
for dealing with various filetypes that may be parsed throughout the entirety of the routine.
However, in the simplest case of a single file type parsed, the entire parser can of course be
implemented within a single class.

In the following, we will walk through the layout of the `<Parsername>Parser` class. First,
every parser class should have a "main" function called `parse()`, which will be called by NOMAD
when the appropriate mainfile is found:

<!-- TODO Add comments to all this code  -->
```python
def parse(self, filepath, archive, logger):

    self.filepath = os.path.abspath(filepath)
    self.archive = archive
    self.logger = logging.getLogger(__name__) if logger is None else logger
    self._maindir = os.path.dirname(self.filepath)
    self._<parsername>_files = os.listdir(self._maindir)  # get the list of files in the same directory as the mainfile
    self._basename = os.path.basename(filepath).rsplit('.', 1)[0]

    self.init_parser()

    if self.<mainfileparser>_parser is None:
        return

    sec_run = self.archive.m_create(Run)
    sec_run.program = Program(name='<PARSERNAME>', version='unknown')

    self.parse_method()
    self.parse_system()
    self.parse_calculation()
    self.parse_workflow()
```

Then, the individual functions to populate that various MetaInfo sections can be defined:

```python
def parse_calculation(self):
    sec_run = self.archive.run[-1]
    sec_calc = sec_run.m_create(Calculation)

    # populate calculation metainfo
    # ...

def parse_system(self, frame):
    sec_run = self.archive.run[-1]
    sec_system = sec_run.m_create(System)
    sec_atoms = sec_system.m_create(Atoms)

    # populate system metainfo
    # ...

def parse_method(self, frame):
    sec_method = self.archive.run[-1].m_create(Method)
    sec_force_field = sec_method.m_create(ForceField)
    sec_model = sec_force_field.m_create(Model)

    # populate method metainfo
    # ...

def parse_workflow(self):

    sec_workflow = self.archive.m_create(Workflow)  # for old workflow, should update for workflow2
    # populate workflow metainfo
    # ...
```

For more information, see [Examples - populating the NOMAD archive ](references/examples_populating_archive.md).