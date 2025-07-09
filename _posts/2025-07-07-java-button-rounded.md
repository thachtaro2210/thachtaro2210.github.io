---
title: "Tạo nút bo góc trong Java Swing đơn giản"
date: 2025-07-07

categories: [Java, UI]
tags: [java swing, ui, button, custom component]

---

Trong bài viết này, mình sẽ chia sẻ cách tạo một nút (JButton) có góc bo tròn trong Java Swing – một bước nhỏ nhưng rất quan trọng khi bạn muốn làm giao diện hiện đại hơn.

## 🧩 Vấn đề

Mặc định, `JButton` trong Swing có hình chữ nhật góc vuông. Muốn làm bo tròn thì phải custom lại cách nó được vẽ.

## ✅ Giải pháp

Chúng ta sẽ tạo một class kế thừa `JButton` và override lại `paintComponent()` để vẽ hình bo tròn.

```java
import java.awt.*;
import javax.swing.*;

public class RoundedButton extends JButton {

    public RoundedButton(String text) {
        super(text);
        setContentAreaFilled(false);
        setFocusPainted(false);
    }

    @Override
    protected void paintComponent(Graphics g) {
        Graphics2D g2 = (Graphics2D) g.create();
        g2.setColor(getBackground());
        g2.fillRoundRect(0, 0, getWidth(), getHeight(), 30, 30);
        super.paintComponent(g);
        g2.dispose();
    }
}
```
🎨 Cách sử dụng:
RoundedButton btn = new RoundedButton("Click Me");


{% raw %}
<iframe src="https://docs.google.com/forms/d/e/1FAIpQLSeeB-npfdj4nPgiWxvV_BFJIpB40BW-uYF9K62YMtDsfn3fsg/viewform?embedded=true"
        width="100%" height="855" frameborder="0" marginheight="0" marginwidth="0">
  Đang tải…
</iframe>
{% endraw %}


btn.setBackground(new Color(66, 135, 245));
btn.setForeground(Color.WHITE);


