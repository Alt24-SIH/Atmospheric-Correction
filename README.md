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

## Installation
To get started, clone this repository and install the necessary dependencies.

```bash
git clone https://github.com/Alt24-SIH/Atmospheric-Correction.git
cd hyperspectral-atmospheric-correction
pip install -r requirements.txt
```
## Usage
1. Load Hyperspectral Image Data
The radiance data and corresponding header files are loaded using custom functions.

```python
radiance_file = "path_to_your_radiance_file"
hdr_file = "path_to_your_hdr_file"
radiance_data = read_radiance_file(radiance_file, hdr_file)
```
2. Atmospheric Correction
Atmospheric correction is performed by calculating the corrected reflectance values for each band of the image.

```python
corrected_reflectance = np.zeros_like(radiance_data, dtype=np.float32)

for b in range(bands):
    if b < len(L_path) and b < len(E_sun) and b < len(T_total):
        if not E_sun[b] * T_total[b] == 0:
            corrected_reflectance[b, :, :] = (np.pi * (radiance_data[b, :, :] - L_path[b])) / (E_sun[b] * T_total[b])
        else:
            corrected_reflectance[b, :, :] = (np.pi * (radiance_data[b, :, :] - L_path[b]))

corrected_reflectance = np.clip(corrected_reflectance, 0, 1)
```
3. Visualizing Corrected Images
The corrected images can be visualized either as RGB composites or single-band images.

RGB Composite:

```python
rgb_bands = [30, 20, 10]  # Example bands for RGB
rgb_corrected_image = get_rgb_composite(corrected_image_normalized, rgb_bands)

plt.figure(figsize=(10, 10))
plt.imshow(rgb_corrected_image)
plt.title('Corrected RGB Composite Image')
plt.axis('off')
plt.show()
```
Single Band:

```python
band_number = 30  # Example band number

corrected_band_image = corrected_image[band_number, :, :]
corrected_band_image_normalized = (corrected_band_image - corrected_band_image.min()) / (corrected_band_image.max() - corrected_band_image.min())

plt.figure(figsize=(10, 10))
plt.imshow(corrected_band_image_normalized, cmap='gray')
plt.colorbar()
plt.title(f'Corrected Reflectance - Band {band_number}')
plt.axis('off')
plt.show()
```
## Methodology
Reading Hyperspectral Data: The read_radiance_file function reads binary hyperspectral image data based on header information.

Performing Atmospheric Correction: Using values from atmospheric correction calculations (L_path, E_sun, T_total), the code calculates reflectance values for each band.

Visualization: The corrected image data is visualized using matplotlib, allowing for analysis of how the atmospheric correction alters the image.

## Examples
To see this code in action, run the provided example scripts in the examples folder. These scripts demonstrate loading data, performing corrections, and visualizing results.

## Contributing
Contributions are welcome! Please feel free to submit a Pull Request.

## License
This project is licensed under the DJSCE, Mumbai. All rights are reserved. Reproduction, distribution, or publication of this repository is permitted only with the explicit consent of the authorized members.
