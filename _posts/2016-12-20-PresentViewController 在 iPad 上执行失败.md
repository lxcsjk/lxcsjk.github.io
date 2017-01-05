---
layout: post
title:  "PresentViewController 在 iPad 上执行失败"
date:   2016-12-20 20:00:03 +0800
categories: jekyll update
tags: iOS
---

在选择头像图片的时候 使用  ``` imagePickerController ```

```
  [self presentViewController:imagePickerController animated:YES completion:^{
            
  }];
```

---

异常信息

>2016-12-20 13:50:23.096416 Training[1963:904966] Warning: Attempt to present <UIImagePickerController: 0x14a9d0c00>  on <MySelfViewController: 0x149d2d480> which is already presenting (null)
>
2016-12-20 13:50:23.106841 Training[1963:904966] [Warning] <_UIPopoverBackgroundVisualEffectView 0x149dcbd40> is being asked to animate its opacity. This will cause the effect to appear broken until opacity returns to 1.

---

google了一下 找到了解决方法 [presentviewcontroller-animated-not-displaying-on-ipad](http://stackoverflow.com/questions/32380967/presentviewcontroller-animated-not-displaying-on-ipad)

```
    dispatch_async(dispatch_get_main_queue(), ^ {
        [self presentViewController:imagePickerController animated:YES completion:^{
            
        }];
    });
```

---


但是这是为啥呢 在iPhone上没问题 歪果仁是这样说的 [here](http://stackoverflow.com/questions/24854802/presenting-a-view-controller-modally-from-an-action-sheets-delegate-in-ios8-i/24854803#24854803)

>The problem seems to come from Apple's switch to using UIAlertController internally to implement the functionality of alert views and action sheets. The issue is seen mostly on iPad and action sheets, because on iPad, action sheets are presented as a popover within a specified view, and what Apple does is travel the responder chain until it finds a view controller and calls presentViewController:animated:completion: with the internal UIAlertController. The problem is less obvious on iPhone and with alert views, because there Apple actually creates a separate window, an empty view controller and presents the internal UIAlertController on top of that, so it seems to not interfere with other presentation.

大概意思是这样的

**presentViewController 需要找到一个视图控制器然后执行，在iPhone上，Apple实际上创建了一个单独的窗口，一个空视图控制器，并在其上显示，因此不会干扰其他显示。而iPad上面没有这样做。**

---






