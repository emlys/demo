#### The ARGS_SPEC

All InVEST models will own a data structure with information about the model’s inputs.
This will be a dictionary (`ARGS_SPEC`) with the structure detailed below.
The purpose of the `ARGS_SPEC` is to document the model's inputs to a level of detail sufficient that 
someone with no prior knowledge of the model could create inputs that are structurally and mathematically valid.
*(Note that this is distinct from creating inputs that are scientifically valid. 
Contextual information about finding data, choosing appropriate values, etc. belongs in the user's guide and not in the `ARGS_SPEC`.)*

The `ARGS_SPEC` is used in a validation function, reducing the amount of work needed to properly and effectively validate model inputs.
In the future, the `ARGS_SPEC` may also be used to generate portions of the user's guide that cover the same content.

The `ARGS_SPEC` dictionary's keys are:

* "model_name": “Habitat Risk Assessment” (The human-readable name of the model)
* "module": “natcap.invest.hra” (The python-importable module name, in practice, use `__name__`)
* "userguide_html": “habitat_risk_assessment.html” (The html page of the UG for this model, relative to the sphinx HTML build root)
* "args_with_spatial_overlap":
    * "spatial_keys": [list of string keys]
    * "different_projections_ok": True or False
* "args": (A dict describing all possible args keys to the model)
    * <args_key>: (the args key, e.g. ‘workspace_dir’)
    * "name": The human-readable name of this input. The workbench UI displays the "name" property as a label for each input. As such, we want to keep them consistent:
        * The name should be as short as possible. Extra description should go in "about" which becomes the tooltip text.
        * It should be all lower-case, except for things that are always capitalized (acronyms, proper names). Any capitalization rules such as "always capitalize the first letter" will be applied on the workbench side.
    * "type": `<string type>` (one of the following):
        * "directory" - a directory that may or may not exist on disk
        * "file" - a file that may or may not exist on disk
        * "raster" - a raster that can be opened with GDAL
        * "vector" - a vector that can be opened with GDAL
        * "csv" - a CSV on disk (comma-or-semicolon delimited, possibly with a UTF-8 BOM)
        * "number" - a scalar value.
        * "ratio" - a unitless proportion represented as a value in the range [0, 1]
        * "percent" - a unitless proportion represented as a value in the range [0, 100]
        * "code" - a unitless positive integer that uniquely identifies something, e.g. a landcover code
        * "freestyle_string" - a string that the user may customize with any valid character
        * "option_string" - a string where the value must belong to a set of options.
        * "boolean" - either `True` or `False` (or something that can be cast to `True` or `False`)
    * "required" (optional, defaults to `True`): `bool` | `<string>`
        * If `True`, the input is required.
        * If `False`, the input is optional.
        * If a string, the string must be evaluate-able as a python expression, which must evaluate to a `bool`.
          The input is conditionally required based on the result of valuation.
          If any args keys are provided within the expression, they will evaluate to either
          `True` or `False` depending on whether the key-value pair is present in `args`,
          there is a value associated with that key, and that value is truthy.
    * "about": String text about this input.

   Each arg dictionary may have additional required or optional attributes specific to its type.
   Four types (`"raster"`, `"vector"`, `"csv"`, `"directory"`) can contain multiple nested data types
   (bands, fields, rows/columns, and paths respectively). To represent that, each of these four types 
   
   These are described for each type:
    - for type `"option_string"`:
      * "options" (list, required): A list of strings that are the allowed options. 
         Dropdown menu options in the UI are made from this list, and this arg's value is
         validated against this list.
    - for type "number":
       * units (pint.Unit, required): A `pint.Unit` object that represents the arg's units e.g. meter, kilowatt-hour, etc.
         Note: to be compatible, all `Unit`s must belong to the same `pint.UnitRegistry` instance initialized in
         src/natcap/invest/utils.py.
    - for type "vector":
      * "geometries" (set, required): A set of strings
      * "fields" (dict, required):
      Allowed sub-types: number, ratio, percent, code, freestyle_string, option_string
    - for type "raster":
      * "bands" (dict, required): 
       Allowed sub-types: number, ratio, percent, code
    - for type "csv":
      The arg may not have both "columns" and "rows". It should use the one that best describes the table, such that 
      It's okay not to include either if the table is too complex to describe this way, such as >2 dimensional tables,
      tables with sub-tables, and tables that are parsed by row/column indices rather than names.
      * "columns" (dict, optional): 
      The value of the "columns" or "rows" key is a dictionary mapping each column name to a nested arg dictionary 
      describing the data in that column. 
         Example: Here is a simple table that's expected to have the columns "lucode" and "name".
         | lucode | name   |
         |--------|--------|
         | 1      | road   |
         | 3      | meadow |
         | ...    | ...    |
         
         Here is a description of that table in the `ARGS_SPEC` format:
         ```
         "columns": {
            "lucode": {
               "type": "code",
               "about": "
            },
            "name": {
               "type": "freestyle_string",
               "about": "description of the LULC code"
            }
         }
         ```
      * "rows" (dict, optional):
        Example: Here is a simple table that's expected to have the rows "radius" and "height".
         |            |    |
         | ---------- | -- |
         | **radius** | 20 |
         | **height** | 50 |
         
         Here is a description of that table in the `ARGS_SPEC` format:
         ```
         "rows": {
            "radius": {
               "type": "number",
               "units": u.meter
            },
            "height": {
               "type": "number",
               "units": u.meter
            }
         }
         ```
    - for type "directory":
      * "contents" (dict, required): A dictionary mapping each row name to a nested arg dictionary describing the data in that row
      Allowed sub-types: directory, csv, raster, vector, file
    * "validation_options": (type-specific validation options. The options listed here are passed as kwargs to the appropriate `check_*` function defined in `natcap.invest.validation`. See those function definitions and docstrings for more details.
