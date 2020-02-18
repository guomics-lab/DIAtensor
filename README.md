# DIA Tensor
DIAtensor is software tool to convert mzXML of data-independent acquisition (DIA) mass spectrometry (MS) to DIA tensor.
This tool is based on Python, which can read mzXML mass spectrometry data, check the integrity of the data, pool the mass-to-charge ratio and retention time, and perform frequency division bin coloring such as intensity values.
With DIA Tensor, a classification deep learning model without the need for a large amount of extraction library chromatographic peaks to identify peptides.

### Download: [https://github.com/](https://github.com/)
DIAtensor manual: [https://github.com/](https://github.com/)
Please also check the commands listed below.

### Publication
[bioRxiv.org](http://bioRxiv.org)

### System requirement
Operating System: Windows 10
Minimal RAM: 16GB 

### Run
1. Open the Windows console CMD
1. Change the directory to the path with DIAtensor.exe
1. Run the DIAtensor.exe with parameters

### Input files
Raw data files: .mzXML
mzXML file convertion: the Peak Picking operation should be checked in the Msconvert software with MS Levels parameter should be set to 1-2.

### Output files
#### File type

1. images (*.png / *.bmp)

dimension _y_ : mz index (from low to high)
dimension _x_ : cycle index (from low to high)

2. tensors (*.diat)

dtype : uint8

dimension _z_ : window index (from low to high)
dimension _y_ : mz index (from low to high)
dimension _x_ : cycle index (from low to high)
dimension _color_ : discretized non-zero intensity value mapping to 1~255

#### File naming

1. sampleId
```
[sampleId]
```

D20181207yix_HCC_SW_T_41A-HCC_yix_9

2. window
```
_win_[number](_[number])
```

MS2 win 20: _win_20
MS2 win 1~66:  win_1_66

3. gradient time
```
_grad_[start_minute]_[end_minute]
```
Effective gradient from 4 to 48 minutes:  _grad_4_48

4. cycle
```
_cycle_[number]
```

Align with 1400: _cycle_1400
No alignment in cycles: _cycle_full

5. m/z range
```
_mz_[number]_[number]
```
_mz_400_1500

6. m/z bin & pool
```
_bin_[number]
```
Binning m / z with 0.5Da accuracy: _bin_0.5
Binning m / z with 0.01 Da and main and auxiliary peak methods: _bin_peaks

7. RT pool
```
_[pool method]_[pool width]_[pool height]
```
4 x 1 regional maximum pooling: _max_rt_4_1

8. color scheme
```
_[color](_[color])_[number]
```

Black to white 255 gradient: _Gray_255
Green to red 255 gradient: _Gr_Re_255

### Command-line tool usage

```
DIAtensor.exe [commands]
```

Commands can be supplied in arbitrary order.

#### Required commands:

```
-i / --input <path>
```

Input address of the mzXML raw file.

```r
-o / --output <path>
```

Output address of the DIAtensor result file.

```
-T / --out_type <’image’/’tensor’>
```

Output type of the DIAtensor result file: image or tensor.

#### Auxiliary commands:

```
-t / --threads <int>
```

Number of threads executing at the same time. default is 1.

```
--min_mz <int>
```

Minimum value of ms2 m / z. The default value is 400.

```
--max_mz <int>
```

Maximum value of ms2 m / z. The default value is 1500.

```
-g / --grad <float> <float>
```

Effective gradient time of the dataset.

```
-a / --auto_cycle
```

When this parameter is used, the cycle number alignment value is automatically calculated and aligned on the data set. This parameter is mutually exclusive with manual_cycle.

```
-c / --manual_cycle <int>
```

This parameter records and aligns the manually selected data set cycle number. This parameter is mutually exclusive with auto_cycle.

```
-b / --bin <float>
```

This parameter records the binning accuracy of m / z (unit is Da), the default is 0.5.

```
--pool_mz
```

When this parameter is used and the binning accuracy parameter bin is 0.01 and the minimum ms2 m / z parameter min_mz is 400, pooling of the main and auxiliary peaks in the m / z dimension will be performed.

```
--pool_rt
```

When this parameter is used, max pooling on the RT dimension (4x1) will be performed.

**Commands below only available if output type is ‘image’:**

```
-D / --dimension <'3D'/'2D'>
```

This parameter determines the output image YN sub-window (3D means that the sub-window is out of the picture, 2D means that the windows are arranged in a big picture in order), the default is the sub-window(3D).

```
-C / --color <'view'/'gray'>
```

This parameter determines whether the output image is a visual image (view) or a grayscale image (gray). The default is a visual image (view).

```
-F / --format <'png'/'bmp'>
```

This parameter determines whether the output image is a png or a bmp.The default is png.

#### Example:

##### Generate tensor (save *.diat)

1. Generation of a tensor by 1400 cycle with pooling in mz dimensions.

```
DIAtensor.exe -i E: /HCCSW -o E: /tensor -T tensor -c 1400 -b 0.01 --pool_mz
```

2. Generation of a tensor by the upper limit of m/z to 2000 Da, auto aligned cycle based on the gradient of 0-45 minutes,with pooling in mz dimensions.

```
DIAtensor.exe -i E: /HCCSW -o E: /tensor -T tensor --max_mz 2000 -a -g 0 45 -b 0.01 --pool_mz
```

##### Read tensor (read *.diat)

Replace <file_path> with your own file path, and run following codes in Python 3.6+ environment with Numpy 1.16.2+ installed.

```
import numpy as np
diat = np.load("<file_path>.diat")
```

##### Generate image (save *.png/*.bmp)

1. Generation of an aligned image by 1400 cycle with pooling in both the m/z and RT dimensions.

```
DIAtensor.exe -i E: /HCCSW -o E: /img -T image -c 1400 -b 0.01 --pool_mz --pool_rt -D 2D -C view -F png
```

2. Generation of a set of aligned images by 1000 cycles and m/z and RT dimensions without pooling, binning with 0.5 Da accuracy.

```
DIAtensor.exe -i E: /HCCSW -o E: /img -T image -c 1000 -b 0.5 -D 3D -C view -F png
```

3. Generation of a set of aligned images by the upper limit of m/z to 2000 Da, aligned with the 1130 cycle based on the gradient of 4-48 minutes, and perform the main and auxiliary peak pooling in the m/z dimension. The RT dimension does not do pooling, and the windowed TPD dataset grayscale bmp image.

```
DIAtensor.exe -i E: /TPD_4_48 -o E: /img -T image --max_mz 2000 -g 4 48 -c 1130 -b 0.01 --pool_mz -D 3D -C gray -F bmp
```

#### Note:

1. The input and output addresses E: / HCCSW and E: / img of the sample data set need to be modified to the folder addresses in the user's computer.
1. When the DIAtensor software is running, the single-process operation requires no less than 16G of memory. It is recommended that 16G memory computers use the default threads 1, 32G memory computers set threads to 2, and other memory sizes and so on.
1. Where there is any difference between the software instruction and the description provided by the software help parameter (-h), one should always follow the manual instruction online.

### 

