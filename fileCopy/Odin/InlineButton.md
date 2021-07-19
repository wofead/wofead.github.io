# Inline Button

> Inline Button Attribute：用于将一个按钮添加到属性的末尾

![img](../image/InlineButton/post-622-5fb7daa2eb197.png)

```cs
// Inline Buttons:
[InlineButton("A")]
public int InlineButton;

[InlineButton("A")]
[InlineButton("B", "Custom Button Name")]
public int ChainedButtons;

private void A()
{
    Debug.Log("A");
}

private void B()
{
    Debug.Log("B");
}
```