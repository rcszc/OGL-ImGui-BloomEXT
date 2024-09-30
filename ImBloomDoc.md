# ImGui For OpenGL - BloomFX

- __Version:__ `1.1` `20241001` __ByRCSZ.__
- __Libraries:__ `opengl32` `glew32` `imgui` 

### 执行后期Bloom渲染
> 1.0

我们需要替换ImGui原有的渲染上下文, 因为渲染完成后需要把ImGui-Scene通过缓冲帧离屏渲染到后期处理管道.

```cpp

BloomImGui::FxBloomSystem* MyGuiBloom = nullptr;

// ...

// 构造参数第一个为 缓冲帧大小(一般与图像窗口大小相同)
// 第二个为可选参数: std::function<void(const std::string&)> (如果需要自定义日志输出)
MyGuiBloom = new BloomImGui::FxBloomSystem(1680, 896);

// v1.1

// ...
// 进入渲染上下文(循环)后我们, 只需要极其简单的调用封装好的上下文.

MyGuiBloom->RenderContextCaptureBegin();

// ... (你的ImGui绘制代码)

MyGuiBloom->RenderContextCaptureEnd();

```
> 1.1

如果你使用的是 Win32 窗口需要注意, 裁剪尺寸不能去计算客户区尺寸, 渲染裁剪后两个参数需要和窗口创建时尺寸参数相同, CreateWindowW -> glViewport. 所以还是强烈推荐 GLFW + OPENGL3 的组合.

```cpp
// 现在我们在初始化时还需要加一项必要配置:
// 配置 ImGui API 的 "NewFrame" 函数.

// GLFW 使用: ImGui_ImplGlfw_NewFrame
// Win32 使用: ImGui_ImplWin32_NewFrame

// 并且需要手动包含 imgui_impl_xxx.h

MyGuiBloom->GetImGuiNewFrameFUNC(ImGui_ImplGlfw_NewFrame);
```

### 设置后期渲染参数
> 1.0

Bloom往往与HDR纹理相关, 我们首先就是可以设置颜色过滤器着色器, 来通过一系列条件过滤出你希望"发光"的片段. 然后我们设置Bloom效果中的模糊半径. 最后我们可设置输出阶段着色器的片段混合权重, 以及亮度. __不过这些都是可选的,系统内有预设值,你可以只设置模糊半径__

```cpp

// 获取过滤参数指针进行设置.
BloomImGui::FxFilterParams* SetFilter = MyGuiBloom->SettingFilterParams();

struct FxFilterParams {
    // 片段均值过滤, 通过RGB的平均值, 来判断当前片段能否通过测试. (这是在第二阶段)
	float ColorRGBchannelsAvgFilter = 0.32f;

    // 片段颜色通道过滤, 当前颜色是否在Min,Max区间, 来判断当前片段能否通过测试. (这是在第一阶段)
	std::array<float, 4> ColorMaxFilter = { 1.0f, 1.0f, 1.0f, 1.0f };
	std::array<float, 4> ColorMinFilter = { 0.0f, 0.0f, 0.0f, 0.0f };
};
```

```cpp
// 设置辉光处理中的模糊半径[0,30].
MyGuiBloom->SettingBloomRadius(12);
```

```cpp
// 获取混合参数指针进行设置.
BloomImGui::FxOutParams* SetBlend = MyGuiBloom->SettingBlendParams();

struct FxOutParams {
    // 模糊片段与原场景片段的混合权重.
    // 公式: TC * SourceBlend + TB * BlurBlend
	float SourceFragmentBlend = 0.72f;
	float BlurFragmentBlend   = 1.28f;

    // 总输出片段颜色混合权重.
	std::array<float, 4> FragmentColorBlend = { 1.0f, 1.0f, 1.0f, 1.0f };
};

```
是的到这里就结束了, 这是一个非常简单的ImGui辉光扩展.

### 注意事项

当窗口尺寸发生变化时需要重新创建辉光处理对象, 并且配置分辨率, 因为这关系到多个帧缓冲纹理的尺寸, 无法动态调整.

---

`END`