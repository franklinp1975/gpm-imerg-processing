# IMERG Precipitation Data Processing Script

## Overview

This repository contains an optimized R script for processing 0.25° GPM IMERG Late Precipitation data. The script is designed to handle data generated by the Precipitation Processing System (PPS) and utilizes the Integrated Multi-satEllite Retrievals for GPM (IMERG) algorithm version 3IMERGH 7.0.

## Repository Name

**IMERG-Precipitation-Data-Processor**

## Description

This repository provides a comprehensive R script for processing precipitation data from NASA's Global Precipitation Measurement (GPM) mission. The script includes functions for data cleaning, spatial analysis, and visualization, making it a valuable tool for researchers and meteorologists working with precipitation data.

## License

This repository is licensed under the [MIT License](LICENSE). Feel free to use, modify, and distribute the code as per the terms of the license.

## Installation

To use this script, you need to have R installed on your system. Additionally, the script requires the following R packages:

- `terra`
- `sf`
- `dplyr`
- `stringr`

You can install these packages using the following command:

```R
install.packages(c("terra", "sf", "dplyr", "stringr"))
```

## Usage

1. **Clone the Repository:**

   ```bash
   git clone https://github.com/yourusername/IMERG-Precipitation-Data-Processor.git
   ```

2. **Navigate to the Repository:**

   ```bash
   cd IMERG-Precipitation-Data-Processor
   ```

3. **Run the Script:**

   Open the script in RStudio or your preferred R environment and execute it.

## Data Download Instructions

Before downloading data files from the Precipitation Processing System (PPS), users must register their email address with PPS by visiting [http://registration.pps.eosdis.nasa.gov/](http://registration.pps.eosdis.nasa.gov/). After registering an email address, users can download IMERG GIS files from the PPS FTP site using FileZilla with the following parameters:

- **Tool:** FileZilla ([source: https://filezilla-project.org/](https://filezilla-project.org/))
- **Product:** IMERG
- **Provider:** NASA’s Global Precipitation Measurement Mission (GPM)
- **Remote Host:** `jsimpsonftps.pps.eosdis.nasa.gov`
- **Username:** `your.registered.email@gmail.com`
- **Password:** `your.registered.email@gmail.com`
- **Local site:** `C:\ONCC_IMERG\Input\`
- **Remote site:** `/data/imerg/gis/05`

## Script Sections

### 1. Package Loading

The script loads the required R packages:

```R
required_packages <- c("terra", "sf", "dplyr", "stringr")
invisible(lapply(required_packages, library, character.only = TRUE))
```

### 2. Configuration

Define main directories and configuration parameters:

```R
dir_config <- list(
  main = normalizePath("C:/ONCC_IMERG"),
  aoi = file.path("C:/ONCC_IMERG", "AOI"),
  input = file.path("C:/ONCC_IMERG", "Input"),
  raw_input = file.path("C:/ONCC_IMERG", "RawInput"),
  output = file.path("C:/ONCC_IMERG", "Outcome"),
  pattern = file.path("C:/ONCC_IMERG", "Pattern", "pattern_1km.tif")
)

scaling_factor <- 0.1
missing_data <- 29999
```

### 3. Processing Functions

Define functions for cleaning directories, loading AOI geojson files, processing raster files, and deleting files and folders:

```R
clean_directory <- function(path, patterns) {
  list.files(path, pattern = paste(patterns, collapse = "|"), full.names = TRUE) |>
    file.remove()
}

load_aoi <- function(pattern, dir_path = dir_config$aoi) {
  list.files(dir_path, pattern, full.names = TRUE) |>
    st_read(quiet = TRUE) |>
    vect()
}

process_raster <- function(file_path, template, aoi) {
  rast(file_path) |>
    resample(template, method = "bilinear") |>
    mask(aoi)
}

deleteFilesAndFolders <- function(folder) {
  if (!dir.exists(folder)) {
    stop("Folder does not exist")
  }
  all_items <- list.files(folder, full.names = TRUE)
  is_directory <- dir.exists(all_items)
  files <- all_items[!is_directory]
  directories <- all_items[is_directory]
  if (length(files) > 0) {
    file.remove(files)
  }
  if (length(directories) > 0) {
    unlink(directories, recursive = TRUE)
  }
  paste("All files and folders in", folder, "have been deleted")
}
```

### 4. Data Preparation

Load spatial data and define regions for processing:

```R
template_raster <- rast(dir_config$pattern)
venezuela_aoi <- load_aoi("Venezuela")

regions <- list(
  Cojedes = load_aoi("Cojedes"),
  Guarico = load_aoi("Guarico"),
  Venezuela = load_aoi("Venezuela")
)
```

### 5. Data Processing Pipeline

Move and process the tif files from the raw data folder to the input folder:

```R
repository <- list.files(path = dir_config$raw_input, pattern = "3B-HHR.*\\.30min.tif$", full.names = TRUE)
base::file.copy(from = repository, to = dir_config$input, overwrite = TRUE, copy.mode = TRUE, copy.date = FALSE)

imerg_files <- list.files(dir_config$input, full.names = TRUE)
F1 <- unlist(strsplit(basename(imerg_files[1]), ".", fixed = TRUE))
F2 <- unlist(strsplit(F1[5], "-", fixed = TRUE))

date_str <- F2[1]
utc_offset <- -4
time_step <- 0.5

base_date <- as.POSIXct(date_str, format = "%Y%m%d", tz = "UTC")
time_points <- seq(0, 23.5, by = time_step)
utc_datetimes <- base_date + time_points * 3600
local_datetimes <- utc_datetimes + utc_offset * 3600

tag <- paste0(
  format(local_datetimes, "S%H%M00."),
  format(local_datetimes, "%Y%m%d")
)

a <- length(imerg_files)
for (i in 1:a) {
  r <- process_raster(file_path = imerg_files[i], template = template_raster, aoi = regions$Venezuela)
  r[r == missing_data] <- NA
  rc <- r * scaling_factor * time_step
  fname_base <- unlist(strsplit(names(r), ".", fixed = TRUE))
  fname <- paste(c(fname_base[4], tag[i], "tif"), collapse = ".")
  writeRaster(rc, file.path(dir_config$output, fname), overwrite = TRUE)
}
```

### 6. Clean the Input Folder

```R
deleteFilesAndFolders(dir_config$raw_input)
deleteFilesAndFolders(dir_config$input)
```

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request with your changes.


## E-video 
Guidelines: https://goo.su/aHicbG

## Contact

For any questions or feedback, please contact [franklinparedes75@gmail.com](franklinparedes75@gmail.com).

