# EnumDevicesBenchmark

Console Utility to benchmark DirectInput8 EnumDevices. Might be useful to you If you're struggling with game freezes/lags :)

I know this isn't the proper way to publish code, but just going to paste it in the readme quickly. Can be compiled as a console app in Visual Studio.


```#include <windows.h>
#include <dinput.h>
#include <iostream>
#include <string>

// Link to libraries automatically
#pragma comment(lib, "dinput8.lib")
#pragma comment(lib, "dxguid.lib")

// Global DirectInput pointer
LPDIRECTINPUT8 g_pDI = nullptr;

BOOL CALLBACK EnumDevicesCallback(const DIDEVICEINSTANCE* pdidInstance, VOID* pContext)
{
    // Declare variables to store timing information
    LARGE_INTEGER start, end, freq;

    QueryPerformanceFrequency(&freq);
    QueryPerformanceCounter(&start);

    // Print out device name (tszProductName is a wide string)
    std::wcout << L"  Device: " << pdidInstance->tszProductName;

    QueryPerformanceCounter(&end);

    // Calculate the elapsed time in milliseconds
    double elapsedSeconds = double(end.QuadPart - start.QuadPart) / double(freq.QuadPart);
    double elapsedMs = elapsedSeconds * 1000.0;

    std::wcout << L" - Time: " << elapsedMs << L" ms" << std::endl;

    return DIENUM_CONTINUE;
}

// Helper function to enumerate devices and measure its time
void EnumerateAndMeasure()
{
    // QueryPerformanceCounter for high-resolution timing
    LARGE_INTEGER freq;
    LARGE_INTEGER start;
    LARGE_INTEGER end;

    QueryPerformanceFrequency(&freq);

    QueryPerformanceCounter(&start);
    HRESULT hr = g_pDI->EnumDevices(
        DI8DEVCLASS_ALL,         // what device class to enumerate
        EnumDevicesCallback,     // callback function
        NULL,                    // user data for callback (not used here)
        DIEDFL_ALLDEVICES        // flags for device enumeration
    );
    QueryPerformanceCounter(&end);

    if (FAILED(hr))
    {
        std::cerr << "EnumDevices failed: 0x" << std::hex << hr << std::dec << std::endl;
        return;
    }

    // Calculate elapsed time in microseconds (or milliseconds)
    double elapsedSeconds = double(end.QuadPart - start.QuadPart) / double(freq.QuadPart);
    double elapsedMs = elapsedSeconds * 1000.0;
    double elapsedMicro = elapsedSeconds * 1'000'000.0;

    std::cout << "Enumeration time: " << elapsedMs << " ms ("
        << elapsedMicro << " µs)" << std::endl;
}

int main()
{
    // 1. Initialize DirectInput
    HRESULT hr = DirectInput8Create(
        GetModuleHandle(NULL),
        DIRECTINPUT_VERSION,
        IID_IDirectInput8,
        (VOID**)&g_pDI,
        NULL
    );

    if (FAILED(hr) || !g_pDI)
    {
        std::cerr << "DirectInput8Create failed: 0x" << std::hex << hr << std::dec << std::endl;
        return -1;
    }

    std::cout << "DirectInput8Create succeeded!" << std::endl;

    // 2. Simple loop: user presses Enter to re-run enumeration, or 'q' to quit
    while (true)
    {
        std::cout << "\nPress ENTER to enumerate devices, or type 'q' to quit...\n";

        // Wait for user input
        std::string input;
        if (!std::getline(std::cin, input))  // handle EOF or error
            break;

        if (input == "q" || input == "Q")
        {
            std::cout << "Exiting loop.\n";
            break;
        }

        // 3. Call enumeration & measure performance
        std::cout << "[Enumerating devices...]\n";
        EnumerateAndMeasure();
    }

    // 4. Cleanup
    if (g_pDI)
        g_pDI->Release();

    return 0;
}
