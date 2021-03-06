# :construction: PFB Python SDK :construction:

Python SDK to create, explore and modify PFB (Portable Format for Biomedical Data) files.


## PyPFB Overview

PyPFB is a python sdk to create, explore, and modify PFB (Portable Format for Bioinformatics) files.

These files start from a Gen3 data dictionary. These can be made from either json hosted on a cloud storage service, like s3, or from a local directory. See PyPFB From Schema for an example.

Once we have a PFB file created from a schema we can start to add data to the file. This is done using JSON files from local directory. We create them in the style of our data-simulator https://github.com/uc-cdis/data-simulator/ Once we have them we can use PFB From JSON to import the structured json into our Serialized PFB file.

At this point we have a PFB file with married schema and serialized data. Now we have a few options for modifying these PFB files. These are good options for breaking changes within the dictionary. This allows a commons operator to export the entire structured database, make modifications to fix the breaking changes, and then re import the file back to the commons.

Changes that are already supported by this SDK are renames (enum and nodes) and adds of records data.


## PFB Schema

[![metadata][1]][1]

## Installation

* From PyPI:

```bash
pip install pypfb[gen3]
```

(The optional `gen3` dependencies add the ability to convert a Gen3 data dictionary into
a PFB file.)

* From source code:

```bash
pip install pipenv
pipenv install
```

(Also add `--dev` for development.)


## Usage

### Main

    Usage: pfb [OPTIONS] COMMAND [ARGS]...

      PFB: Portable Format for Biomedical Data.

    Commands:
      add     Add records into a PFB file.
      from    Generate PFB from other data formats.
      make    Make a blank record for add.
      rename  Rename different parts of schema.
      show    Show different parts of a PFB file.
      to      Convert PFB into other data formats.

### Show different parts of PFB

    Usage: pfb show [OPTIONS] COMMAND [ARGS]...

      Show records of the PFB file.

      Specify a sub-command to show other information.

    Options:
      -i, --input FILENAME  The PFB file.  [default: <stdin>]
      -n, --limit INTEGER   How many records to show, ignored for sub-commands.
                            [default: no limit]

    Commands:
      metadata  Show the metadata of the PFB file.
      nodes     Show all the node names in the PFB file.
      schema    Show the schema of the PFB file.

    Examples:
      schema:
        pfb show -i data.avro schema
      nodes:
        pfb show -i data.avro nodes
      metadata:
        pfb show -i data.avro metadata
      records:
        pfb show -i data.avro -n 5

### Convert Gen3 data dictionary into PFB schema

    Usage: pfb from [PARENT OPTIONS] dict DICTIONARY

      Convert Gen3 data DICTIONARY into a PFB file.

      If DICTIONARY is a HTTP URL, it will be downloaded and parsed as JSON; or
      it will be treated as a local path to a directory containing YAML files.

    Parent Options:
      -o, --output FILENAME  The output PFB file.  [default: <stdout>]

    Examples:
      URL:
        pfb from -o thing.avro dict https://s3.amazonaws.com/dictionary-artifacts/gtexdictionary/3.2.2/schema.json
      Directory:
        pfb from -o gdc.avro dict /path/to/dictionary/schemas/

### Convert JSON for corresponding datadictionary to PFB

    Usage: pfb from [PARENT OPTIONS] json [OPTIONS] [PATH]

      Convert JSON files under PATH into a PFB file.

    Parent Options:
      -o, --output FILENAME  The output PFB file.  [default: <stdout>]

    Options:
      -s, --schema FILENAME  The PFB file to load the schema from.  [required]
      --program TEXT         Name of the program.  [required]
      --project TEXT         Name of the project.  [required]

    Example:
      pfb from -o data.avro json -s schema.avro --program DEV --project test /path/to/data/json/

### Make new blank record

    Usage: pfb make [OPTIONS] NAME

      Make a blank record according to given NODE schema in the PFB file.

    Options:
      -i, --input PFB  Read schema from this PFB file.  [default: <stdin>]

    Example:
      pfb make -i test.avro demographic > empty_demographic.json

### Add new record to PFB

    Usage: pfb add [OPTIONS] PFB

      Add records from a minified JSON file to the PFB file.

    Options:
      -i, --input JSON  The JSON file to add.  [default: <stdin>]

    Example:
      pfb add -i new_record.json pfb.avro

### Rename different parts of PFB (schema evolution)

    Usage: pfb rename [OPTIONS] COMMAND [ARGS]...

      Rename different parts of schema.

    Options:
      -i, --input FILENAME   Source PFB file.  [default: <stdin>]
      -o, --output FILENAME  Destination PFB file.  [default: <stdout>]

    Commands:
      enum  Rename enum.
      node  Rename node.
      type  Rename type (not implemented).

    Examples:
      enum:
        pfb rename -i data.avro -o data_enum.avro enum demographic_ethnicity old_enum new_enum
      node:
        pfb rename -i data.avro -o data_update.avro node demographic information

### Rename node

    Usage: pfb rename [PARENT OPTIONS] node [OPTIONS] OLD NEW

      Rename node from OLD to NEW.

### Rename enum

    Usage: pfb rename [PARENT OPTIONS] enum [OPTIONS] FIELD OLD NEW

      Rename enum of FIELD from OLD to NEW.

