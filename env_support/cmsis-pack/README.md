# How to Create CMSIS-Pack



## STEP 1 Update 'lv_conf_cmsis.h'

1. Copy the **lv_conf_template.h** to '**cmsis-pack**' directory

2. Set the macro protector to '1'

```c
...
/* clang-format off */
#if 1 /*Set it to "1" to enable content*/
...
```

remove the misleading guide above this code segment.

```c
/*
 * Copy this file as `lv_conf.h`
 * 1. simply next to the `lvgl` folder
 * 2. or any other places and
 *    - define `LV_CONF_INCLUDE_SIMPLE`
 *    - add the path as include path
 */
```


3. Add including for '**RTE_Components.h**'

```c
#ifndef LV_CONF_H
#define LV_CONF_H

#include <stdint.h>
#include "RTE_Components.h"
...
```

4. Update `LV_STDIO_INCLUDE` and `LV_STRING_INCLUDE`

   ```c
   #define LV_STDLIB_INCLUDE <stdlib.h>
   #define LV_STDIO_INCLUDE  <stdio.h>
   #define LV_STRING_INCLUDE <string.h>
   ```



5. Remove macro definitions for

   - LV_USE_GPU_STM32_DMA2D
   - LV_USE_GPU_NXP_PXP
   - LV_USE_GPU_NXP_VG_LITE
   - LV_USE_GPU_SWM341_DMA2D
   - LV_USE_GPU_GD32_IPA
   - LV_USE_GPU_ARM2D
   - LV_USE_DEMO_WIDGETS
   - LV_USE_DEMO_BENCHMARK
   - LV_USE_IME_PINYIN
   - LV_USE_FILE_EXPLORER

6. Update `LV_LOG_PRINTF` to `1` and `LV_LOG_LEVEL` to `LV_LOG_LEVEL_USER`

7. Update `LV_DEMO_BENCHMARK_RGB565A8` to `1`

8. Set `LV_FONT_MONTSERRAT_12` and `LV_FONT_MONTSERRAT_16` to `1` (So Widgets and Benchmark can be compiled correctly, this is for improving the out of box experience.)

9. Update macro `LV_ATTRIBUTE_MEM_ALIGN` and `LV_ATTRIBUTE_MEM_ALIGN_SIZE`  to force a WORD alignment.
```c
#define LV_ATTRIBUTE_MEM_ALIGN_SIZE     4
#define LV_ATTRIBUTE_MEM_ALIGN          __attribute__((aligned(4)))
```
Make sure `LV_MEM_SIZE` is no less than `(64*1024U)`.



6. Update Theme related macros:

```c
#ifdef RTE_GRAPHICS_LVGL_USE_EXTRA_THEMES
    /*A simple, impressive and very complete theme*/
    #define LV_USE_THEME_DEFAULT 1
    #if LV_USE_THEME_DEFAULT

        /*0: Light mode; 1: Dark mode*/
        #define LV_THEME_DEFAULT_DARK 0

        /*1: Enable grow on press*/
        #define LV_THEME_DEFAULT_GROW 1

        /*Default transition time in [ms]*/
        #define LV_THEME_DEFAULT_TRANSITION_TIME 80
    #endif /*LV_USE_THEME_DEFAULT*/

    /*A very simple theme that is a good starting point for a custom theme*/
    #define LV_USE_THEME_BASIC 1

    /*A theme designed for monochrome displays*/
    #define LV_USE_THEME_MONO 1
#else
    #define LV_USE_THEME_DEFAULT    0
    #define LV_USE_THEME_BASIC      0
    #define LV_USE_THEME_MONO       0
#endif
```
7. Update `LV_TICK_CUSTOM` related macros:
```c
/*Use a custom tick source that tells the elapsed time in milliseconds.
 *It removes the need to manually update the tick with `lv_tick_inc()`)*/
#ifdef __PERF_COUNTER__
    #define LV_TICK_CUSTOM 1
    #if LV_TICK_CUSTOM
        extern uint32_t SystemCoreClock;
        #define LV_TICK_CUSTOM_INCLUDE          "perf_counter.h"
        #define LV_TICK_CUSTOM_SYS_TIME_EXPR    get_system_ms()
    #endif   /*LV_TICK_CUSTOM*/
#else
    #define LV_TICK_CUSTOM 0
    #if LV_TICK_CUSTOM
        #define LV_TICK_CUSTOM_INCLUDE "Arduino.h"         /*Header for the system time function*/
        #define LV_TICK_CUSTOM_SYS_TIME_EXPR (millis())    /*Expression evaluating to current system time in ms*/
        /*If using lvgl as ESP32 component*/
        // #define LV_TICK_CUSTOM_INCLUDE "esp_timer.h"
        // #define LV_TICK_CUSTOM_SYS_TIME_EXPR ((esp_timer_get_time() / 1000LL))
    #endif   /*LV_TICK_CUSTOM*/
#endif       /*__PERF_COUNTER__*/
```
9. Thoroughly remove the `DEMO USAGE` section and add following code:

   ```c
   /*Show some widget. It might be required to increase `LV_MEM_SIZE` */
   #if LV_USE_DEMO_WIDGETS
       #define LV_DEMO_WIDGETS_SLIDESHOW 0
   #endif

   /*Benchmark your system*/
   #if LV_USE_DEMO_BENCHMARK
       /*Use RGB565A8 images with 16 bit color depth instead of ARGB8565*/
       #define LV_DEMO_BENCHMARK_RGB565A8 0
   #endif
   ```



