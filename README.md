# ImGui Kiero Hook - DX11 Overlay Tutorial

This guide walks you through creating a basic **DirectX 11 ImGui overlay** using a **Kiero hook**. It explains everything step-by-step, making it ideal for beginners learning about game hacking, overlay development, or ImGui injection.

> **Disclaimer**: This project is for **educational purposes only**. Do **not** use it to cheat in online multiplayer games.

---

## 🧰 Requirements

- Visual Studio 2019 or newer  
- A DirectX 11 game or application  
- [kiero](https://github.com/rdbo/ImGui-DirectX-11-Kiero-Hook) and [imgui](https://github.com/ocornut/imgui)

---

## 🚀 Steps

### 1. Set Up Your Visual Studio Project
- Create a new **DLL project**  
- Set **Character Set** to **Multi-Byte**  
- Set **Subsystem** to **Windows**  
- Link `d3d11.lib` and `dxgi.lib`:  
  `Project Properties > Linker > Input > Additional Dependencies`

---

### 2. Include Dependencies
- Add **Kiero** and **ImGui** to your project  
- Include the following ImGui source files:

```cpp
imgui.cpp  
imgui_draw.cpp  
imgui_tables.cpp  
imgui_widgets.cpp  
imgui_impl_win32.cpp  
imgui_impl_dx11.cpp
```

---

### 3. Hook the Present Function

```cpp
typedef HRESULT(__stdcall* Present)(IDXGISwapChain*, UINT, UINT);
Present oPresent = nullptr;
```

---

### 4. Setup ImGui Context Once

```cpp
ID3D11Device* device = nullptr;
ID3D11DeviceContext* context = nullptr;
ID3D11RenderTargetView* renderTargetView = nullptr;
HWND window = nullptr;

void InitImGui(IDXGISwapChain* swapChain) {
    DXGI_SWAP_CHAIN_DESC desc;
    swapChain->GetDesc(&desc);
    window = desc.OutputWindow;

    swapChain->GetDevice(__uuidof(ID3D11Device), (void**)&device);
    device->GetImmediateContext(&context);

    ID3D11Texture2D* backBuffer = nullptr;
    swapChain->GetBuffer(0, __uuidof(ID3D11Texture2D), (LPVOID*)&backBuffer);
    device->CreateRenderTargetView(backBuffer, nullptr, &renderTargetView);
    backBuffer->Release();

    ImGui::CreateContext();
    ImGui_ImplWin32_Init(window);
    ImGui_ImplDX11_Init(device, context);
}
```

---

### 5. Custom WndProc for Input

```cpp
WNDPROC originalWndProc;

LRESULT CALLBACK WndProc(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam) {
    if (ImGui_ImplWin32_WndProcHandler(hWnd, msg, wParam, lParam))
        return true;
    return CallWindowProc(originalWndProc, hWnd, msg, wParam, lParam);
}
```

---

### 6. Your Hook Function

```cpp
bool showMenu = true;

HRESULT __stdcall hkPresent(IDXGISwapChain* pSwapChain, UINT SyncInterval, UINT Flags) {
    if (!device) {
        InitImGui(pSwapChain);
        originalWndProc = (WNDPROC)SetWindowLongPtr(window, GWLP_WNDPROC, (LONG_PTR)WndProc);
    }

    if (GetAsyncKeyState(VK_INSERT) & 1)
        showMenu = !showMenu;

    ImGui_ImplDX11_NewFrame();
    ImGui_ImplWin32_NewFrame();
    ImGui::NewFrame();

    if (showMenu) {
        ImGui::Begin("DX11 Hook");
        ImGui::Text("Hello, ImGui!");
        ImGui::End();
    }

    ImGui::Render();
    context->OMSetRenderTargets(1, &renderTargetView, nullptr);
    ImGui_ImplDX11_RenderDrawData(ImGui::GetDrawData());

    return oPresent(pSwapChain, SyncInterval, Flags);
}
```

---

### 7. Start the Hook

```cpp
DWORD WINAPI MainThread(LPVOID) {
    if (kiero::init(kiero::RenderType::D3D11) == kiero::Status::Success) {
        kiero::bind(8, (void**)&oPresent, hkPresent); // Present is vtable index 8
    }
    return 0;
}
```

---

### 8. Injecting the DLL

```cpp
BOOL APIENTRY DllMain(HMODULE hModule, DWORD reason, LPVOID) {
    if (reason == DLL_PROCESS_ATTACH) {
        DisableThreadLibraryCalls(hModule);
        CreateThread(nullptr, 0, MainThread, hModule, 0, nullptr);
    }
    return TRUE;
}
```

---

## 📝 Final Notes

- Build as a **DLL**
- Inject using a tool like **Xenos**
- Press **Insert** to toggle the menu

---

## 📦 Credits

- [ocornut/imgui](https://github.com/ocornut/imgui)  
- [Rebzzel/kiero](https://github.com/Rebzzel/kiero)
