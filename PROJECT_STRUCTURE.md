# Project Structure Overview

This repository implements a calculator and visualiser for spectral sequences.
The core algebraic logic lives in a small collection of Python modules that
model the pages, differentials, and algebraic elements.  The `seqsee_main.py`
script consumes JSON input that describes a spectral sequence chart and renders
an interactive HTML document.

## Algebraic Core

### `utilities.py`
* **`Prime`** – memoised helpers for testing primality (`is_prime`) and
  retrieving the first `n` primes (`first_n_prime`).
* **`Matrix`** – thin wrapper around SymPy's matrices that adds ordering, hash
  support, smart concatenation (`hstack`), and multi-matrix reduction
  (`multi_reduction`).
* **`Vector`** – column-vector specialisation of `Matrix` that simplifies
  indexing and representation.
* **`Poly`** – subclass of SymPy's `Poly` that prints monomials as expressions.
* **`_next_config`** – iterates through coefficient configurations within
  prescribed bounds for brute-force enumeration.
* **`convex_integral_combinations`** – enumerates all non-negative integral
  combinations of vectors that hit a target bi-degree; used throughout the
  spectral sequence code to find admissible monomials.

### `element.py`
* **`Bidegree`** – vector subclass representing the `(p, q)` degree of a class.
* **`HomoElem`** – wraps homogeneous polynomials, tracking both their absolute
  bi-degree and the actual class that survives on a given page.  The class
  provides arithmetic (`__add__`, `__sub__`, `__mul__`, `__pow__`) and
  introspection helpers (`isZero`, `divides`).

### `module.py`
* **`Module`** – linear algebra structure for a fixed bi-degree on a page.  It
  stores the surviving subspace, the kernel, and exposes a `classify` method
  that labels coordinates as kernel elements, survivors, or invalid vectors.

### `page.py`
* **`Page`** – represents a single page `E_r` of the spectral sequence.  It
  caches `Module` instances per bi-degree and owns a `Differential`.  The
  `generate_module` method builds new modules by combining kernel and image data
  from the previous page.

### `differential.py`
* **`Differential`** – calculates the matrix representation of the page
  differential.  The `get_matrix` method expands known differentials, derives
  new relations via products, and prompts for missing information when the
  algebra alone is insufficient.

### `spectral_sequence.py`
* **`SpectralSequence`** – orchestrates the entire computation by storing the
  generators, relations, and page configuration.  It provides convenience
  utilities such as `kill` (register relations), `get_abs_basis`,
  `get_abs_dimension`, `get_abs_info`, and `add_page` (construct a new page with
  the appropriate differential degree).

## Visualisation Script

### `seqsee_main.py`
* **`CssStyle`** – helper class that accumulates nested CSS declarations and
  can render them to text.
* **`load_schema`** / **`load_template`** – load the JSON schema used to
  validate inputs and the Jinja2 template used to render the HTML output.
* **`get_schema_default`** / **`get_value_or_schema_default`** – fetch default
  values from the schema, falling back to data when provided.
* **`cssify_name`** – convert arbitrary identifiers into CSS-safe class names.
* **`style_and_aliases_from_attributes`** – translate the attribute lists used
  in the JSON format into concrete CSS declarations and aliases.
* **`generate_style`** – combine a base style with alias lookups from
  `global_css` to produce a final `CssStyle` object.
* **`ensure_json_path_is_defined`** – lazily populate nested dictionaries so
  later code can assume the structure exists.
* **`compute_chart_dimensions`** – determine the SVG viewport bounds by
  inspecting the nodes included in the chart.
* **`calculate_absolute_positions`** – compute visual offsets when multiple
  nodes share the same bi-degree so that they render without overlap.
* **`generate_nodes_svg`** / **`generate_edges_svg`** – create the SVG snippets
  that render the spectral sequence chart.
* **`generate_svg`** – orchestrate SVG generation after preparing absolute
  positions.
* **`generate_html`** – render the entire HTML document by combining CSS, SVG,
  and metadata through the Jinja2 template.
* **`set_scale`** – update the global pixel scaling used to render coordinates.
* **`generate_css_styles`** – populate `global_css` with colour and attribute
  aliases derived from the schema and input data.
* **`process_json`** – validate and process a JSON file on disk, writing the
  resulting HTML output.
* **`process_data`** – variant of `process_json` that operates on an in-memory
  dictionary.
* **`main`** – command-line entry point; expects input and output file paths.

## Ancillary Files

* **`README.md`** – high-level description of the project goals.
* **`DevelopmentDoc.md`** – notes collected during the original development.
* **`template.html.jinja`** – HTML template consumed by `seqsee_main.py`.
* **`input_schema.json`** – JSON schema used to validate chart descriptions.
* **`Test.py` / `test2.py` / `test3.py`** – assorted experimental scripts and
  manual tests for the rendering pipeline.

This overview should help new contributors orient themselves quickly and locate
functionality when extending the spectral sequence calculator or its charting
front-end.
