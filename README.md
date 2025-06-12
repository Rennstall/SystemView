# SystemView

## Target Setup

1. Clone the [SystemView](https://github.com/Rennstall/SystemView) repository into `/BSW` as submodule.

    ```sh
    cd ./lib/BSW
    git submodule add https://github.com/Rennstall/SystemView
    ```

2. Copy compile flags into `platformio.ini` from the newewst `SWTemplate` repo.

3. initialize SystemView recording in `src/freertos.c`
    ```c
    #include "SystemView/SEGGER_SYSVIEW.h"
    /* ... */
    void MX_FREERTOS_Init(void) {
        SEGGER_SYSVIEW_Conf();

        /* ... */
        /* user task setup */
        /* ... */
    }
    ```

4. Patch FreeRTOS files
    > The patch is supplied by `CGEN` and should be executed with `postgen.bat`

    Patched files are in 
    * `boards/*/FreeRTOSConfig.h`
    * `lib/FreeRTOS/task.c`
    * `lib/FreeRTOS/include/FreeRTOS.h`
    * `lib/FreeRTOS/include/task.h`
    * `lib/FreeRTOS/portable/GCC/ARM_CM4F/port.c`
    * `lib/FreeRTOS/portable/GCC/ARM_CM4F/portmacro.h`

    The new files are stored in `scripts/CGEN/SystemView/`


> [!INFO]
> The ECU will start recording as soon as `SystemView` is attached

> [!ATTENTION]
> A debugger must be initially attached for reading timestamps.
> A workaround is needed to unlock the debug/trace core without attached debugger



4. One Shot recording is possle
    * add this snippet around the snippet to be recorded

    ```c
    SEGGER_SYSVIEW_Start();
    /* do stuff */
    SEGGER_SYSVIEW_Stop();
    ```


5. Add plotting variables (if desired)

    For whatever reason, the variable will only show up in the plot window
    when the RegisterData is called somewhat regularly.

    When calling `init` only once at startup, with subsequent `...SampleData` calls, the
    value will show up in the console, but not in the plot window.
   
    ```c
    #include "SystemView/SEGGER_SYSVIEW.h"

    void init(){

        /* setup the signal, can be done everywhere */
        SEGGER_SYSVIEW_DATA_REGISTER vPlot;
        vPlot.ID = 0;
        vPlot.sName = "TestSignal";
        vPlot.DataType = SEGGER_SYSVIEW_TYPE_U32;
        vPlot.Offset = 0;
        vPlot.RangeMax = 600;
        vPlot.RangeMin = 0;
        vPlot.sUnit = "V";

        /* either start one-shot recording here if required */
        // SEGGER_SYSVIEW_Start();

        SEGGER_SYSVIEW_RegisterData(&vPlot);
    }

    void loop(){

        /* setup logging variables */
        uint32_t voltage = 0;
        SEGGER_SYSVIEW_DATA_SAMPLE sample;
        sample.ID = 0;
        sample.pValue.pU32 = &voltage;

        /* actual tracing */
        for (uint32_t v = 0; v < 600; v++) {
            voltage = v;
            SEGGER_SYSVIEW_SampleData(&sample);
            vTaskDelay(1);
        }

        /* hold in case of one-shot recording */
        // SEGGER_SYSVIEW_Stop();
    }
    ```



