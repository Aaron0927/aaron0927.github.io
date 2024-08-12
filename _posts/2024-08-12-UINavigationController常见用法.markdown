---
layout: post
title: "UINavigationController常见用法"
date: 2024-08-12 22:30:00 +0800
categories: jekyll blog
---


本文主要是收集关于`UINavigationController`相关的常用功能，对于`UINavigationController`的介绍，下面的文章介绍的很详细了：
* [# 学习UINavigationController（1）](https://juejin.cn/post/6937492008160198663)
* [# 学习UINavigationController（2）](https://juejin.cn/post/6937878071370334215/)
* [# 学习UINavigationController（3）](https://juejin.cn/post/6938056448056229896/)
* [# 学习UINavigationController（4）](https://juejin.cn/post/6938429340275179557/)

### 页面结构
`UINavigationController`的页面结构图如下：

```
UINavigationController
	|-UINavigationBar
	|-controllers
		|-UIViewController ~> UINavigationItem
		|-UIViewController ~> UINavigationItem
		|-UIViewController ~> UINavigationItem
		|...
```
![](../static/img/UINavigationController.awebp)

1. UINavigationController 只有一个 UINavigationBar，对UINavigationBar修改会影响到所有页面的显示
2. UINavigationController有 viewControllers 对象，UINavigationBar有 items 对象，每一个 Controller 有自己的 UINavigationItem，可以对自己的UINavigationItem修改
3. UINavigationController提供backItem和 leftItem，leftItem 优先级更高

### 常见用法
通常情况下，我们会在项目中创建`UINavigationController`的基类，以下就以`BaseNavigationController` 举例：

1. 隐藏`UITabBar`
```swift
override func pushViewController(_ viewController: UIViewController, animated: Bool) {
	// viewController是即将要 push 的控制器
	if viewControllers.count > 0 {
		viewController.hidesBottomBarWhenPushed = true
	}
	super.pushViewController(viewController, animated: animated)
}
```


2. 隐藏系统导航栏，使用自定义导航栏
	1. 创建`BaseViewController`，作为所有控制器的基类
	2. 通过设置透明图片来隐藏系统导航栏，但是保留系统导航栏的交互效果
	3. 创建自定义导航栏方法

示例代码如下：
```swift
override func viewDidLoad() {
	super.viewDidLoad()

	// 隐藏默认返回按钮的文字，并修改返回按钮图标颜色
	navigationItem.backBarButtonItem = UIBarButtonItem.init(title: "", image: UIImage(), primaryAction: nil, menu: nil)
	navigationItem.backBarButtonItem?.tintColor = .black

	// 设置导航栏透明
	navigationController?.navigationBar.setBackgroundImage(UIImage(), for .default)
	navigationController?.navigationBar.shadowImage = UIImage()

	// 添加自定义导航栏背景
	addNavBar(.white)
}

// MARK: 添加自定义导航栏方法
// 使用背景色填充导航栏
func addNavBar(_ color: UIColor) {
	let size = CGSize(width: view.bounds.width, height: kStatusH + kNavBarH)
	let navImageView = UIImageView(image: UIImage(size: size, color: color))		view.addSubview(navImageView)
}

// 使用自定义视图填充导航栏
func addCustomNavNar(_ view: UIView) {
	let size = CGSize(width: view.bounds.width, height: kNavBarH)
	view.frame = CGRectMake(0, 0, size.width, size.height)
	navigationController?.navigationBar.addSubview(view)
}
```

通过颜色创建`UIImage` ：
```swift
extension UIImage {
    convenience init(size: CGSize, color: UIColor, scale: CGFloat = UIScreen.main.scale) {
        let renderer = UIGraphicsImageRenderer(size: size, format: UIGraphicsImageRendererFormat.default())
        let image = renderer.image { ctx in
            color.setFill()
            ctx.fill(CGRect(x: 0, y: 0, width: size.width, height: size.height))
        }
        guard let cgImage = image.cgImage else {
            fatalError("Failed to create CGImage from UIImage")
        }
        self.init(cgImage: cgImage, scale: scale, orientation: .up)
    }
}
```

视图尺寸相关代码：
```swift
// 状态栏高度
var kStatusBarH: CGFloat {
    if #available(iOS 13.0, *) {
        let window = UIApplication.shared.windows.filter {$0.isKeyWindow}.first
        return window?.windowScene?.statusBarManager?.statusBarFrame.height ?? 0
    } else {
        return UIApplication.shared.statusBarFrame.height
    }
}

// 导航栏高度
var kNavBarH: CGFloat {
    UINavigationController().navigationBar.frame.size.height
}
```


3. 隐藏系统导航栏，并保留系统侧滑手势
隐藏系统导航栏后，对应的侧滑返回也没有了，如果想要保留系统的侧滑返回手势，在对应的控制器中添加如下代码：
```swift
// 在控制器中添加以下代码
override func viewWillAppear(_ animated: Bool) {
	super.viewWillAppear(animated)

	// 隐藏系统导航栏后，系统的侧滑手势就没有了
	navigationController?.setNavigationBarHidden(true, animated: true)
	// 保留系统侧滑手势
	navigationController?.interactivePopGestureRecognizer?.delegate = self
	navigationController?.interactivePopGestureRecognizer?.isEnabled = true
}
```


4. 实现全屏侧滑手势
	实现思路：
	1. 获取系统的侧滑手势
	2. 获取侧滑手势的视图
	3. 获取侧滑手势的 target&action
	4. 创建自定义手势，并添加到系统侧滑手势视图中，添加 target&action


获取系统侧滑手势和视图
```swift
// 获取系统侧滑手势
let popGesture = interactivePopGestureRecognizer

// 获取手势视图
guard let gestureView = popGesture?.view else { return }
```

通过 `runtime` 获取系统手势的所有属性
```swift
var count: UInt32 = 0
// 获取系统手势的所有属性
guard let ivars = class_copyIvarList(UIGestureRecognizer.self, &count) else { return }
for i in 0..<count {
	let ivar = ivars[Int(i)]
	let name = ivar_getName(ivar)!
	print(String(cString: name))
}
```

从打印的属性中获取到我们要找到的属性`_targets`，使用 KVC 获取
```swift
let targets = popGesture?.value(forKey: "_targets") as? [NSObject]
```

打印出来是这样的一个数组
```
(action=handleNavigationTransition:, target=<_UINavigationInteractiveTransition 0x101b37930>)
```

获取 target&action
```swift
guard let targetObj = targets?.first else { return }
// 取出 target 和 action
guard let target = targetObj.value(forKey: "target") else { return }
let action = Selector(("handleNavigationTransition:"))
```

添加自定义手势
```swift
// 创建自己的pan手势
let panGesture = UIPanGestureRecognizer()
gestureView.addGestureRecognizer(panGesture)
panGesture.addTarget(target, action: action)
```

这样就借助系统的侧滑手势完成了全屏返回功能