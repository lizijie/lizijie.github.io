---
layout: post
title: 自动化设置Unity3d GameView窗口分辨率的一个想法
key: 202006262223
tags: unity
---

[TOC]
源于新人在开发界面前，容易忘记要先在GameView设置分辨率大小。导致在目标分辨率下布局效果达不到要求。

从技术层面，我想一个解决办法是，分辨率默在配置表中定义，当首次打开Unity工程时，GameView默认使用这个分辨率。

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2020-09-26-%E8%87%AA%E5%8A%A8%E5%8C%96%E8%AE%BE%E7%BD%AEUnity3d%20GameView%E7%AA%97%E5%8F%A3%E5%88%86%E8%BE%A8%E7%8E%87%E7%9A%84%E4%B8%80%E4%B8%AA%E6%83%B3%E6%B3%95/size_view.png)

翻了UnityEditor源码，发现其并不开放设置GameView窗口的接口。于是写了以下代码，尝试通过修改其配置文件GameView.asset，追加配置。同时Unity也不开放其内部YAML的序列化接口，于使用了AssetStore的第三库YamlDotNet。但YamlDotNet的输出内容，Unity原生的YAML格式，有一下差别，见图。
```c#
void Test()
{
    string inFile = UnityEditorInternal.InternalEditorUtility.unityPreferencesFolder + "/GameViewSizes.asset";
    StreamReader input = File.OpenText(inFile);

    YamlStream yaml = new YamlStream();
    yaml.Load(input);
    input.Dispose();

    // 添加一项分辨率配置
    YamlMappingNode mapping = (YamlMappingNode)(yaml.Documents[0].RootNode);
    YamlMappingNode standaloneNode = (YamlMappingNode)(mapping["MonoBehaviour"]["m_Standalone"]);
    YamlSequenceNode m_Custom = (YamlSequenceNode)(standaloneNode["m_Custom"]);

    YamlMappingNode node = new YamlMappingNode();
    node.Add("m_BaseText", "test");
    node.Add("m_SizeType", "1");    
    node.Add("m_Width", "10");
    node.Add("m_Height", "20");
    m_Custom.Add(node);

    string outFile = UnityEditorInternal.InternalEditorUtility.unityPreferencesFolder + "/GameViewSizesTemp.asset";
     StreamWriter output = File.CreateText(outFile);
     yaml.Save(output);
     output.Dispose();
}
```

![](https://raw.githubusercontent.com/lizijie/lizijie.github.io/master/assets/images/2020-09-26-%E8%87%AA%E5%8A%A8%E5%8C%96%E8%AE%BE%E7%BD%AEUnity3d%20GameView%E7%AA%97%E5%8F%A3%E5%88%86%E8%BE%A8%E7%8E%87%E7%9A%84%E4%B8%80%E4%B8%AA%E6%83%B3%E6%B3%95/compare.png)

待解决

<br>	
<br>	
<b>原文:<br>
<https://lizijie.github.io/2020/04/20/%E5%A5%97%E6%96%B9%E6%A1%88%E6%98%AF%E8%A7%A3%E5%86%B3%E9%97%AE%E9%A2%98%E7%9A%84%E9%94%99%E8%AF%AF%E6%80%9D%E8%B7%AF.html>
<br>
作者github:<br>	
<https://github.com/lizijie>
</b>
