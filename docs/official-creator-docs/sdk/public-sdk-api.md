# Public SDK API

VRChat SDK 提供了一系列接口和方法，您可以使用它们来增强您的世界和头像构建过程。

您可以在两个 SDK 中找到 **Public SDK API** 文件夹：

- Packages > VRChat SDK - Worlds > Editor > VRCSDK > SDK3 > Public SDK API
- Packages > VRChat SDK - Avatars > Editor > VRCSDK > SDK3A > Public SDK API

然而，大多数事件和方法都是世界和模型 SDK 之间共享的，并且定义在 **基础 SDK Packages** 中。

# 目前有哪些是可用的？

要获取最新的事件和方法列表，我们建议可以直接在上面提到的文件中查看。

但这里有一个可供参考的简短列表：

-  SDK 主面板的 OnEnable/OnDisable 事件
- 构建开始/结束事件
- 上传成功/错误事件
- 构建、构建并测试，以及构建并上传方法

::: tip
如果您在构建过程中遇到异常，您可以在接口定义中查看预期异常的列表。
:::
## 示例

### 获取构建器的实例

连接到 `OnSdkPanelEnable` 将会确保 SDK 窗口被打开并且构建器已注册。然后您可以使用 `TryGetBuilder` 来获取您需要的构建器实例。

> 您可以在任何时候调用 `VRCSdkControlPanel.TryGetBuilder`，但如果 SDK 窗口没有打开或您尝试访问的构建器不可用，它将返回 `false` 。

```csharp
[InitializeOnLoadMethod]
public static void RegisterSDKCallback()
{
    VRCSdkControlPanel.OnSdkPanelEnable += AddBuildHook;
}

private IVRCSdkAvatarBuilderApi _builder;

private static void AddBuildHook(object sender, EventArgs e)
{
    VRCSdkControlPanel.TryGetBuilder<IVRCSdkAvatarBuilderApi>(out _builder);
}
```

### 在构建前运行代码

`OnSdkBuildStart` 在 SDK 启动构建过程之前运行，但在通过验证和构建请求回调之后。

```csharp
[InitializeOnLoadMethod]
public static void RegisterSDKCallback()
{
    VRCSdkControlPanel.OnSdkPanelEnable += AddBuildHook;
}

private static void AddBuildHook(object sender, EventArgs e)
{
    if (VRCSdkControlPanel.TryGetBuilder<IVRCSdkAvatarBuilderApi>(out var builder))
    {
        builder.OnSdkBuildStart += OnBuildStarted;
    }
}

private static void OnBuildStarted(object sender, object target)
{
    Debug.Log("Building " + ((GameObject) target).name);
}
```

### 从脚本中构建

```csharp
[MenuItem("My Tools/Build Selected Avatar")]
public static async void BuildSelectedAvatar()
{
    var avatar = Selection.activeGameObject;
    if (!VRCSdkControlPanel.TryGetBuilder<IVRCSdkAvatarBuilderApi>(out var builder)) return;
    try {
      await builder.BuildAndTest(avatar);
    } catch (Exception e) {
      Debug.LogError(e.Message);
    }
}
```
## 注意
::: caution
如果您目前使用 reflection 来访问 SDK 内部，我们建议您尽快切换到 Public API。
:::

我们将尽最大努力提供一个稳定的 API，但它仍然可能在未来发生变动。我们建议利用 semver 来定义您的工具与哪个版本的 SDK 兼容。[在这里了解更多详情](https://vcc.docs.vrchat.com/vpm/packages/#versions-and-ranges)。
