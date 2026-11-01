# MOS gm/ID LUT Processing Specification

Use this specification to convert local MOS sweep data into a reusable gm/ID LUT with loading, querying, and plotting scripts.

## 1. Preserve raw data

Keep original CSV files unchanged. Save cleaned data, LUT files, metadata, and scripts separately. Record every unit and sign conversion.

## 2. Standardize units

| Quantity | Unit |
|---|---|
| L, Wtotal, Wfinger | um |
| VGS, VDS, VSB, VT, VDSAT | V |
| ID and other currents | A |
| GM, GDS, GMB | S |
| All capacitances | F |
| temperature_C in metadata | degC |

For a pygmid-compatible file, use K for TEMP and document the conversion.

Derived units:
- GM/ID: 1/V
- ID/Wtotal: A/um
- Cgg/Wtotal: F/um
- GM/GDS: dimensionless
- GM/(2*pi*Cgg): Hz

Verify source units before conversion. Ask when units cannot be established; do not guess.

## 3. Define the data structure

Use parameter-array axis order (L, VGS, VDS, VSB).
Save actual coordinate values for every axis, sorted in ascending order.
Each operating-point parameter array must have shape (nL, nVGS, nVDS, nVSB).

Store NMOS and PMOS separately. Keep different corners, temperatures, and reference geometries in separate datasets.

## 4. Define bias and sign conventions

For NMOS, voltage axes represent VGS, VDS, and VSB.
For PMOS, use positive magnitudes VSG, VSD, and VBS, while retaining field names VGS, VDS, and VSB for interface compatibility.

Preserve original OP values and signs. Define an explicit raw-to-LUT sign mapping and verify ID, GM, GDS, GMB, and capacitance definitions at a known bias point.
Use the positive normal-operating-direction magnitude of ID for design queries.
Do not apply abs indiscriminately, especially to capacitance-matrix entries.

## 5. Record geometry and simulation conditions

Metadata must include device cell/model names, PDK/model revision, model section, corner, temperature_C, Wtotal_um, Wfinger_um, nf, multi, mismatch settings, geometry-field definitions, and sign mapping.

Example reference configuration: Wtotal=10 um, nf=10, multi=1, Wfinger=1 um.
Verify the actual configuration against the generated netlist and PDK definitions. Do not count nf twice.

ID, GM, capacitances, and other quantities describe the reference device.
Use verified Wtotal for ID/W and Cgg/W.
Label linear scaling to other geometries as an approximation requiring verification with the actual PCell.

## 6. Preserve parameters

Store all exported OP parameters, including ID, GM, GDS, GMB, VT, VDSAT, and capacitances. Do not retain only gm/ID or plotted curves.
Explicitly mark unavailable parameters as missing; do not invent them.

## 7. Audit data

Check unique coordinates, expected point counts, missing points, duplicates, NaNs, infinities, unit consistency, and consistency of geometry and simulation conditions.
Map rows into arrays using all four coordinates; do not reshape solely according to CSV row order.
Report anomalies instead of silently deleting or filling points.

## 8. Provide queries and plots

Support:
- Parameter queries by L, VGS, VDS, and VSB.
- VGS inversion from gm/ID, L, VDS, and VSB.
- ID/W, Cgg/W, gm/gds, and other queries at that operating point.
- On-demand plots with specified axes, fixed biases, and grouping variables.

Do not silently extrapolate.
If gm/ID inversion has multiple solutions, report them and request an operating-region selection.
Use explicit, configurable thresholds for near-zero denominators and document filtering without deleting raw data.

## 9. Validate and deliver

First verify queries at original grid points. Then validate interpolation against independent off-grid Spectre results.
Report errors separately for each parameter, with the tested domain and low-current filtering criteria.
Agreement at original grid points alone does not validate interpolation.

Deliver raw data, cleaned data, LUT files, metadata, build scripts, query/plot scripts, a data-audit report, and an interpolation-validation report.

This public document contains generic processing requirements only. Device-specific metadata and simulation datasets are separate local artifacts.

