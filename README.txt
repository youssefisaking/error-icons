the code i used (its c++) #include <windows.h>
#include <vector>

// Define GET_X_LPARAM and GET_Y_LPARAM macros for mouse coordinate extraction
#ifndef GET_X_LPARAM
#define GET_X_LPARAM(lp) ((int)(short)LOWORD(lp))
#endif
#ifndef GET_Y_LPARAM
#define GET_Y_LPARAM(lp) ((int)(short)HIWORD(lp))
#endif

static std::vector<POINT> g_points;
static HICON g_errorIcon = nullptr;

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam) {
    switch (msg) {
    case WM_CREATE:
        g_errorIcon = LoadIcon(NULL, IDI_ERROR); // system error icon
        return 0;

    case WM_MOUSEMOVE: {
        POINT p = { GET_X_LPARAM(lParam), GET_Y_LPARAM(lParam) };
        g_points.push_back(p);

        // Limit trail length to avoid memory bloat
        if (g_points.size() > 300) g_points.erase(g_points.begin());

        InvalidateRect(hWnd, NULL, FALSE);
        return 0;
    }

    case WM_PAINT: {
        PAINTSTRUCT ps;
        HDC hdc = BeginPaint(hWnd, &ps);

        for (const auto& pt : g_points) {
            int x = pt.x - 16, y = pt.y - 16;
            DrawIconEx(hdc, x, y, g_errorIcon, 32, 32, 0, NULL, DI_NORMAL);
        }

        EndPaint(hWnd, &ps);
        return 0;
    }

    case WM_LBUTTONDOWN:
        g_points.clear();
        InvalidateRect(hWnd, NULL, TRUE);
        return 0;

    case WM_DESTROY:
        if (g_errorIcon) DestroyIcon(g_errorIcon);
        PostQuitMessage(0);
        return 0;
    }
    return DefWindowProc(hWnd, msg, wParam, lParam);
}

int APIENTRY WinMain(HINSTANCE hInst, HINSTANCE, LPSTR, int nCmdShow) {
    WNDCLASS wc = { 0 };
    wc.lpfnWndProc = WndProc;
    wc.hInstance = hInst;
    wc.lpszClassName = L"ErrorTrailWindow";
    wc.hCursor = LoadCursor(NULL, IDC_ARROW);
    wc.hbrBackground = (HBRUSH)(COLOR_WINDOW + 1);

    if (!RegisterClass(&wc)) {
        return 1;
    }

    HWND hWnd = CreateWindow(wc.lpszClassName, L"Error Icon Trail",
        WS_OVERLAPPEDWINDOW, CW_USEDEFAULT, CW_USEDEFAULT, 800, 600,
        NULL, NULL, hInst, NULL);

    if (!hWnd) {
        return 1;
    }

    ShowWindow(hWnd, nCmdShow);
    UpdateWindow(hWnd);

    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }
    return (int)msg.wParam;
}