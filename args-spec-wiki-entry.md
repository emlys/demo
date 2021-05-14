#### The ARGS_SPEC

All InVEST models will own a data structure with information about the model’s inputs.
This will be a dictionary (`ARGS_SPEC`) with the structure detailed below.
The `ARGS_SPEC` will be used in a validation function, reducing the amount of work needed to properly and effectively validate model inputs.

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
        * “Directory” - a directory that may or may not exist on disk
        * “File” - a file that may or may not exist on disk
        * “Raster” - a raster that can be opened with GDAL
        * “Vector” - a vector that can be opened with GDAL
        * “CSV” - a CSV on disk (comma-or-semicolon delimited, possibly with a UTF-8 BOM)
        * “Number” - a scalar value.
        * “FreestyleString” - a string that the user may customize with any valid character
        * “OptionString” - a string where the value must belong to a set of options.
        * “Boolean” - either true or false (or something that can be cast to True or False)
    * "required": `True` | `False` | `<boolean expression of args_keys>`
        * If `True`, the input is required.
        * If `False`, the input is optional.
        * If an expression, the input is conditionally required based on evaluating the expression.
          If any args keys are provided within the expression, they will evaluate to either
          `True` or `False` depending on whether the key-value pair is present in `args`,
          there is a value associated with that key, and that value is truthy.
    * "about": String text about this input.
    * "validation_options": (type-specific validation options. The options listed here are passed as kwargs to the appropriate `check_*` function defined in `natcap.invest.validation`. See those function definitions and docstrings for more details.