### Convert PFB into Neptune (bulk load format for Gremlin)

    Usage: pfb to [PARENT OPTIONS] gremlin [OPTIONS] [OUTPUT]

      Convert PFB into CSV files under OUTPUT for Neptune bulk load (Gremlin).

      The default OUTPUT is ./gremlin/.

    Options:
      --gzip / --no-gzip  Whether gzip the output.  [default: yes]

    Example:
      pfb to -i data.avro gremlin


## Examples

    pfb from dict http://s3.amazonaws.com/dictionary-artifacts/kf-dictionary/1.1.0/schema.json > ./tests/schema/kf.avro

    pfb from json ./tests/data -s ./tests/schema/kf.avro --program DEV --project test > tests/pfb-data/test.avro

    cat tests/pfb-data/test.avro | pfb rename node slide slide_test > tests/pfb-data/rename_test.avro

    cat tests/pfb-data/test.avro | pfb rename enum state validated validated_test > tests/pfb-data/rename_test.avro

    cat tests/pfb-data/test.avro | pfb show -n 1 | jq

    cat tests/pfb-data/test.avro | pfb show --schema | jq

    cat tests/pfb-data/test.avro | pfb to gremlin ./output/

## Minimal File Handoff PFB Example

```bash
# Minimal file handoff PFB examples
# this is all done in the minimal_example/ directory
$> cd minimal_example

# clear out any old .avro files
$> rm minimal*.avro

# Create the PFB avro based on the minimal_file schema
$> pfb from -o minimal_file.avro dict minimal_file.json

# output
Loading dictionary: minimal_file.json
Parsing dictionary...
Writing PFB...
Done, created PFB file at: minimal_file.avro

# make a template JSON to fill in for a file record
$> pfb make -i minimal_file.avro files | json_pp > temp_files.json

# edit temp_files.json to be the following and put its content in sample_file_json/files.json (you can see an example file in example_files.json).  Notice we're moving the "object" sub-structure up compared to the file generated for us.
{
   "id" : "7d9dd25c-2b9a-43cc-8b95-db1f8d6fe1dc",
   "name" : "submitted_aligned_reads",
   "submitter_id" : "HG01101_cram",
   "created_datetime" : "2020-01-27T14:32:19.373454+00:00",
   "error_type" : "file_size",
   "file_format" : "BAM",
   "file_name" : "foo.bam",
   "file_size" : 512,
   "file_state" : "registered",
   "md5sum" : "bdf121aadba028d57808101cb4455fa7",
   "object_id" : "dg.4503/cc32d93d-a73c-4d2c-a061-26c0410e74fa",
   "project_id" : "tutorial-synthetic_data_set_1",
   "state" : "uploading",
   "submitter_id" : "HG01101_cram",
   "subject_id" : "p1011554-9",
   "drs_uri" : "drs://example.org/dg.4503/cc32d93d-a73c-4d2c-a061-26c0410e74fa",
   "updated_datetime" : "2020-01-27T14:32:19.373454+00:00"
}

# now load the example data into the avro PFB file
$> pfb from -o minimal_file_data.avro json -s minimal_file.avro --program DEV --project test sample_file_json/

# output
Loading schema...
1/1: submitted_aligned_reads
{'id': '7d9dd25c-2b9a-43cc-8b95-db1f8d6fe1dc', 'name': 'submitted_aligned_reads', 'submitter_id': 'HG01101_cram', 'created_datetime': '2020-01-27T14:32:19.373454+00:00', 'error_type': 'file_size', 'file_format': 'BAM', 'file_name': 'foo.bam', 'file_size': 512, 'file_state': 'registered', 'md5sum': 'bdf121aadba028d57808101cb4455fa7', 'object_id': 'dg.4503/cc32d93d-a73c-4d2c-a061-26c0410e74fa', 'project_id': 'tutorial-synthetic_data_set_1', 'state': 'uploading', 'subject_id': 'p1011554-9', 'drs_uri': 'drs://example.org/dg.4503/cc32d93d-a73c-4d2c-a061-26c0410e74fa', 'updated_datetime': '2020-01-27T14:32:19.373454+00:00'}
NODE NAME: submitted_aligned_reads id
NODE NAME: submitted_aligned_reads name
NODE NAME: submitted_aligned_reads submitter_id
NODE NAME: submitted_aligned_reads error_type
NODE NAME: submitted_aligned_reads file_format
NODE NAME: submitted_aligned_reads file_name
NODE NAME: submitted_aligned_reads file_size
NODE NAME: submitted_aligned_reads file_state
NODE NAME: submitted_aligned_reads md5sum
NODE NAME: submitted_aligned_reads object_id
NODE NAME: submitted_aligned_reads project_id
NODE NAME: submitted_aligned_reads state
NODE NAME: submitted_aligned_reads subject_id
NODE NAME: submitted_aligned_reads drs_uri
Done!

# take a look inside the data avro file
$> cat minimal_file_data.avro | pfb show -n 1

# look at schema
$> cat minimal_file_data.avro | pfb show schema

# dump to csv
$> cat minimal_file_data.avro | pfb to gremlin ./output/
```

  [1]: ./doc/schema.svg
