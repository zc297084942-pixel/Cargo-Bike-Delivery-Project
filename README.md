# Cargo-Bike Delivery Routing: Scenario-Level and Cycle-Level Insertion

This project evaluates two approaches to reorganizing cargo-bike deliveries from a micro-hub. The objective is to minimize total network travel distance, including each return to the hub, with a cargo-bike capacity of 14 deliveries per cycle.

The Python workflow reads delivery plans, matches activity coordinates to network nodes, constructs routes using a cheapest-insertion heuristic, and exports detailed Excel comparisons against the original delivery sequences. The example illustrates spatial data processing, heuristic routing, and transparent reporting of operational results.

## Research question

How much delivery distance can be saved by improving stop order within existing cycles, and how much additional saving is possible when deliveries can also be regrouped across cycles?

Cycle-level insertion evaluates improvements while retaining the original delivery assignments. Scenario-level insertion evaluates the value of allowing those assignments to change. This comparison helps assess the operational benefit of greater delivery-grouping flexibility, while identifying the consequences of count-based cycle splitting.

## Broader experiment: results reported across 40 cases

The accompanying report, `semi_report_tamu_CZ.docx` (Cargo-Bike Insertion Simulation Results), describes experiments across **40 input files, 240 scenarios, 4,800 deliveries, and 447 cycles**. Its aggregate results are:

| Method | Original distance (km) | Resulting distance (km) | Distance saved (km) | Reduction |
| --- | ---: | ---: | ---: | ---: |
| Cycle-level basic insertion | 6,770.527 | 3,994.988 | 2,775.539 | 40.99% |
| Scenario-level basic insertion | 6,770.527 | 3,499.128 | 3,271.399 | 48.32% |

Across the reported experiment, scenario-level insertion saves an additional **495.860 km** compared with cycle-level insertion. Relative to the common original baseline, this is approximately **7.32 percentage points** of additional distance reduction, calculated from the reported distance totals.

The report also identifies three cases where scenario-level insertion performs slightly worse than cycle-level insertion: `LowSW8`, `MidSW8`, and `MidSW8_Block4`. A longer pooled route can introduce costly hub-return legs when divided at fixed capacity boundaries. These exceptions show why cross-cycle regrouping does not guarantee a shorter final set of cycles and motivate future hub-aware splitting or local improvement around cycle boundaries.

**Reproducibility scope:** The 40-case totals above are taken from the report. The supplied code-and-data package contains the HighNE4 input and supporting network data, not all 40 XML inputs. Running the included script reproduces the HighNE4 example below; it does not reproduce the entire 40-case experiment in one run.

## Reproducible example: HighNE4

Before running the script, extract od_network_distance_matrix.zip into the project folder. Ensure that od_network_distance_matrix.csv is alongside process_highne4_two_methods_commented.py.

The included `HighNE4.xml` contains **4 scenarios and 120 deliveries**. Each XML person is treated as a separate scenario. Both methods produce 11 cycles in total for this example.

| Method | Original distance (km) | Resulting distance (km) | Distance saved (km) | Reduction |
| --- | ---: | ---: | ---: | ---: |
| Scenario-level basic insertion | 217.181 | 92.961 | 124.220 | 57.20% |
| Cycle-level basic insertion | 217.181 | 122.263 | 94.919 | 43.70% |

These values were reproduced by running the supplied `process()` function against the included inputs. They describe this example, rather than a guarantee of performance for other datasets. The coordinate-to-node matching error is zero for the included example.

## Files

Keep the following files together in the same working directory when running the script.

| File | Purpose |
| --- | --- |
| `process_highne4_two_methods_commented.py` | Commented Python implementation of both routing methods and Excel reporting. |
| `HighNE4.xml` | MATSim-format activity plans containing the scenarios, micro-hub visits, and deliveries. |
| `od_network_distance_matrix.csv` | Directed network distance matrix in meters, with 1,796 origins and 1,796 destinations. The first column contains row node IDs; the header contains destination node IDs. |
| `b_line_network_combined_linked_nodes.xlsx` | Network workbook containing `Summary`, `Linked_Nodes`, `Nodes`, and `Links`. The script reads `node_id`, `x`, and `y` from `Nodes`. |
| `HighNE4_basic_scenario_and_cycle_level_insertion_results.xlsx` | Included reference output for the reproducible HighNE4 example. Running the script overwrites this file by default. |
| `MidNE4_basic_scenario_and_cycle_level_insertion_results.xlsx` | Additional supplied output. Its corresponding `MidNE4.xml` input is not included, so it is not a separately reproducible example in this package. Its method-level totals match those of the HighNE4 reference output. |
| `pseudocode.docx` | Pseudocode describing the cycle-level and scenario-level procedures. |

The separately supplied `semi_report_tamu_CZ.docx` provides the broader experiment, comparative figures, and detailed results table. It may be placed alongside these files as supporting documentation; it is not required to run the code.

## Methods

### Common preparation and baseline

1. Select the plan marked `selected="yes"` for each person. If no plan is marked, use the first plan.
2. Identify `micro_hub` and `delivery` activities. Micro-hub visits delimit the original delivery cycles.
3. Match each activity coordinate to its nearest network node using SciPy's `cKDTree`.
4. Calculate the baseline distance by summing the original XML cycles, including hub departure and return legs.

