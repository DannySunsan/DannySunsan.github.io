---
title: wpf输入法对键盘输入事件的影响
tags: 输入法
categories: wpf
---

#引子
想要使用TextBox控件来输入文件名，考虑到文件名不能是非法字符，所以需要对输入字符进行控制，我分别对PreviewTextInput、PreviewKeyDown和Pasting事件做了处理。在PreviewTextInput中通过正则表达式禁止输入非法字符(//:*?"<>)，在PreviewKeyDown中控制首字符不为空格，最后在Pasting中替换非法字符。逻辑上没什么大的问题，在英文输入法下面测试也很正常，不过在中文输入法的情况下，出现了问题---button事件和input事件的顺序变了。

![控制代码](TextBoxInput.png)

英文输入法下面测试时事件的响应顺序为：PrevieKeyDown -> PreviewTextInput

中文输入法下面测试时事件的响应顺序为：PreviewTextInput ->  PrevieKeyDown 

TextInput事件使你能够以与设备无关的方式侦听文本输入。 键盘是文本输入的主要方式，但通过语音、手写和其他输入设备也可以生成文本输入。

对于键盘输入， WPF 首先发送相应的 KeyDown / KeyUp 事件。 如果未处理这些事件，并且键是文本 (而不是控件键（如方向箭头或函数键）) ，则 TextInput 会引发事件。 和事件之间并不总是有一种简单的一对一映射， KeyDown / KeyUp TextInput 因为多个击键可以生成一个文本输入字符，而单个击键可以生成多个字符的字符串。 对于中文、日语和韩语等语言，这种情况尤其适用，它们使用输入法编辑器 (Ime) 在其相应的字母表中生成成千上万个可能的字符。

当 WPF 发送 KeyUp / KeyDown 事件时， Key Key.System 如果按下了 ALT + S，则将设置为，如果击键可能成为 TextInput 事件 (的一部分，例如) 。 这样， KeyDown 事件处理程序中的代码就可以检查 Key.System 和（如果找到）对随后引发的事件的处理程序保留处理 TextInput 。 在这些情况下， TextCompositionEventArgs 可以使用参数的各种属性来确定原始击键。 同样，如果输入法处于活动状态，则的 Key 值为 Key.ImeProcessed ，并 ImeProcessedKey 提供原始击键或击键。