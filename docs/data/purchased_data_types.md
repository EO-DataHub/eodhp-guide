# Purchased Data Types

## Overview

This document describes the file naming conventions, formats, and structure for purchased satellite imagery data from Airbus and Planet. The information is based on sample data.

## Airbus

### Product Bundles

Airbus imagery is available in several product bundles:

- **Visual** - Optimized for visual interpretation
- **General Use** - Standard processing level
- **Basic** - Basic processing level
- **Analytic** - Science-grade processing with radiometric calibration

### PHR (Pléiades High Resolution)

[Airbus Pléiades Page](https://space-solutions.airbus.com/imagery/our-optical-and-radar-satellite-imagery/pleiades/)

#### PHR File Naming Convention

**Format:** `{File Prefix}_{Satellite Identifier}_{Product Type}_{Acquisition Timestamp}_{Processing Level}_{Segment Identifier}_{Tile Identifier}.{File Extension}`

**Example:** `IMG_PHR1A_MS_201805011120113_ORT_7331857101-2_R1C1.JP2`

**Field Descriptions:**

- **File Prefix** - Prefix indicating file type
  - `IMG` = Full-resolution raster (.JP2/.TIF)
  - `ICON` = Low-resolution preview/quicklook (.JPG)

- **Satellite Identifier** - Satellite designation
  - `PHR1A` = Pléiades-1A
  - `PHR1B` = Pléiades-1B

- **Product Type** - Spectral content / product type
  - `P` = Panchromatic only
  - `MS` = Multispectral only (4 bands, 2 m resolution)
  - `PMS` = Pan + Multispectral bundle
  - `PMS-N` = Pansharpened natural colour (RGB at 0.5 m)
  - Other variants exist (e.g., `PMS-X`)

- **Acquisition Timestamp** - Start of scene acquisition time (UTC)
  - Format: `YYYYMMDDHHMMSS` + one digit for tenths of a second
  - Example: `201805011120113` → 2018-05-01 11:20:11.3 UTC

- **Processing Level** - Processing and correction level
  - `ORT` = Orthorectified (terrain-corrected)
  - `SEN` or `PRI` = Sensor/Primary (no terrain correction)
  - `ORR` = Orthorectified reflectance (atmospherically corrected)

- **Segment Identifier** - Unique segment identifier (Airbus internal job ID)
  - Format: `{10-digit-order-number}-{delivery-attempt-number}`
  - First 10 digits = order/segment number
  - Digit after "-" = delivery attempt number (usually 1, rarely higher)

- **Tile Identifier** - Tile numbering (only present when scene is split into tiles)
  - Format: `R{row}C{column}` (e.g., `R1C1` = top-left tile)
  - Omitted when the whole scene is delivered as one file

- **File Extension** - File format
  - `.JP2` or `.TIF` = Full-resolution imagery
  - `.JPG` = Preview only

#### PHR File Formats

- **MS, P, PMS products** - Delivered as JP2/J2W files (Compressed JPEG2000)
- **PMS-N products** - Delivered as TIF/TFW files (Uncompressed or LZW-compressed GeoTIFF)

**Note:** J2W and TFW are world files used for georeferencing. They are not needed in this case as the projection information is already embedded in the JP2/TIF files.

---

### PNEO (Pléiades Neo)

[Airbus Pléiades Neo page](https://space-solutions.airbus.com/imagery/our-optical-and-radar-satellite-imagery/pleiades-neo/)

#### PNEO File Naming Convention

**Format:** `{File Prefix}_{Satellite Identifier}_{Acquisition Timestamp}_{Product Type}_{Processing Level}_{Product Code}_{Segment Identifier}_{Version Counters}_{Bit Depth Flag}_{Band Composition}_{Tile Identifier}.{File Extension}`

**Example:** `IMG_PNEO3_202209171103597_PMS-N_ORT_PWOI_000317842_1_1_F_1_RGB_R1C1.TIF`

**Field Descriptions:**

- **File Prefix** - Prefix indicating file type
  - `IMG` = Full-resolution imagery
  - `ICON` = Preview/quicklook

- **Satellite Identifier** - Satellite designation (short code)
  - `PNEO3` = Pléiades Neo 3
  - `PNEO4` = Pléiades Neo 4
  - `PNEO5` = Pléiades Neo 5
  - `PNEO6` = Pléiades Neo 6

- **Acquisition Timestamp** - Start of scene acquisition time (UTC)
  - Format: `YYYYMMDDHHMMSS` + one digit for tenths of a second
  - Example: `202209171103597` → 2022-09-17 11:03:59.7 UTC

- **Product Type** - Spectral content / product type
  - `PAN` = Panchromatic only (30 cm resolution)
  - `MS` = Multispectral only (6 bands, 1.2 m resolution)
  - `MS-FS` = Multispectral Full Scene (or sometimes just MS)
  - `PMS` = Pan + Multispectral bundle
  - `PMS-N` = Pansharpened natural-colour RGB (30 cm)
  - `PMS-FS` = Pansharpened Full Scene

- **Processing Level** - Processing and correction level
  - `ORT` = Orthorectified
  - `SEN` = Sensor level

- **Product Code** - Fixed code for "Project World Imagery" or internal product code
  - Example: `PWOI`

- **Segment Identifier** - Unique tasking / segment ID
  - Format: 9 or 10 digits
  - Airbus internal order number

- **Version Counters** - Version/delivery counters
  - Format: `{counter1}_{counter2}` (e.g., `1_1`)
  - Rarely changes

- **Bit Depth Flag** - Bit depth / compression flag
  - Format: `{flag}_{number}` (e.g., `F_1`)
  - `F` = 12-bit stored as 16-bit (Full resolution)
  - `R` = 8-bit reduced (rare)

- **Band Composition** - Band composition of this specific tile
  - `RGB` = Natural colour composite
  - `P` = Panchromatic (single band)
  - `NED` = Normalised Elevation Difference (single-band height difference product)

- **Tile Identifier** - Tile numbering (when scene is tiled)
  - Format: `R{row}C{column}` (e.g., `R1C1` = top-left tile)
  - Omitted on single-tile deliveries

- **File Extension** - File format
  - `.TIF` = GeoTIFF

#### PNEO File Formats

All PNEO products are delivered as GeoTIFF files.

---

### SPOT

[Airbus SPOT page](https://space-solutions.airbus.com/imagery/our-optical-and-radar-satellite-imagery/spot/)

#### SPOT File Naming Convention

**Format:** `{File Prefix}_{Satellite Identifier}_{Product Type}_{Acquisition Timestamp}_{Processing Level}_{Segment Identifier}_{Tile Identifier}.{File Extension}`

**Example:** `IMG_SPOT7_MS_201909211046032_ORT_7331860101_R1C2.TIF`

**Field Descriptions:**

- **File Prefix** - Prefix indicating file type
  - `IMG` = Full-resolution imagery
  - `ICON` = Preview/quicklook

- **Satellite Identifier** - Satellite designation
  - `SPOT6` = SPOT 6
  - `SPOT7` = SPOT 7
  - (SPOT6 and SPOT7 have identical products)

- **Product Type** - Spectral content / product type
  - `P` = Panchromatic only (1.5 m resolution)
  - `MS` = Multispectral only (4 bands B, G, R, NIR – 6 m resolution)
  - `PMS` = Pan + Multispectral bundle
  - `PMS-N` = Pansharpened natural-colour RGB (1.5 m)

- **Acquisition Timestamp** - Start of scene acquisition time (UTC)
  - Format: `YYYYMMDDHHMMSS` (no tenths-of-a-second digit on SPOT)
  - Example: `201909211046032` → 2019-09-21 10:46:03 UTC

- **Processing Level** - Processing and correction level
  - `ORT` = Orthorectified (terrain-corrected)
  - `SEN` = Sensor/Primary (rarely delivered)

- **Segment Identifier** - Unique segment / order identifier
  - Format: 10-digit order number

- **Tile Identifier** - Tile numbering (when scene is delivered in tiles)
  - Format: `R{row}C{column}` (e.g., `R1C1` = top-left tile)
  - Omitted when delivered as a single file

- **File Extension** - File format
  - `.TIF` = GeoTIFF

#### SPOT File Formats

All SPOT products are delivered as GeoTIFF files.

---

## Planet

### Planetscope

#### Planet Basemaps

Planet Basemaps are monthly or quarterly mosaics providing surface-reflectance, cloud-free, visually beautiful tiles.

[Planet Basemaps Product Page](https://www.planet.com/products/basemap/)

**Typical File Path Structure:**

`{directory_structure}/{product_line}_{processing_type}_{delivery_method}_{mosaic_period}_mosaic/{tile_identifier}_{suffix}.{file_extension}`

**Example:** `monthly_basemaps/may_2024_basemap_normalized_sr/0971529b-c3e8-4d9d-865f-ff290b624a39/ps_monthly_sen2_normalized_analytic_8b_sr_subscription_2024_05_mosaic/1000-1407_ortho_udm2_clip.tif`

**Path Component Descriptions:**

- **Product Line** - Product line identifier
  - `ps_monthly_sen2_` = PlanetScope Monthly Basemap (Sen2 = harmonised to Sentinel-2 grid/bands)

- **Processing Type** - Processing and correction type
  - `normalized` = Atmospherically corrected + BRDF normalised
  - `analytic` = Science-grade (not just visual)
  - `8b` = 8-band (B1 Coastal, B2 Blue, B3 Green, B4 Red, B5 RE1, B6 RE2, B7 RE3, B8 NIR)
  - `sr` = Surface reflectance (values 0–10000, multiply by 0.0001 for true reflectance)

- **Delivery Method** - Method of delivery
  - `subscription` = Delivered via Planet Basemaps subscription

- **Mosaic Period** - Time period covered by the mosaic
  - Format: `YYYY_MM` (year and month)
  - Example: `2024_05` = May 2024

- **Mosaic Indicator** - Indicates this is a seamless mosaic
  - `mosaic` = Seamless mosaic (not a single scene)

- **Tile Identifier** - Tile identifier on Planet's global UTM grid
  - Format: `{row}-{tile}` (e.g., `1000-1407`)
  - First four digits = UTM zone "row"
  - Last four digits = tile within that row
  - Multiple adjacent tiles may be delivered (e.g., `1000-1407`, `1000-1408`, `1000-1409`)

**Common File Suffixes:**

- `…_quad_clip.tif` - The 3.7 m 4-band or 8-band visual mosaic
- `…_ortho_udm2_clip.tif` - Mask to remove clouds/shadow if needed
- `…_metadata_clip.json` - Detailed info about what dates went into the mosaic
- `…_provenance_raster_clip.tif` - Single-band GeoTIFF indicating which original PlanetScope scene contributed the final pixel value

#### Planet Basemaps File Formats

Delivered as GeoTIFF files

#### PlanetScope Individual Scenes

**PlanetScope Individual Scene File Naming Convention:**

**Format:** `{Acquisition Date}_{Acquisition Time}_{Satellite ID}_{Product Level}_{Data Type}.{File Extension}`

**Example:** `20230228_100631_43_2427_1B_AnalyticMS_8b.tif`

**Field Descriptions:**

- **Acquisition Date** - Acquisition date (UTC)
  - Format: `YYYYMMDD`
  - Example: `20230228` → 2023-02-28

- **Acquisition Time** - Acquisition time and hundredths of a second
  - Format: `HHMMSS_ss`
  - Example: `100631_43` → 10:06:31.43 UTC

- **Satellite ID** - Planet's internal satellite identifier
  - Examples: `2427`, `242d`, `227a`, etc.

- **Product Level** - Product processing level
  - `1B` = Orthorectified analytic product (preferred)
  - `1A` = Older basic ortho product (rare now)
  - `3B` = Older-style orthorectified product, usually delivered as multi-band GeoTIFF + separate files

- **Data Type** - Data type and band configuration
  - `AnalyticMS` = Science-grade multispectral
  - `8b` = 8-band (Coastal, Blue, Green I, Yellow, Red, Red-Edge, NIR-1, NIR-2)
  - Older scenes may use `AnalyticMS` = 4-band or `AnalyticMS_SR` = surface reflectance

- **File Extension** - File format
  - `.tif` = Standard GeoTIFF with embedded georeferencing (no world files needed)

**Most Common Files from a Single Scene:**

- `…_1B_AnalyticMS_8b.tif` - Main 8-band analytic image (primary file)
- `…_1A_udm2.tif` - Cloud/shadow mask (overlay to remove bad pixels)
- `…_metadata.xml` or `.json` - Detailed scene info (acquired date/time, cloud cover %, etc.)

#### PlanetScope File Formats

**1B** Orthorectified analytic scene, delivered as standard GeoTIFF
**3B** Older-style orthorectified product, usually delivered as a multi-band GeoTIFF + separate files for orthorectification

### SkySat Collect

Planet's high-resolution constellation (50–72 cm panchromatic + 4-band multispectral).

#### SkySat File Naming Convention

**Format:** `{Acquisition Date}_{Acquisition Time}_{Satellite ID}_{Upload ID}_{Product Type}_{Mask Type}.{File Extension}`

**Example:** `20231015_124731_ssc16_u0001_analytic_udm2.tif`

**Field Descriptions:**

- **Acquisition Date** - Acquisition date (UTC)
  - Format: `YYYYMMDD`
  - Example: `20231015` → 2023-10-15

- **Acquisition Time** - Acquisition time
  - Format: `HHMMSS` (no hundredths in SkySat naming)
  - Example: `124731` → 12:47:31 UTC

- **Satellite ID** - Individual SkySat satellite identifier
  - Format: `ssc{number}` (e.g., `ssc16`, `ssc17`)
  - Range: ssc1 through ssc21+ (individual SkySat satellites)

- **Upload ID** - Upload / product bundle ID
  - Format: `u{number}` (e.g., `u0001`, `u0002`)
  - Planet internal counter
  - Usually `u0001` for standard analytic product
  - Sometimes `u0002` for different processing run of the same collect

- **Product Type** - Main image product type
  - `analytic` = Main image product (4-band pansharpened: Blue, Green, Red, NIR at ~50 cm)
  - Some older deliveries are PAN-only or separate Pan + MS; almost all new ones are 4-band analytic

- **Mask Type** - Usable data mask type
  - `udm` = Old usable data mask (legacy cloud/shadow mask, single-band)
  - `udm2` = Current Usable Data Mask 2 (preferred, multi-class mask)
    - 0 = null
    - 1 = clear
    - 2–6 = cloud/haze/shadow classes

- **File Extension** - File format
  - `.tif` = GeoTIFF

**Most Common Files from a SkySat Scene:**

- `…_analytic.tif` - The 50 cm 4-band pansharpened image (primary file)
- `…_udm2.tif` - Current cloud/shadow/haze mask
- `…_metadata.json` - All technical details

#### SkySat File Formats

Delivered as GeoTIFF files

## File Format Summary

### Airbus Products

| Product | File Format | Notes |
|---------|-------------|-------|
| PHR (MS, P, PMS) | JP2/J2W | JPEG2000 |
| PHR (PMS-N) | TIF/TFW | GeoTIFF |
| PNEO | TIF | GeoTIFF |
| SPOT | TIF | GeoTIFF |

### Planet Products

| Product | File Format | Notes |
|---------|-------------|-------|
| Planetscope Basemaps | TIF | GeoTIFF |
| Planetscope Individual Scenes (1A and 1B) | TIF | GeoTIFF with embedded georeferencing |
| Planetscope Individual Scenes (3B) | TIF | Multi-band GeoTIFF + separate files for georeferencing |
| SkySat Collect | TIF | GeoTIFF |

---

## References

- [Airbus data format guide](https://content.satimagingcorp.com/media/pdf/User_Guide_Pleiades.pdf)
- [Dimap format](https://step.esa.int/main/wp-content/help/versions/9.0.0/snap/org.esa.snap.snap.help/general/overview/BeamDimapFormat.html)
- [Airbus Pléiades Page](https://space-solutions.airbus.com/imagery/our-optical-and-radar-satellite-imagery/pleiades/)
- [Airbus Pléiades Neo page](https://space-solutions.airbus.com/imagery/our-optical-and-radar-satellite-imagery/pleiades-neo/)
- [Airbus SPOT page](https://space-solutions.airbus.com/imagery/our-optical-and-radar-satellite-imagery/spot/)
- [Planet Basemaps Product Page](https://www.planet.com/products/basemap/)
