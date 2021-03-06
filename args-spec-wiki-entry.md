# The ARGS_SPEC

All InVEST models will own a data structure with information about the model’s inputs.
This will be a dictionary (`ARGS_SPEC`) with the structure detailed below.

## Purpose
The purpose of the `ARGS_SPEC` is to document the model's inputs to a level of detail sufficient that 
someone with no prior knowledge of the model could create inputs that are structurally and mathematically valid.

Current uses of the ARGS_SPEC are:
- in a validation function, reducing the amount of work needed to properly and effectively validate model inputs.
- to generate portions of the user's guide that cover the same content.
- in the workbench UI, to label and describe input fields.

## Contents
*(Note that this is distinct from creating inputs that are scientifically valid. 
Contextual information about finding data, choosing appropriate values, etc. belongs in the user's guide and not in the `ARGS_SPEC`.)*

## Structure
The `ARGS_SPEC` dictionary's keys are:

### `model_name`
type: `str`

The human-readable name of the model (e.g. “Habitat Risk Assessment”). This is displayed in the UI.

Style guidelines:
- Don't include "Model" in the name

### `module`
type: str

The python-importable module name (e.g. “natcap.invest.hra”). In practice, use `__name__`

### `userguide_html` 
type: `str`

The html page of the UG for this model, relative to the sphinx HTML build root (e.g. “habitat_risk_assessment.html”)

### `args_with_spatial_overlap`
Optional

type: `dict`

This is an optional dictionary with two recognized keys:
    * `"spatial_keys"` (`list[str]`): list of arg keys in `ARGS_SPEC['args']` that are spatial inputs
    * `"different_projections_ok"` (`bool`): Whether it's okay for the rasters and/or vectors in `spatial_keys` to have different projections
    
### `args`
type: `dict`

This is a dictionary describing all of the model's arguments (inputs). It maps unique arg identifiers to dictionaries that describe each arg. Keys are strings and are often similar to the corresponding arg `name`, but use underscores instead of spaces, are often shorter, and use abbreviations.

#### ARGS_SPEC args

Each value in `ARGS_SPEC['args']` is a dictionary that describes that arg with the following properties:
    
#### `name` 
`str`

The human-readable name of this input. The workbench UI displays the "name" property as a label for each input.

Style guidelines:
    * The name should be kept short. Extra description should go in "about" which becomes the tooltip text.
    * It should be all lower-case, except for things that are always capitalized (acronyms, proper names). Any capitalization rules such as "always capitalize the first letter" will be applied on the workbench side.
        
#### `type`
`str`

Indicates the input format.

Must be one of the following:
    * "boolean" - either `True` or `False` (or something that can be cast to `True` or `False`)
    * "csv" - a CSV on disk (comma-or-semicolon delimited, possibly with a UTF-8 BOM)
    * "directory" - a directory that may or may not exist on disk
    * "file" - a file that may or may not exist on disk
    * "freestyle_string" - a string that the user may customize with any valid character
    * "integer" - a unitless whole number (positive or negative) that uniquely identifies something, e.g. a landcover code
    * "number" - a scalar value.
    * "option_string" - a string where the value must belong to a set of options.
    * "percent" - a unitless proportion represented as a value in the range [0, 100]
    * "raster" - a path to a raster file
    * "ratio" - a unitless proportion represented as a value in the range [0, 1]
    * "vector" - a path to a vector file
        
#### `required` 
`bool` or `str`, optional, defaults to `True`.

Specifies if the input must be provided always, sometimes, or never.
    * If `True`, the input is required.
    * If `False`, the input is optional.
    * If a string, the string must be evaluate-able as a python expression, which must evaluate to a `bool`.
      The input is conditionally required based on the result of valuation.
      If any args keys are provided within the expression, they will evaluate to either
      `True` or `False` depending on whether the key-value pair is present in `args`,
      there is a value associated with that key, and that value is truthy.
          
#### `about`
type: `str`
Human-friendly plain text description of this input.
Style guidelines:

   Each arg dictionary may have additional required or optional attributes specific to its type.
   Four types (`"raster"`, `"vector"`, `"csv"`, `"directory"`) can contain multiple nested data types
   (bands, fields, rows/columns, and paths respectively). To represent that, each of these four types 
   
   These are described for each type:
    - for type `"option_string"`:
       - `"options"` (`list[str]`, required): A list of the allowed options. Dropdown menu options in the UI are made from this list, and this arg's value is validated against this list.
      
    - for type "number":
       - `"units"` (`pint.Unit`, required): A `pint.Unit` object that represents the arg's units e.g. meter, kilowatt-hour, etc.
         Note: to be compatible, all `Unit`s must belong to the same `pint.UnitRegistry` instance initialized in
         `src/natcap/invest/utils.py`.
         
    - for type "vector":
      * `"geometries`" (`set{str}`, required): A set of geometry names that are allowed
      * `"fields"` (`dict`, required):
      Allowed sub-types: number, ratio, percent, code, freestyle_string, option_string
      
    - for type "raster":
      * `"bands"` (`dict`, required): 
       Allowed sub-types: number, ratio, percent, code
    
    - for type "directory":
      * `"contents"` (dict, required): A dictionary mapping each row name to a nested arg dictionary describing the data in that row
      Allowed sub-types: directory, csv, raster, vector, file
       
    - for type "csv":
      The arg may not have both "columns" and "rows". It should use the one that best describes the table, such that 
      It's okay not to include either if the table is too complex to describe this way, such as >2 dimensional tables,
      tables with sub-tables, and tables that are parsed by row/column indices rather than names.
      * `"columns"` (dict, optional): 
      The value of the "columns" or "rows" key is a dictionary mapping each column name to a nested arg dictionary 
      describing the data in that column. 
         Example: Here is a simple table that has the columns "lucode" and "name".
         | lucode | name   |
         |--------|--------|
         | 1      | road   |
         | 3      | meadow |
         | ...    | ...    |
         
         Here is a description of that table in the `ARGS_SPEC` format:
         ```
         "columns": {
            "lucode": {
               "type": "integer",
               "about": "LULC code"
            },
            "name": {
               "type": "freestyle_string",
               "about": "description of the LULC class"
            }
         }
         ```
      * `"rows"` (`dict`, optional):
        Example: Here is a simple table that has the rows "radius" and "height".
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
         
## Style Guidelines for the `about` Text

### `boolean`s
The `about` text should say what happens when the value is `True`, beginning with an imperative: "Run the valuation model.", "Calculate flow direction from the provided DEM." This is more concise than phrasing like "Whether or not to ...". Avoid phrases like "If checked," "If selected,", "Check this box to..." which reference that the UI displays a checkbox for boolean types. The `about` text should be independent of the UI.
  
### `number` `expression`s
The `expression` can be any python expression and so can't be auto-documented into a user-friendly sentence. Any restrictions on numbers must also be described in the `about` text. For example, if the `expression` is `"value >= 0"`, the `about` text should say something like, "Must be greater than or equal to 0."

### conditional requirements 
The `required` value can be any python expression and so can't be auto-documented into a user-friendly sentence. Any conditional requirement must also be described in the `about` text. For example, if the `required` value is `"run_valuation"`, the `about` text should say something like, "Required if Run Valuation is selected."

### cross-referencing values
Values that are cross-referenced across multiple inputs, like LULC codes in a LULC raster and biophysical table, need to be documented. The args spec doesn't store this dependency info at all. We like this phrasing to explain the dependency between input values: "All values in X must have corresponding entries in Y". For example, "All values in this raster must have corresponding entries in the Biophysical Table."

### square bracket notation
Suggested phrasing to explain the square bracket notation used with non-static column names:
“Replace [XYZ] with <thing> names matching those in the <other thing>, so that there is one column for each <thing>.” Such as for the **foraging_activity_[SEASON]_index** columns in Pollination: “Replace [SEASON] with season names matching those in the Biophysical Table, so that there is one column for each season.”