10. Remove following macro definitions in the `3rd party libraries` section:

    - \#define LV_USE_FS_STDIO 0
    - \#define LV_USE_FS_POSIX 0
    - \#define LV_USE_FS_WIN32 0
    - \#define LV_USE_FS_FATFS 0
    - \#define LV_USE_LODEPNG 0
    - \#define LV_USE_BMP 0
    - \#define LV_USE_RLE 0
    - \#define LV_USE_SJPG 0
    - \#define LV_USE_GIF 0
    - \#define LV_USE_BARCODE 0
    - \#define LV_USE_QRCODE 0
    - \#define LV_USE_FREETYPE 0
    - \#define LV_USE_TINY_TTF 0
    - \#define LV_USE_RLOTTIE 0
    - \#define LV_USE_FFMPEG 0

11. Remove unsupported devices from the `DEVICES` section

    - LV_USE_SDL

    - LV_USE_LINUX_FBDEV

    - LV_USE_NUTTX_FBDEV

    - LV_USE_LINUX_DRM

    - LV_USE_TFT_ESPI

12. rename '**lv_conf_template.h**' to '**lv_conf_cmsis.h**'.



## STEP 2 Check, Update and Run the 'gen_pack.sh'

```sh
if [ `uname -s` = "Linux" ]
  then
  CMSIS_PACK_PATH="/home/$USER/.arm/Packs/ARM/CMSIS/5.7.0/"
  PATH_TO_ADD="$CMSIS_PACK_PATH/CMSIS/Utilities/Linux64/"
else
  CMSIS_PACK_PATH="/C/Users/$USER/AppData/Local/Arm/Packs/ARM/CMSIS/5.7.0"
  PATH_TO_ADD="/C/Program Files (x86)/7-Zip/:$CMSIS_PACK_PATH/CMSIS/Utilities/Win32/:/C/xmllint/"
fi
[[ ":$PATH:" != *":$PATH_TO_ADD}:"* ]] && PATH="${PATH}:${PATH_TO_ADD}"
echo $PATH_TO_ADD appended to PATH
echo " "
```



### A. For Windows users

Update the '**CMSIS_PACK_PATH**' accordingly (Usually just replace the name gabriel with your own windows account name is sufficient.).

Update the '**PATH_TO_ADD**' to point to the installation folders of **7Zip** and **xmllint**.

Launch the git-bash and go to the cmsis-pack folder.

enter the following command:

```sh
./gen_pack.sh
```



### B. For Linux Users

Update '**PATH_TO_ADD**' if necessary.

go to the **cmsis-pack** folder.

enter the following command:

```sh
./gen_pack.sh
```

