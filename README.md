# github-action-morph-kgc
This GitHub Action action creates a knowledge graph from structured or semi-structured sources using RML mappings and [Morph-KGC](https://github.com/oeg-upm/morph-kgc).

## Usage
The mapping file extension needs to be `.rml.ttl` or `.rml.nt`.
In order to execute the action include the word `morph-kgc` in the comment of the commit, otherwise the action will not be triggered.
Create a `.github.workflows/morph-kgc.yaml` file in the repository with the following example workflow:

```
name: morph-kgc
on:   
  push:
    branches:    
      - main
  pull_request:
    branches:    
      - main

jobs:    
  validate:
    runs-on: ubuntu-latest
    name: action-morph-kgc
    steps:
      - name: checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0

       - name: commit trigger
        id: 'commit_trigger'
        run: |

          case "${{ github.event.head_commit.message }}" in
              *morph-kgc* ) echo "::set-output name=trigger::true";;
              * ) echo "::set-output name=trigger::false";;
          esac

      - name: installing morph-kgc
        run: |
          if ${{ steps.commit_trigger.outputs.trigger }}
          then
            pip install morph-kgc
          fi

      - name: action-morphkgc
        uses: ./
        id: 'action-morphkgc'
        with:
          na_filter: 'yes'
          na_values: ',#N/A,N/A,#N/A N/A,n/a,NA,<NA>,#NA,NULL,null,NaN,nan,None'
          output_dir: 'morph-kgc'
          output_file: 'result'
          output_format: 'N-QUADS'
          only_printable_characters: 'no'
          safe_percent_encoding: ':/'

      - name: running morph-kgc
        run: |
          if ${{ steps.action-morphkgc.outputs.run }} && ${{ steps.commit_trigger.outputs.trigger }}
          then
            python3 -m morph_kgc ./morph-kgc-exec/config.ini
            rm -r ./morph-kgc-exec
            git config --global user.name 'github-actions[bot]'
            git config --global user.email '41898282+github-actions[bot]@users.noreply.github.com'
            git add .
            set +e
            git status | grep "nothing to commit, working tree clean"
            if [ $? -eq 0 ]; then set -e; echo "INFO: No changes since last run"; else set -e; \
              git commit -m "morph-kgc result for ${{ github.actor }} - ${{ github.event.number }}" --allow-empty; git push --force; fi
          fi

```
## Inputs
### `output_dir` (optional)
The output directory for morph-kgc.
- `Default value`: output
### `output_file`  (optional)
File to write the results to. If it is empty (no value) then an independent output file is generated for each group of the mapping partition.

- `Default value`: result
### `output_format` (optional)
RDF serialization to use for output results.

- `Valid values`: N-TRIPLES, N-QUADS
- `Default value`: N-QUADS
### `only_printable_characters` (optional)
Remove characters in the genarated RDF that are not printable.

- `Valid values`: yes, no, true, false, on, off, 1, 0
- `Default value`: no
### `safe_percent_encoding` (optional)
Set of ASCII characters that should not be percent encoded. All characters are encoded by default.

- `Example value`: :/
- `Default value`:
### `na_filter` (optional)
If specified, the values from the option na_values will be interpreted as NULL.

- `Valid values`: yes, no, true, false, on, off, 1, 0
- `Default value`: yes
### `na_values` (optional)
Set of values to be interpreted as NULL when retrieving data from the input sources. The valid values are a list of values separated by commas.

## Outputs
### run
Tells the pipeline if morph-kgc needs to be executed.
- `true` if there are mapping files.
- `false` if there are no mapping files.

---

El trabajo realizado en este repositorio ha sido elaborado por Miguel Jorge Garcia-Muñoz en el contexto de un Trabajo Fin de Grado de la [ETSIINF](https://www.fi.upm.es/) del curso 2021-2022, bajo la dirección de Oscar Corcho.
