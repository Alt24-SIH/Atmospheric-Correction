# Hyperspectral Image Atmospheric Correction
This project performs atmospheric correction on hyperspectral images, transforming radiance values to reflectance values. The corrected reflectance values are then applied to visualize the corrected image.

## Table of Contents
- [Introduction](#introduction)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Methodology](#methodology)
- [Examples](#examples)
- [Contributing](#contributing)
- [License](#license)

## Introduction
Atmospheric correction is a crucial step in remote sensing that adjusts the data collected from satellites or airborne sensors to account for atmospheric effects. This project demonstrates how to:

- Load hyperspectral image data.
- Apply atmospheric correction to convert radiance values to reflectance.
- Visualize corrected reflectance data as images.

## Requirements
Before running the code, ensure you have the following installed:

- Python 3.7 or later
- NumPy
- Matplotlib
- Spectral Python (spectral)
- GDAL
- Rasterio

## Installation
To get started, clone this repository and install the necessary dependencies.

```bash
git clone https://github.com/Alt24-SIH/Atmospheric-Correction.git
cd hyperspectral-atmospheric-correction
pip install spectral
pip install gdal
pip install pyopengl
pip install rasterio
pip install matplotlib numpy
```
## Usage
1. Load Hyperspectral Image Data
The radiance data and corresponding header files are loaded using custom functions.

```python
img = spectral.envi.open(r"D:\SIH stuff\f100506t01p00r07rdn_b\f100506t01p00r07rdn_b_sc01_ort_img.hdr", r"D:\SIH stuff\f100506t01p00r07rdn_b\f100506t01p00r07rdn_b_sc01_ort_img")
img_data = img.load() 
```
2. Atmospheric Correction
Atmospheric correction is performed by calculating the corrected reflectance values for each band of the image.

```python
for index, wavelength in enumerate(wavelengths):
    band_data = img_data[: ,: , index]
    
    params = atmos_params[wavelength/1000]
    path_radiance = params['atmospheric_intrinsic_radiance']
    transmittance_up = params['transmittance_up']
    transmittance_down = params['transmittance_down']
    diffuse_solar_irradiance = params['diffuse_solar_irradiance']
    direct_solar_irradiance = params['direct_solar_irradiance']

    corrected_band = (np.pi * (band_data - path_radiance) * d**2) / (transmittance_up * (direct_solar_irradiance * np.cos(solar_zenith) * transmittance_down + diffuse_solar_irradiance))
    corrected_reflectance[:, :, index] = corrected_band
```

3. Storing the corrected image usin rasterio

```python
output_path = r"D:\SIH stuff\f100506t01p00r07rdn_b\outputs\atmos_correct_1.tif"
corrected_reflectance = corrected_reflectance.astype(np.float32)

with rasterio.open(
    output_path,
    'w',
    driver='GTiff',
    height=corrected_reflectance.shape[1],
    width=corrected_reflectance.shape[2],
    count=corrected_reflectance.shape[0],
    dtype=corrected_reflectance.dtype,
    crs='EPSG:4326'
) as dst:
    for band_index in range(corrected_reflectance.shape[0]):
        dst.write(corrected_reflectance[band_index, :, :], band_index + 1)
```

4. Visualizing Corrected Images
The corrected images can be visualized either as RGB composites or single-band images.

RGB Composite:

```python
with rasterio.open(output_path) as src:
    red_band = src.read(30)
    green_band = src.read(19)
    blue_band = src.read(11)

op_rgb_image = np.dstack((red_band, green_band, blue_band))

op_rgb_image = op_rgb_image.astype(np.float32)
op_rgb_image = (op_rgb_image - op_rgb_image.min()) / (op_rgb_image.max() - op_rgb_image.min())
plt.subplot(1, 2, 2)
plt.imshow(op_rgb_image)
plt.title("RGB Equivalent of Corrected Reflectance")
plt.axis('off')
```

## Methodology
Reading Hyperspectral Data: The spectral.envi.open function reads hyperspectral image data based on header information.

Performing Atmospheric Correction: Using values from atmospheric correction calculations (L_path, E_sun, T_up, T_down, E_diffuse), the code calculates reflectance values for each band.

Visualization: The corrected image data is visualized using matplotlib, allowing for analysis of how the atmospheric correction alters the image.

## Examples
To see this code in action, run the provided example scripts in the examples folder. These scripts demonstrate loading data, performing corrections, and visualizing results.

## Contributing
Contributions are welcome! Please feel free to submit a Pull Request.

## License
This project is licensed under the DJSCE, Mumbai. All rights are reserved. Reproduction, distribution, or publication of this repository is permitted only with the explicit consent of the authorized members.
