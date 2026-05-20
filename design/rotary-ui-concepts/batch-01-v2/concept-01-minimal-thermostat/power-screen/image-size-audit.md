# Image Size Audit

This document records the dimensions and file sizes of all 24 generated concept images.

| Option | Model | Dimensions | File Size |
| :--- | :--- | :--- | :--- |
| 01: Glowing Power | default | 1248x1248 | 932K |
| 01: Glowing Power | gpt-image-2 | 1248x1248 | 952K |
| 01: Glowing Power | nano-banana-2 | 2048x2048 | 3.6M |
| 01: Glowing Power | nano-banana-pro | 2048x2048 | 3.5M |
| 02: Thin-Line | default | 1248x1248 | 444K |
| 02: Thin-Line | gpt-image-2 | 1248x1248 | 432K |
| 02: Thin-Line | nano-banana-2 | 2048x2048 | 3.7M |
| 02: Thin-Line | nano-banana-pro | 2048x2048 | 3.7M |
| 03: Pulsing Orb | default | 1248x1248 | 1020K |
| 03: Pulsing Orb | gpt-image-2 | 1248x1248 | 816K |
| 03: Pulsing Orb | nano-banana-2 | 2048x2048 | 3.6M |
| 03: Pulsing Orb | nano-banana-pro | 2048x2048 | 3.5M |
| 04: Iris Aperture | default | 1248x1248 | 1.3M |
| 04: Iris Aperture | gpt-image-2 | 1248x1248 | 1.5M |
| 04: Iris Aperture | nano-banana-2 | 2048x2048 | 4.6M |
| 04: Iris Aperture | nano-banana-pro | 2048x2048 | 4.5M |
| 05: Eclipse Corona | default | 1248x1248 | 1.1M |
| 05: Eclipse Corona | gpt-image-2 | 1248x1248 | 1.4M |
| 05: Eclipse Corona | nano-banana-2 | 2048x2048 | 4.3M |
| 05: Eclipse Corona | nano-banana-pro | 2048x2048 | 4.3M |
| 06: Radar Sweep | default | 1248x1248 | 1.4M |
| 06: Radar Sweep | gpt-image-2 | 1248x1248 | 1.3M |
| 06: Radar Sweep | nano-banana-2 | 2048x2048 | 4.2M |
| 06: Radar Sweep | nano-banana-pro | 2048x2048 | 3.3M |

## Observations
- The `nano-banana` models consistently output at 2048x2048 resolution, resulting in file sizes between 3.3MB and 4.6MB.
- The `gpt-image-2` and `default` models output at 1248x1248 resolution, with file sizes generally under 1.5MB.
- Option 02 (Thin-Line) has the smallest file sizes across the 1248x1248 models due to its highly minimal design with large areas of solid black.