Both methods use the same matched nodes, directed OD matrix, and original baseline.

### Scenario-level basic insertion

Pool all deliveries within a scenario. Starting with an empty hub-to-hub route, repeatedly insert the delivery at the position with the smallest incremental distance. Split the resulting delivery sequence into consecutive groups of at most 14, then add hub departure and return legs to every group.

Deliveries can move across their original XML cycle boundaries. Split points are determined by delivery count; they are **not optimized for hub-return distance**.

Code identifier: `basic_scenario_level_no_hub_aware`.

### Cycle-level basic insertion

Preserve the original XML cycle membership and apply cheapest insertion separately within each cycle. Deliveries can be reordered inside a cycle but cannot move between cycles.

Code identifier: `cycle_level_basic_insertion`.

The script does not split or reject oversized original cycles in this method. The included example has no original cycle exceeding 14 deliveries; other inputs require a capacity check before use.

### Insertion rule

For a candidate delivery `c` inserted between nodes `a` and `b`, the added distance is:

```text
increment = distance(a, c) + distance(c, b) - distance(a, b)
```

All candidate deliveries and insertion positions are considered at each step. Ties are resolved by delivery index, then insertion position. Distances preserve direction, so `distance(a, b)` need not equal `distance(b, a)`.

## Run the example

Use a Python 3 environment with NumPy, pandas, SciPy, and openpyxl installed. The supplied package does not pin dependency versions.

From a terminal opened in the folder containing the script and inputs:

```bash
python -m pip install numpy pandas scipy openpyxl
python process_highne4_two_methods_commented.py
```

No MATSim installation or commercial optimization solver is required to run this script. XML parsing uses Python's standard library.

The script writes `HighNE4_basic_scenario_and_cycle_level_insertion_results.xlsx` and prints the output filename and method summaries. Close the workbook in Excel before running if your operating system locks open files. Copy the reference workbook elsewhere first if you want to retain it unchanged.

To analyze another input, edit `XML_PATH` and `OUT_XLSX` in the script's settings section. Update `OD_CSV`, `NETWORK_XLSX`, and `CAPACITY` as needed. Paths are resolved relative to the terminal's working directory, not automatically relative to the script.

## Output workbook

| Worksheet | Contents |
| --- | --- |
| `Method_Summary` | Aggregate counts, before/after distances, savings, reduction, and maximum coordinate-matching error for each method. |
| `Scenario_Comparison` | Scenario-level deliveries, cycle counts, capacity setting, hub node, and distance comparisons. |
| `Cycle_Comparison` | Original and resulting cycle distances, delivery counts, display routes, and network-node routes. |
| `Route_Order_Display` | Route sequences using both display labels and actual network node IDs. |
| `Leg_Details_Display` | Individual directed legs and their distances. |
| `Node_Mapping` | Activity coordinates, matched network nodes, and nearest-node matching errors. |
| `Notes` | Definitions of the methods and major processing conventions. |

### Interpreting the results

- `before_km` and `after_km`: total closed-cycle distance before and after reordering.
- `saved_km`: `before_km - after_km`.
- `reduction_pct`: savings divided by baseline distance, stored as a fraction and displayed as a percentage in Excel. Aggregate reduction uses aggregate distances, not an average of scenario percentages.
- `display_label`: original XML activity position for deliveries; the hub is displayed as `1`. These labels are not network node IDs and need not be consecutive.
- `actual_node` / actual-node routes: matched node IDs used for OD lookups. Hub display label `1` does not imply network node ID `1`.
- `match_error` / `max_match_error`: Euclidean coordinate-to-node matching distance in the input coordinate system's units, not the OD travel distance.

Original baseline routes are repeated under both methods in the detailed sheets to support comparisons. Do not sum those baseline records across methods as though they represent different deliveries.

## Assumptions and limitations

- Each delivery consumes one capacity unit. Every cycle starts and ends at the scenario's first identified micro-hub; inputs with multiple distinct hubs need additional handling.
- The objective is distance only. XML activity times and durations are recorded during parsing but are not enforced as time windows or service-time constraints. Travel time, rider shifts, battery range, and fleet scheduling are not optimized.
- The OD matrix is supplied as input. This script does not compute shortest paths or verify the matrix's network or mode assumptions.
- Node IDs must correspond to consecutive 1-based row and column positions in the CSV. The script converts the matrix to an array and uses `node_id - 1`; it does not remap arbitrary node labels.
- XML and network coordinates must use the same coordinate reference system. No coordinate transformation or maximum matching-error threshold is applied.
- Cheapest insertion is a heuristic with no global optimality guarantee. No acceptance safeguard retains the original route when the heuristic performs worse, so negative savings are possible.
- Scenario-level splitting can change cycle membership and count. The final evaluation includes all resulting hub-return legs, but the splitting decision itself does not optimize them.
- Regret insertion, hub-aware dynamic programming, and route-map generation are not implemented in this supplied script.

## Data documentation and reuse

The package contains the inputs required for the HighNE4 example. External data provenance and licensing are not specified in the supplied files; no additional reuse permissions are asserted here.
