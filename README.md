# SystemView

## Target Setup

1. Clone the [SystemView](https://github.com/Rennstall/SystemView) repository into `/BSW` as submodule.

    ```sh
    cd ./lib/BSW
    git submodule add https://github.com/Rennstall/SystemView
    ```

2. initialize and start SystemView recording in `src/freertos.c`
    ```c
    #include "SystemView/SEGGER_SYSVIEW.h"
    /* ... */
    void MX_FREERTOS_Init(void) {
        SEGGER_SYSVIEW_Conf();

        /* ... */
        /* user task setup */
        /* ... */

        SEGGER_SYSVIEW_Start();
    }
    ```


3. Add plotting variables (if desired)
    ```c
    #include "SystemView/SEGGER_SYSVIEW.h"

    void myFunc(){

        /* setup the signal, can be done everywhere */
        SEGGER_SYSVIEW_DATA_REGISTER vPlot;
        vPlot.ID = 0;
        vPlot.sName = "TestSignal";
        vPlot.DataType = SEGGER_SYSVIEW_TYPE_U32;
        vPlot.Offset = 0;
        vPlot.RangeMax = 600;
        vPlot.RangeMin = 0;
        vPlot.sUnit = "V";

        /* either start one-shot recording here
         * or start it with freertos.c for cont. recording */
        // SEGGER_SYSVIEW_Start();

        SEGGER_SYSVIEW_RegisterData(&vPlot);

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



4. One Shot recording is possle
    * remove the call to `SEGGER_SYSVIEW_Start()` in `freertos.c`
    * add this snippet around the snippet to be recorded

    ```c
    SEGGER_SYSVIEW_Start();
    /* do stuff */
    SEGGER_SYSVIEW_Stop();
    ```