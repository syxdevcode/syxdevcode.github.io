---
title: Flutter基础
date: 2025-09-01 16:55:44
tags:
  - Flutter
categories:
  - Flutter
---

## Widget 状态

Flutter引入了两种主要的 Widget 类：有状态和无状态 Widget。

许多 Widget 没有可变状态：它们没有任何随时间变化的属性（例如，图标或标签）。这些 Widget 继承自 StatelessWidget。

然而，如果 Widget 的独特特性需要根据用户交互或其他因素而改变，那么该 Widget 就是有状态的。例如，如果一个 Widget 有一个计数器，每次用户点击按钮时都会递增，那么计数器的值就是该 Widget 的状态。当该值改变时，需要重建该 Widget 以更新其 UI 部分。这些 Widget 继承自 StatefulWidget，并且（因为 Widget 本身是不可变的）它们将可变状态存储在一个单独的继承自 State 的类中。StatefulWidget 没有 build 方法；相反，它们的用户界面是通过它们的 State 对象构建的。

每当您修改 State 对象时（例如，通过增加计数器），您必须调用 setState() 来通知框架再次调用 State 的 build 方法以更新用户界面。

拥有单独的状态和 Widget 对象使得其他 Widget 可以以完全相同的方式对待无状态和有状态 Widget，而不必担心丢失状态。父级不需要持有子级以保留其状态，而是可以随时创建子级的新实例而不会丢失子级的持久状态。框架会完成所有适当查找和重用现有状态对象的工作。

## 状态管理

许多 Widget 包含状态，可以使用 Widget 中的构造函数来初始化其数据，因此 build() 方法可以确保任何子 Widget 都使用所需的数据进行实例化。

InheritedWidget，提供了一种从共享祖先获取数据的简单方法。您可以使用 InheritedWidget 创建一个状态 Widget，该 Widget 包装 Widget 树中的一个共同祖先。

![inherited-widget.png](/img1/inherited-widget.png)

每当 ExamWidget 或 GradeWidget 对象中的一个需要 StudentState 中的数据时，它现在可以通过以下命令访问它：

``` dart
final studentState = StudentState.of(context);
```

of(context) 调用接收构建上下文（指向当前 Widget 位置的句柄），并返回树中最接近的与 StudentState 类型匹配的祖先。InheritedWidget 还提供了一个 updateShouldNotify() 方法，Flutter 调用该方法来确定状态更改是否应触发使用它的子 Widget 的重建。

## 渲染和布局

Flutter 使用自己的 Widget 集。绘制 Flutter 视觉效果的 Dart 代码被编译成原生代码，该原生代码使用 Impeller 进行渲染。Impeller 随应用程序一起提供，允许开发者升级他们的应用程序以保持与最新性能改进同步，即使手机尚未更新到新的 Android 版本。对于 Windows 或 macOS 等其他原生平台上的 Flutter 也是如此。

### 从用户输入到 GPU

![render-pipeline.png](/img1/render-pipeline.png)

### 构建：从 Widget 到 Element

当 Flutter 需要渲染此片段时，它会调用 build() 方法，该方法返回一个 Widget 子树，根据当前应用状态渲染 UI。在此过程中，build() 方法可以根据其状态插入新的 Widget。

在构建阶段，Flutter 将代码中表示的 Widget 转换为相应的元素树，每个 Widget 对应一个元素。每个元素代表 Widget 在树层级结构中给定位置的特定实例。元素有两种基本类型：

* ComponentElement，其他元素的宿主。
* RenderObjectElement，参与布局或绘制阶段的元素。

![widget-element.png](/img1/widget-element.png)

RenderObjectElement 是它们的 Widget 对应物和底层 RenderObject 之间的中介。

### 布局和渲染

渲染树中每个节点的基础类是 RenderObject，它定义了布局和绘画的抽象模型。这非常通用：它不限于固定数量的维度甚至笛卡尔坐标系。每个 RenderObject 都知道它的父级，但对它的子级了解甚少，除了如何访问它们以及它们的约束。这为 RenderObject 提供了足够的抽象，能够处理各种用例。

在构建阶段，Flutter 为元素树中的每个 RenderObjectElement 创建或更新一个继承自 RenderObject 的对象。RenderObject 是原始对象：RenderParagraph 渲染文本，RenderImage 渲染图像，而 RenderTransform 在绘制其子项之前应用变换。

![trees.png](/img1/trees.png)

大多数 Flutter Widget 都由继承自 RenderBox 子类的对象渲染，该子类表示 2D 笛卡尔空间中固定大小的 RenderObject。RenderBox 提供了盒约束模型的基础，为每个要渲染的 Widget 建立了最小和最大宽度及高度。

为了执行布局，Flutter 以深度优先遍历的方式遍历渲染树，并将大小约束从父级传递给子级。在确定其大小时，子级必须遵守父级给定的约束。子级通过将大小传递回其父对象，并在父级建立的约束范围内进行响应。

![constraints-sizes.png](/img1/constraints-sizes.png)

在遍历树的单次过程中，每个对象都在其父级的约束范围内具有定义的大小，并准备好通过调用 paint() 方法进行绘制。

盒约束模型作为在 O(n) 时间内布局对象的方式非常强大：

* 父级可以通过将最大和最小约束设置为相同的值来指定子对象的大小。例如，手机应用中最顶层的渲染对象将其子级约束为屏幕大小。（子级可以选择如何使用该空间。例如，它们可能只是在其指定约束内将要渲染的内容居中。）
* 父级可以指定子级的宽度，但赋予子级高度的灵活性（或者指定高度但提供宽度的灵活性）。一个真实的例子是流式文本，它可能必须适应水平约束，但垂直方向会根据文本数量而变化。

即使子对象需要知道它有多少可用空间来决定如何渲染其内容，此模型也适用。通过使用 LayoutBuilder Widget，子对象可以检查传递下来的约束并使用这些约束来确定如何使用它们，例如：

```dart
Widget build(BuildContext context) {
  return LayoutBuilder(
    builder: (context, constraints) {
      if (constraints.maxWidth < 600) {
        return const OneColumnLayout();
      } else {
        return const TwoColumnLayout();
      }
    },
  );
}
```

所有 RenderObject 的根是 RenderView，它代表渲染树的总输出。当平台要求渲染新帧时（例如，由于 垂直同步 或纹理解压/上传完成），会调用 compositeFrame() 方法，该方法是渲染树根部的 RenderView 对象的一部分。这会创建一个 SceneBuilder 来触发场景更新。当场景完成时，RenderView 对象将合成的场景传递给 dart:ui 中的 Window.render() 方法，该方法将控制权传递给 GPU 来渲染它。

参考：

[Flutter 架构概览](https://docs.flutterdart.cn/resources/architectural-overview#rendering-native-controls-in-a-flutter-app)