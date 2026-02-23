# Chemotaxis Tracking Features Data

## Overview

This repository contains HDF5 feature files generated from *C. elegans* chemotaxis tracking videos using [Tierpsy Tracker](https://github.com/ver228/tierpsy-tracker). The data captures worm navigation behavior in response to odor patches under different conditioning paradigms.

## Experimental Background

### Study Objective
Comparing navigation strategies of *C. elegans* with different conditioning histories in chemotaxis assays.

### Experimental Conditions
| Condition | Description |
|-----------|-------------|
| **Mock** | Control condition (no prior conditioning) |
| **Aversive** | Worms pre-exposed to odour with starvation |
| **Sexual** | Worms pre-exposed to odour with starvation in the presence of mates |

### Assay Setup
- Worms are placed on an agar plate with an odor patch
- Video recordings capture worm locomotion and navigation toward/away from the odor source
- Single-worm tracking mode (`oneworm`) is used for high-resolution behavioral analysis

---

## File Organization

### Directory Structure
```
Results/
├── {experiment_prefix}_{date}_{replicate}_{timestamp}/
│   └── metadata_featuresN_oneworm.hdf5
```

### Naming Convention
Files follow the pattern: `{condition}_{date}_{replicate}_{processing_timestamp}`

---

## HDF5 File Structure

Each `metadata_featuresN_oneworm.hdf5` file contains the following datasets:

### Core Datasets (Present in All Files)

| Dataset | Shape | Description |
|---------|-------|-------------|
| `trajectories_data` | Table | Frame-by-frame tracking data with worm indices, coordinates, and skeleton validity flags |
| `coordinates` | (N, segments, 2) | Smoothed skeleton and contour coordinates for each valid frame |
| `base_coordinates` | (N, 2) | Base/tail coordinates of the worm |
| `base_x` | (N,) | X-coordinate of worm base |
| `base_y` | (N,) | Y-coordinate of worm base |
| `neck_coordinates` | (N, 2) | Neck region coordinates |
| `neck_x` | (N,) | X-coordinate of worm neck |
| `neck_y` | (N,) | Y-coordinate of worm neck |


### Time Series Features (Present in Most Files)

| Dataset | Description |
|---------|-------------|
| `timeseries_data` | Comprehensive behavioral features computed per frame |

#### Key Features in `timeseries_data`:

**Locomotion Metrics:**
- `speed`, `speed_{body_part}` - Movement speed (µm/s), signed by direction
- `angular_velocity`, `angular_velocity_{body_part}` - Rotational speed (°/s)
- `length` - Skeleton length (µm)
- `area` - Body area (µm²)

**Posture Features:**
- `curvature_{body_part}` - Body curvature at specific segments
- `eigen_projection_[1-7]` - Eigenworm coefficients (shape descriptors)
- `width_{body_part}` - Body width at head_base, midbody, tail_base


**Behavioral State Flags:**
- `motion_mode` - Forward (1), paused (0), or backward (-1) locomotion
---

## Data Access

### Python Example

```python
import h5py
import pandas as pd
import numpy as np

# Open file
filepath = "Results/chemotaxis_ave_210825_2_20250722_143547/metadata_featuresN_oneworm.hdf5"

with h5py.File(filepath, 'r') as f:
    # List all datasets
    print("Datasets:", list(f.keys()))
    
    # Load trajectories data as DataFrame
    traj_data = pd.DataFrame(f['trajectories_data'][:])
    
    # Load timeseries features (if present)
    if 'timeseries_data' in f:
        timeseries = pd.DataFrame(f['timeseries_data'][:])
    
    # Load odor patch coordinates
    if 'food_cnt_coord' in f:
        odor_patch = f['food_cnt_coord'][:]
        print(f"Odor patch defined by {len(odor_patch)} boundary points")
    
    # Load skeleton coordinates
    coords = f['coordinates'][:]  # Shape: (n_frames, n_segments, 2)
```

### Key Column Descriptions in `trajectories_data`

| Column | Description |
|--------|-------------|
| `frame_number` | Video frame index |
| `worm_index_joined` | Trajectory ID (consistent across tracking gaps) |
| `coord_x`, `coord_y` | Centroid coordinates (pixels or µm) |
| `timestamp_time` | Real time (seconds) |
| `was_skeletonized` | Whether skeleton was successfully extracted |
| `skeleton_id` | Row index linking to `/coordinates` |

---


## Processing Pipeline

These files were generated using the following Tierpsy Tracker workflow:

1. **Video Compression** - Background subtraction and ROI extraction
2. **Tracking** - Worm detection and trajectory linking
3. **Skeletonization** - Midline and contour extraction
4. **Feature Extraction** - Tierpsy Features (featuresN) computation

For detailed pipeline documentation, see the [Tierpsy Tracker documentation](https://github.com/ver228/tierpsy-tracker/blob/master/docs/OUTPUTS.md).

---

## Units

| Measurement | Unit | Notes |
|-------------|------|-------|
| Coordinates | µm (microns) | If `microns_per_pixel` calibration was applied |
| Speed | µm/s | |
| Angular velocity | °/s | |
| Time | seconds | Based on video frame rate |
| Curvature | 1/µm | |

Check the HDF5 file attributes for `microns_per_pixel` and `expected_fps` to confirm calibration.

---

## References

- Javer, A., et al. (2018). An open-source platform for analyzing and sharing worm-behavior data. *Nature Methods*, 15(9), 645-646. [DOI: 10.1038/s41592-018-0112-1](https://doi.org/10.1038/s41592-018-0112-1)
- [Tierpsy Tracker GitHub Repository](https://github.com/ver228/tierpsy-tracker)
- [Tierpsy Features Documentation](https://github.com/ver228/tierpsy-tracker/blob/master/docs/OUTPUTS.md)

---

## Contact

For questions about this dataset, please contact the lab maintainers.

