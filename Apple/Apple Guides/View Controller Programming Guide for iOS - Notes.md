# 资料
- [自定义转场动画的 demo](http://www.iosask.com/article/140)


---

## Views 在运行时的展示: ([链接在此](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/DefiningYourSubclass.html#//apple_ref/doc/uid/TP40007457-CH7-SW1))
- loading process(加载阶段):
    1. 使用storyboard文件中的信息,实例化(instantiate)views;
    2. 连接所有的 outlets 和 actions;
    3. 把 root view 放进 view controller 的 view 里面;
    4. 调用 awakeFromNib 方法; 当这个方法被调用时, view controller的 trait conllection(?不会翻) 是空的, 而且 views 可能不在正确的位置上;
    5. 调用 viewDidLoad 方法. 使用这个方法用来添加或删除views, 修改布局约束, 还可以为你的 views 加载数据;
- 展现阶段:
    1. 调用 viewWillAppear: 方法, 这个方法让控制器知道它的 views 即将要在屏幕上显示了;
    2. 更新 views 的 layout;
    3. 将 views 展示到屏幕上;
    4. 方法当 views 在屏幕上显示时,调用 viewDidAppear: 方法;

---

## 管理 view 的 layout
当 views 的尺寸和位置发生改变时, UIKit 会为你的 view hierarchy 更新 layout 信息. 对于那些使用自动布局配置的 views, UIKit 使用自动布局引擎根据当前的约束进行布局的更新. 不仅如此, UIKit 还会让其它的对象, 比如正在使用的(active) presentation controller, 知晓布局的变化从而让它们可以做出相应的响应.

在布局过程中, UIKit 会通知你若干次, 从而你能够执行一些额外的与布局有关的任务. 使用这些通知修改你的布局约束, 或者在布局约束已经被使用之后对布局做一些轻微的调整. 在布局过程中, UIKit 为相关的 view controller 做了以下一些事情:
1. 更新 view controller 和它的 views 的 trait collection. 如有需要, 详见:[When Do Trait and Size Changes Happen?](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/TheAdaptiveModel.html#//apple_ref/doc/uid/TP40007457-CH19-SW6);
2. 调用 viewWillLayoutSubviews 方法;
3. 调用当前 UIPresentationController 对象的 containerViewWillLayoutSubviews 方法;
4. 调用 layoutSubviews 布局 view controller 的 root view;
5. 将计算好的布局信息应用到 views 上面;
6. 调用 view controller 的 viewDidLayoutSubviews 方法;
7. 调用 UIPresentationController 对象的 containerViewDidLayoutSubviews 方法;

View controllers 可以通过 viewWillLayoutSubviews 和 viewDidLayoutSubviews 方法去执行额外的可能影响到布局过程的更新. 在布局前, 你可能会添加或删除 views,更新 views 的尺寸和位置, 更新约束, 或者更新其他和 view 相关的 properties. 在布局之后, 你可能需要 reload 一下 tableView 的数据, 更新一些 views 的 content, 或者对一些 views 的尺寸和位置做最后的调整.

这里有一些 tips 可以帮助你更有效率地管理布局:
- **使用自动布局.** 使用自动布局创建的约束在不同的屏幕尺寸之间放置内容,是一种简单且灵活的方式;
- **利用 top and bottom layout guides.** 借助这些 guides 进行布局可以确保你的视图内容一致可见. Top layout guide 的位置与 status bar 和 navigation bar 有关. 相似地, bottom layout guide 的位置和 tabbar 或 toolbar 的高度有关;
- **在添加或移除 views 时切记更新约束.** 如果你动态添加或者移除 views, 记住一定要更新相关约束;
- **在 view controller 的 views 处于动画过程中时, 需要暂时移除约束.** 当使用 UIKit Core Animation 给 views 添加动画时, 在动画过程中需要移除约束, 然后在动画执行完毕之后, 再把约束添加回来. 请记住,如果在动画过程中, views 的尺寸和位置发生变化了,要更新约束.

---

# Configuring a Container in Interface Builder
![Container view](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/container_view_embed_2x.png) Container view 对象是一个'占位符'. 它代表一个子视图控制器的内容.使用这个 view 来确定 root view 和在 container 中其它 views 的尺寸和位置关系.

当你加载了一个其中含有若干个 container view 的视图控制器, Interface Builder 也会加载和那些 views 有关的子视图控制器. children 必须要和 parent 在同一时间实例化(instantiated), 这是为了建立合适的 parent-child 关系.

如果你不使用 Interface Builder 来建立 parent-child container 关系, 你必须使用程序将每一个 child 添加到 container view controller 来创建那些关系, 就像 [ Adding a Child View Controller to Your Content ](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/ImplementingaContainerViewController.html#//apple_ref/doc/uid/TP40007457-CH11-SW13) 中所描述的那样.

---

# Implementing a Custom Container View Controller
要实现一个 container view controller, 你必须在 view controller 和它的 child view controllers 之间建立某种关系. 在你试图管理这些 child view controllers 之前, 你必须建立这种父子关系. 这样做使得 UIKit 知道你的 view controller 正在管理 child view controllers 的尺寸和位置. 你可以在 IB 中 或者使用纯代码建立这种关系. 当你使用纯代码机那里父子关系时, 你需要在 view controller 的 setup 过程中显式添加或者移除 child view controllers.

## Adding a Child View controller to Your Content
如果需要在你的 content 中使用纯代码插入一个 child view controller, 以下是在相关的 view controlers 之间建立父子关系的步骤:
1. 调用 addChildViewController: 方法. 这个方法告诉了 UIKit 你的 container view controller 现在正在管理 child view controller 的 view.
2. 将你的子试图控制器的 root view 加入到 container 的 view 的层级(hierarchy)当中. 在这个过程中要时刻记住需要设置 child view controller 的 view 的尺寸和位置;
3. 为 child view controller 的 root view 添加约束;
4. 调用 child view controller 的 didMoveToParentViewController: 方法;

下面的代码展示了一个 child view controller 是如何嵌入一个 container 的. 在建立好父子关系之后, container 设置好了它 child view 的 frame ( 因为是在 container 这个控制器的 .m 中, 所以 content.view 的 frame 是 container 设置的而不是 content 设置的 ) , 然后 container 将 child's view 添加到了它的层级(hierarchy)之中. 

设置 child's view 的 frame size 非常重要, 因为这能够保证 child's view 能够在 container 中正确地显示出来. 在添加了 child's view 之后, container 调用了 content 的 didMoveToParentViewController: 方法, 这个方法给了 child view controller 一个响应 view ownership 改变的一个机会 ( 这句话的意思应该是 child view controller 的 root view 的所有权由 child view controller 移交到了 container ).
```
- (void) displayContentController: (UIViewController*) content {
   [self addChildViewController:content];
   content.view.frame = [self frameForContentController];
   [self.view addSubview:self.currentClientView];
   [content didMoveToParentViewController:self];
}

```
在刚才的例子中, 需要注意你只需要调用 child 的 didMoveToParentViewController: 方法. 这是因为 addChildViewController: 方法帮你调用了 child 的 willMoveToParentViewController: 方法. 而之所以需要你自己调用 didMoveToParentViewController: 的原因是这个方法必须要在 child's view 加入到 container's view 的层级 (hierarchy) 之后, 才能调用.

当使用自动布局时,  container 和 child 之间的约束设置应该在 child's root view 加入到 container's view 的层级 (hierarchy) 之后. 设置的约束应该只影响 child's root view 的尺寸和位置. 在 child's view 的层级中, 不要把 root view 和其他的 views 弄错了.

## Removing a Child View Controller 
要从 content 中移除一个 child view controller, 需要按照如下方式移除 view controllers 之间的父子关系:
1. 调用 child 的 willMoveToparentViewController: 方法, 参数传 nil;
2. 如果你为 child's root view 配置了约束, 全部移除;
3. 从 container's view 层级中移除 child's root view;
4. 调用 child 的 removeFromParentViewController 方法用于结束父子控制器之间的父子关系;

移除一个 child view controller 永久地切断了 parent 和 child 之间的联系. 只有确定以后不会再使用 child view controller 之后才能移除掉它. 举个栗子, 一个 navigation controller 在一个新的 child view controller 刚 push 到 navigation stack 上时, 不会移除它现有的 child view controllers. 它只会在子视图控制器从 navigation stack 中 pop off 后才会移除它们.

---

# Preserving and Restoring State （保留和恢复状态）

View controllers 在状态保存和状态恢复的过程中是至关重要的。 状态保存记录了在 app 被挂起之前的配置，这样一来，被挂起的 app 在启动之后便可以恢复配置。将 app 的配置恢复为之前的状态可以为用户节约时间，以带给用户更好的体验。

大多数的保存和恢复过程都是自动的，但是你需要告诉 iOS 需要保存 app 的哪部分。保存 app 的 view controllers 的步骤如下所示：
- （Required）将 restoration identifiers 分配给需要保存配置的 view controllers，详见[  Tagging View Controllers for Preservation ](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/PreservingandRestoringState.html#//apple_ref/doc/uid/TP40007457-CH28-SW2) ；
- （Required）告诉 iOS 该如何在 app 启动的时候创建和定位新的 view controller 对象，详见[  Restoring View Controllers at Launch Time ](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/PreservingandRestoringState.html#//apple_ref/doc/uid/TP40007457-CH28-SW5) ；
- （Optional）对于每个 view controller 来说，保存任何特定的配置数据需要将 view controller 复原到其原始的配置状态。详见 [ Encoding and Decoding Your View Controller’s State ](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/PreservingandRestoringState.html#//apple_ref/doc/uid/TP40007457-CH28-SW6) ；

如果需要看保存和恢复过程的概述，见 [ App Programming Guide for iOS. ](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007072)

## *Tagging View Controllers for Preservation*

UIKit 只会保存那些你允许保存的 view controllers。每一个 view controller 有一个 [ restorationIdentifier ](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621499-restorationidentifier)属性，默认值为 nil。设置这个属性为一个可用的字符串告诉 UIKit 这个 view controller 和它的 views 需要被保存。你可以在 stroryboard 中或者使用纯代码设置 restoration identifier。

在设置 restoration identifier 时，请记住把所有的处于 view controller 层级的 parent view controllers 的 restoration identifier 也一并设置。 在保存过程中，UIKit 从 window 的 root view controller 开始，然后沿着 view controller 的层级（ hierarchy ）走一遍。 如果在这一层级中有一个 view controller 没有 restoration identifier， 那么这个 view controller 和它的 child view controllers 以及 presented view controllers 都会被忽略。

### Choosing Effective Restoration Identifiers

UIKit 使用 restoration identifier 字符串重新创建 view controllers，所以最好选择使用对于你的代码来讲更容易辨认的字符串。 如果 UIKit 不能自动创建其中一个 view controller， 它会询问你让你创建一个， 为你提供 view controller 和它所有的 parent view controllers 的 restoration identifiers。 这个 identifiers 链表示 view controller 的 restoration path，而且 identifiers 链也是你决定哪个 view controller 应该被访问 （requested）的依据。Restoration path 从 root view controller 开始，它包括了到被访问的 view controller 其中所有的 view controllers （当然也包括被访问的 view controller）。

Restoration identifiers 通常是 view controller 的类名。 如果你在许多地方试用了同一个类， 那么你可能需要将其赋予一个更有意义的值。例如，你可能会赋予一个基于被 view controller 管理的 data 的字符串。。。

对于每一个 view controller 来说，restoration path 必须唯一。如果一个 container view controller 有两个 children，那么 container 必须给每一个 child 设置不同且唯一的 restoration identifier。一些 UIKit 中的 container view controllers 会自动不区分他们的 child view controllers，让你为每一个 child view controller 使用同一个 restoration identifier。举个栗子，UINavigationController 这个类会基于 child 在 navigation stack 中的位置为每一个 child 添加信息。如需获得关于某一个确切的 view controller 的更多信息，请参阅对应的 class reference。

### Excluding Groups of View Controllers
如果需要把一整组 view controllers 从 restoration process 中移除, 需要设置 parent view controller 的 restoration identifier 为 nil. 下图展示了将 restoration identifier 设置为 nil 对 view controller 层级所造成的影响. 缺少 preservation data 可以防止 view controller 接下来被恢复.

![Excluding view controllers from the automatic preservation process](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/state_vc_caveats_2x.png)

移除一个或多个 view controllers 并不会将它们从接下来的恢复中完全移除。在app启动阶段，任何 view controller，只要是 app 默认 setup 过程中的一部分，都依然会被创建，如下图所示。这种 view controllers 被重新创建时会使用它们默认的配置，**但它们依然会被创建**。

![Loading the default set of view controllers](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/state_vc_caveats_2_2x.png)


从自动保存过程中移除一个 view controller 并不能防止手动保存。在 restoration archive 中保存一个 view controller 的 reference 可以保存 view controller 的状态信息。举个栗子，前两张图的第一张，如果途中的 app delegate 保存了 navigation controller 的 3 个 children，那么它们的状态将会被保存。在恢复期间，app delegate 可以重新创建那几个 view controllers 然后将它们 push 到 navigation controller's stack 中。

### Preserving a View Controller's Views
一些 views 有一些额外的状态信息, 这些信息和 view 相关但是和 parent view controller 并无关联. 例如, 对于一个 scroll view, 你可能想保存它的 scroll position. 但是 view controller 只负责提供 scroll view 的内容( content ), 而 scroll view 的视觉状态信息则由 scroll view 自己负责.

为了保存一个 view 的状态, 需要做以下事情:
- 为 view 的 restorationIdentifier 属性设置一个可用的字符串;
- 拥有这个 view 的 view controller 也应设置一个可用的 restorationIdentifier;
- 对于 table views 和 collection views, 数据源需要遵守 [UIDataSourceModelAssociation](https://developer.apple.com/documentation/uikit/uidatasourcemodelassociation) 协议.

为 views 的 restoration identifier 赋值告诉了 UIKit 它应该在 preservation archive 记录下那个 view 的状态. 在 view controller 之后的恢复中, UIKit 也会恢复具有 restoration identifiers 的 views 的状态.

## *Restroing View Controllers at Launch Time*

在 app 启动期间，UIKit 试图将你的 app 恢复至之前的状态。在那时，UIKit 会询问 app 去创建（或者定位）包含被保存的用户界面的 view controller 对象。在定位 view controllers 的时候，UIKit 会按照以下顺序进行查找：
1. **如果 view controller 有一个 restoration class， UIKit 会要求这个 class 提供那个 view controller。** UIKit 会调用与 restoration class 关联的 *viewControllerWithRestorationIdentifierPath:coder:* 方法去获取 view controller。如果这个方法返回值为 nil，则假定 app 不想重新创建 view controller，然后 UIKit 会停止查找。
2. **如果 view controller 没有一个 restoration class，UIKit 会询问 app delegate 来提供那个 view controller。** UIKit 会调用 app delegate 的 application:viewControllerWithRestorationIdentifierPath:coder：方法去查找不包含 restoration class 的 view controller。如果这个方法的返回值为 nil，那么 UIKit 会隐式查找 view controller。
3. **如果已经存在一个有着正确的 restoration path 的 view controller，则 UIKit 会使用这个对象。** 如果 app 在启动阶段创建 view controllers（无论是通过代码还是 storyboard），而且那些 view controllers 有 restoration identifiers，UIKit 会根据他们的 restoration paths 隐式地查找。
4. **如果这个 view controller 是从 storyboard 加载的， UIKit 会使用已保存的 storyboard 信息去定位和创建它。** UIKit 将有关于 view controller 的 storyboard 的信息保存在了 restoration archive 中。在恢复期间，如果在使用其他方式后没有查找到这个 view controller，则 UIKit 会使用这些信息去定位相同的 storyboard 文件，然后实例化相应的 view controller。

为 view controller 设置 restoration class 可以防止 UIKit 隐式搜索那个 view controller。 使用 restoration class 可以让你对是否创建一个新的 view controller 有更多控制权。 例如，如果你的 class 决定不该重新创建那个 view controller，那么 viewControllerWithRestorationIdentifierPath:coder:方法可以返回 nil。如果当前没有 restoration class 存在时，UIKit 会为了查找，创建和恢复那个 view controller 做一切努力。

当使用 restoration class 时，viewControllerWithResorationIdentifierPath:coder：方法应该为这个类创建一个新的实例，执行最减短的实例化过程，然后返回结果对象。以下代码展示了你应该如何使用这个方法从 storyboard 加载一个 view controller。由于这个 view controller 是从 storyboard 加载的，这个方法使用了 UIStateRestorationViewControllerStoryboardKey 这个 key 去从 archive 获取 storyboard。注意，这个方法并不是试图要配置 view controller 层面的数据。这一步要等到 view controller 的状态被 decoded 之后才会触发。

```
// Creating a new view controller during restoration

+ (UIViewController*) viewControllerWithRestorationIdentifierPath:(NSArray *)identifierComponents
                      coder:(NSCoder *)coder {
   MyViewController* vc;
   UIStoryboard* sb = [coder decodeObjectForKey:UIStateRestorationViewControllerStoryboardKey];
   if (sb) {
      vc = (PushViewController*)[sb instantiateViewControllerWithIdentifier:@"MyViewController"];
      vc.restorationIdentifier = [identifierComponents lastObject];
      vc.restorationClass = [MyViewController class];
   }
    return vc;
}
```
对 restoration identifier 和 restoration class 重新赋值是一个好习惯，因为在重新手动创建
 view controllers 中很有效。恢复 restoration identifier 的最简单的方法是：取 identiferComponents 数组的 last item 然后把它赋值给你的 view controller。
 
对于在你的 app 启动时期，从 main storyboard 创建的对象们，请勿对每一个对象创建新的实例。让 UIKit 隐式查找那些对象或者使用 app delegate 的 application:viewControllerWithRestorationIdentifierPath:coder: 方法去查找现有的对象。

## *Encoding and Decoding Your View Controller's State*

对于每一个即将要保存的对象， UIKit 通过调用对象的 encodeRestorableStateWithCoder：方法给对象一个保存它状态的机会。在恢复期间，UIKit 调用对应的 decodeRestorableStateWithCoder：方法对对象进行解码然后将状态应用于对象上。这些方法的实现虽然是可选的，但却推荐这样做。你可能使用它们去保存和恢复以下类型的信息：
- 任何正在展示的数据的 reference （不是数据本身）；
- 对于一个 container view controller，它的 child view controllers 的 references；
- 有关当前选择的信息；
- 对于有用户配置的 view 的 view controllers，那个 view 的当前配置的相关信息；

在你的编码和解码方法中，你可以对对象和任意数据类型进行编码（encode），只要是 coder 支持的类型。对于所有的对象，除了 views 和 view controllers，这个对象必须遵守 NSCoding 协议而且实现了协议中状态写入的方法。对于 views 和 view controllers，coder 不会使用 NSCoding 协议去保存对象的状态。取而代之，coder 保存对象的 restoration identifier，然后将其添加到可保存对象的列表中，而这个列表会导致对象的 encodeRestorableStateWithCoder：方法被调用。

View controllers 的 encodeRestorableStateWithCoder: 和 decodeRestorableStateWithCoder: 方法必须在实现中调用 super。调用 super 让 parent class 由一个能够存储和恢复任何额外信息的机会. 下面的代码展示了实现这些方法的例子, 例子中保存了一个可以确定特定的 view controller 的一个数值.

```
// Encoding and decoding a view controller’s state

- (void)encodeRestorableStateWithCoder:(NSCoder *)coder {
   [super encodeRestorableStateWithCoder:coder];
 
   [coder encodeInt:self.number forKey:MyViewControllerNumber];
}
 
- (void)decodeRestorableStateWithCoder:(NSCoder *)coder {
   [super decodeRestorableStateWithCoder:coder];
 
   self.number = [coder decodeIntForKey:MyViewControllerNumber];
}
```

Coder 对象在 encode 和 decode 的过程中并不是共享的. 每一个具备可保存状态的对象都有属于它自己的 coder 对象. 对于特定的 coders 的使用意味着你不用担心 keys 在命名空间之间的冲突. 但是, 不要用以下几个 keys: UIApplicationStateRestorationBundleVersionKey, UIApplicationStateRestorationUserInterfaceIdiomKey, UIStateRestorationViewControllerStoryboardKey. 这些 keys 是被 UIKit 用来保存 有关 view controller 状态的额外信息的.

如果想要知道更多关于 view controller 实现 encode 和 decode 的信息, 见 [UIViewController Class Reference](https://developer.apple.com/documentation/uikit/uiviewcontroller).

## *Tips for Saving and Restoring Your View Controllers*
如果你想为你的 view controllers 的状态保存添加更多支持的话, 请考虑使用以下的 tips 作为指导:
- **记住, 你可能不想保存所有的 view controllers.** 在某些情况, 保存 view controller 是不合理的. 例如, 如果 app 正在展示一个变化, 你可能想要取消这个操作然后将 app 恢复至变化之前的样子. 在这种情况下, 你不应该保存那些需要重新输入新密码的 view controller.
- **在恢复期间, 避免交换 view controller 的类.** 状态保存系统 encode 了需要保存的 view controller 的类. 在恢复期间, 如果 app 返回了一个对象, 而这个对象的类与原始对象的类不一样(或与子类也不一样), 那么系统不会让 view controller 去 decode 任何状态信息. 因此, 交换了 view controller 之后的类并不会恢复所有的状态信息.
- **状态保存系统希望你像预期的那样使用 view controllers.** 在恢复过程中重新建立界面依赖于 view controllers 的包含关系. 如果你不正确地使用 container view controllers, 那么保存系统就不能为你找到 view controllers. 举个例子, 请勿将 view controller 的 view 嵌入一个不同的 view, 除非他们的 view controllers 之间有包含关系!


---

# Presenting a View Controller
有两种方法可以将一个 view controller 展示到屏幕上: 将其嵌入一个 container view controller 或者 present 它. Container view controllers 为 app 提供最基本的导航, 但是 present view controllers 也是一种重要的导航工具. 你可以使用 present 的方式直接将一个新的 view controller 展示在最上层. 一般来说, 当你想要实现一个 modal 的界面时, 你使用 present 方式把 view controller 展现出来, 不过你也可以用作别的用途.

UIViewController 类本身内建了对 present 的支持. 你可以从任何 view controller present 其它的 view controller, 尽管 UIKit 可能将访问导向一个不同的 view controller. Presenting
一个 view controller 在原本的 view controller( 也叫做 presneting view controller )和新的 view controller( 也叫做 presented view controller ) 之间创建了一种关系. 这种关系组成了 view controller 层级的一部分, 而且这种关系存留着直到 presented view controller 被 dismissed.

## *The Presentation and Transition Process*
使用 present view controller 的方式来让一个新的内容以动画的形式出现在屏幕上是非常快捷高效的. Presentation 机制是内建在 UIKit 里面的, 它让你使用原生的或者自定义的动画来展示一个新的 view controller. 原生的 presentation 和 animation 只需要非常少量的代码, 因为 UIKit 为你做了大部分的工作. 你也可以创建自定义的 presentation 和 animation, 这只需要付出很少的努力就可以让它们应用在你的 view controllers 当中.

你可以使用代码或者 segues 来为 view controller 添加 presentation. 如果在设计阶段就知道 app 的 navigation, 那么 segue 是添加 presentation 最好的方式. 对于更多动态的界面来说, 或者那些没有直接可以使用 segue 的情况下, 建议使用 UIViewController 的方法去 present 你的 view controllers.

### Presentation Styles
一个 view controller 的 presentation style 管理着 view controller 在屏幕上的展现. UIKit 定义了许多标准的 presentation styles, 每一个都有着其独特的展示方式和目的. 你也可以自定义 presentation styles. 当你设计你的 app 时, 选择那些看起来最适合的 presentation style 然后修改 presented view controller 的 modalPresentationStyle 属性的值.

#### Full-Screen Presentation Styles
Full screen presentation styles 覆盖整个屏幕, 并且可以防止与下方的内容进行交互. 在水平的 regular 场景中,  只有一种可以完整地覆盖全屏. 其余的会在两个 contollers 之间加入一层暗淡的 view 或者干脆是透明的, 这样可以展示下方 view controller 的视图的一部分. 在水平 compact 场景中, full-screen presentation 自动采用 UIModalPresentationFullScreen 方式去完全覆盖下方的内容.

下图展示了在水平 regular 场景下, presentation 使用的 3 种不同 styles 的表现. 图中, 绿色的 view controller 是 presenting view controller, 蓝色的是 presented view controller. 对于一些 presentation styles, UIKIT 在两个 view controllers 之间会插入一张暗色视图.

![The full screen presentation styles](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_PresentationStyles%20_fig_8-1_2x.png)
> 注意:  
当使用 UIModalPresentationFullScreen 方式去 present 一个 view controller 时, UIKit 通常会在转场动画结束后移除下方的 view controller. 你可以通过设置 style 为 UIModalPresentationOverFullScreen 来防止下方的视图被移除. 可能会在 presented view controller 有一个透明的区域来让下方的内容透过进行展示的时候用上这招.

在使用 full-screen presentation styles 时, 发起 presentation 的 view controller 本身必须覆盖整个屏幕. 如果 presenting view controller 不覆盖全屏, UIKit 会沿着 view controller 层级一直向上找, 直到找到为止. 如果到最后都没有找到的话, 那么 UIKit 会使用 window 的 root view controlelr.

#### The Popover Style
UIModalPresentationPopover style 在 view controller 上展示了一个 popover view. Popover 是一种展示与选中物体有关的额外信息或者是 items 列表的有效手段. 在水平 regular 场景中, popover view 仅仅覆盖了屏幕的一小部分, 如下图所示. 在水平 compact 场景中, popover 默认采用 UIModalPresentationOverFullScreen 这种方式. 在 popview 之外进行点击会自动让 popview 消失.

![The popover presentation style](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_popover-style_2x.png)

因为在水平 compact 场景中, popover 默认采用 full-screen presentation, 那么为了让 popover 适配, 你需要经常修改 popover 的代码. 在全屏模式中, 你需要一个让 popover 消失的方法. 你可以添加一个按钮, 把 popover 嵌入在一个可以消失的 container view controller 中, 或者让其自己改变适配方式.

如果需要更多配置 popover presentation 的 tips, 详见: [ Presenting a View Controller in a Popover](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/PresentingaViewController.html#//apple_ref/doc/uid/TP40007457-CH14-SW13).

#### The Current Context Styles
使用 UIModalPresentationCurrentContext 会将 presented view controller 覆盖在一个指定的 view controller 上. 当使用 contextual style 时, 你可以把你想要覆盖的 view controller 的 definesPresentationContext 属性设置为 YES. 下图展示了一个 在 split view controller 中只覆盖一个子视图控制器的 current context presentation.

![The current context presentation style](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_CurrentContextStyles_2x.png)

> 注意:  
当使用 UIModalPresentationFullScreen 方式去 present 一个 view controller, UIKit 通常在转场动画完成后会把下方的视图移除, 可以使用 UIModalPresentationOverCurrentContext 方式防止其被移除.

定义了 presentation context 的 view controller 也可以定义 presentation 时的转场动画. 通常, UIKit 通常按照 presented view controller 的 modalTransitionStyle 属性的值去为 view controllers 添加动画. 如果 presentation context view controller 把它的 providesPresentationContextTransitionStyle 属性设置为 YES, UIKit 则会使用那个 view controller 的 modalTransitionStyle 属性的值.

当转场至水平 compact 场景时, current context styles 会自适应为 UIModalPresentationFullScreen. 如果要改变效果, 使用合适的 presentation 代理去为 view controller 指定一个不同的 presentation.

#### Custom Presentation Styles
UIModalPresentationCustom 允许你使用自定义的方式去 present 一个 view controller. 详情请参见: [Creating Custom Presentations](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/DefiningCustomPresentations.html#//apple_ref/doc/uid/TP40007457-CH25-SW1)


### Transition Styles
Transition styles 决定了用于展现一个 presented view controller 的动画. 对于原生的 transition styles, 你可以设置 view controller 的 modalTransitionStyle 属性的值来决定该如何 present. 当你 present 这个 view controller 时, UIKit 会创建对应那个 style 的动画. 举例, 下图展示了一个标准的上滑( UIModalTransitionStyleCoverVertical )转场动画. View controller B 从屏幕外开始, 然后向上滑动直到覆盖 view controller A. 当 view controller B 退场, 动画反转, B 下滑然后展现 A.

![A transition animation for a view controller](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_SlideTransition_fig_8-1_2x.png)

你可以通过使用 animator object 和 transitioning delegate 创建自定义的转场动画. Animator object 可以创建时 view controller 呈现在屏幕上的转场动画. Transitioning delegate 为 UIKit 在合适的时间提供那个 animator object. 想要知道更多关于实现自定义转场动画, 请移步至: [Customizing the Transition Animations](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/CustomizingtheTransitionAnimations.html#//apple_ref/doc/uid/TP40007457-CH16-SW1).

### Presenting Versus Showing a View Controller

UIViewController 类提供了两种展示 view controller 的途径:
- showViewController:sender: 和 showDetailViewController:sender: 方法提供了最具适应性和弹性的展示 view controllers 的方式. 这些方法允许 Presenting view controller 决定应该如何使用最佳方案处理 presentation. 举个栗子, 一个 container view controller 可能要引入一个 view controller 作为 child 而不是 modally present 出来. 而默认的 present 是 modal 的.
- presentViewController:animated:completion: 方法通常 modally present 一个 view controlelr. 而调用这个方法的 view controller 最终并不想处理这个 presentation 但是这些 presentation 总是 modal 出来的. 这个方法可适用于水平 compact 场景的 presentation style.

showViewController:sender: 和 showDetailViewController:sender: 方法是发起 presentations 的推荐方法. 一个 view controller 可以任意调用他们, 而不用知道其它 view controller 的层级或者当前 view controller 在层级中的位置. 这些方法也可以使 view controller 在 app 的不同部分的重用更加轻松, 而不用写条件路径.

## *Presenting a View Controller*
为一个 view controller 启动 presentation 有若干种方法:
- 使用 segue 来自动 present. Segue 使用 Interface Builder 中的信息实例化了 presented view controller 然后 present 这个 view controller. 想知道更多关于配置 segues 的信息, 请移步至: [Using Segues](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/UsingSegues.html#//apple_ref/doc/uid/TP40007457-CH15-SW1).
- 使用 showViewController:sender: 或者 showDetailViewController:sender:方法去展示一个 view controller. 在自定义的 view controllers 中, 你可以使用这些方法去改变 present 的行为以更适合你的 view controller.
- 调用 presentViewController:animated:completion: 方法 modally present view controller.

### Showing View Controllers
当使用 showViewController:sender: 和 showDetailViewController:sender: 方法时, 在屏幕上展示一个新的 view controller 的过程是很简单的:
1. 创建你想要 present 的 view controller 对象. 当创建这个 view controller 时, 你需要使用数据初始化它, 以方便它完成任务.
2. 把新的 view controller 的 modalPresentationStyle 属性设置为合适的值. 不过这个 style 可能不是最终 presentation 使用的值.
3. 把新的 view controller 的 modalTransitionStyle 属性设置为需要的值. 不过这个 style 可能不是最终 presentation 使用的值.
4. 调用当前 view controller 的 showViewController:sender: 和 showDetailViewController:sender: 方法.

UIKit 转发 showViewController:sender: 和 showDetailViewController:sender: 这两个方法至合适的 presenting view controller. 于是, 接下来, view controller 可以决定该如何把 presentation 做到最好, 还有可以根据续期改变 presentation 和 transition styles. 举个栗子, 一个导航控制器可能需要 push 一个 view controller 到它的 navigation stack 的栈顶.

如果需要知道更多关于展现 view controller 和 modally present view controller 的区别, 请查看: [Presenting Versus Showing a View Controller](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/PresentingaViewController.html#//apple_ref/doc/uid/TP40007457-CH14-SW4).

### Presenting View Controllers Modally
当直接 present 一个 view controller 的时候, 你需要告诉 UIKit 应该怎样展示和为 新的 view controller 添加动画.
1. 创建一个你需要 present 的 view controller 对象.  
在创建 view controller 的过程中, 你需要使用数据初始化它, 以方便它完成任务.
2. 把新的 view controller 的 modalPresentationStyle 属性设置为合适的值. 不过这个 style 可能不是最终 presentation 使用的值.
3. 把新的 view controller 的 modalTransitionStyle 属性设置为需要的值. 不过这个 style 可能不是最终 presentation 使用的值.
4. 调用当前 view controller 的 presentViewController:animated:completion: 方法.

调用 presentViewController:animated:completion: 方法的 view controller 有可能不是最后真正执行 modal presentation 的那个控制器. Presentation style 决定了那个 view controller 是如何被 presented 出来的, 包括 presenting view controller 需要的特征. 如果当前的 presenting view controller 不适合执行 present, 那么 UIKit 会沿着 view controller 的层级向上走直到找到适合执行的. 在 modal presentation 完成的时候, UIKit 会更新相关的 view controller 的 presentingViewController 和 presentedViewController 属性.

下面的代码演示了如何使用代码去 present 一个 view controller. 当用户新添加了一个新的 recipe, app 会通过 present 一个 navigation controller 来提示用户一些关于 recipe 的基本信息. 之所以要选择 navigation controller 是因为它具有摆放 Cancel 和 Done 按钮的标准位置. 另外, 使用 navigation  controller 在将来也可以在界面上更方便地扩充新的 recipe. 你所需要做的只是在 navigation stack 上 push 新的 view controllers.

```
// Presenting a view controller programmatically

- (void)add:(id)sender {
   // Create the root view controller for the navigation controller
   // The new view controller configures a Cancel and Done button for the
   // navigation bar.
   RecipeAddViewController *addController = [[RecipeAddViewController alloc] init];
 
   addController.modalPresentationStyle = UIModalPresentationFullScreen;
   addController.transitionStyle = UIModalTransitionStyleCoverVertical;
   [self presentViewController:addController animated:YES completion: nil];
}
```

### Presenting a View controller in a Popover
在 present popovers 之前需要一些额外的配置工作. 在你将 modal presentation style 设置为 UIModalPresentationPopover 之后, 你还需要配置下面与 popover 有关的属性:
- 将你的 view controller 的 preferredContentSize 属性设置为合适的 size;
- 使用 UIPopoverPresentationController 对象来设置 popover anchor point, 这个对象可以通过 view controller 的 popoverPresentationController 属性获得. 只需要设置一下其中一个即可:
    - 为一个 bar button item 设置  [barButtonItem](https://developer.apple.com/documentation/uikit/uipopoverpresentationcontroller/1622314-barbuttonitem) 属性;
    - 在你众多的 views 其中的一个设置 [sourceView](https://developer.apple.com/documentation/uikit/uipopoverpresentationcontroller/1622313-sourceview) 和 [sourceRect](https://developer.apple.com/documentation/uikit/uipopoverpresentationcontroller/1622324-sourcerect) 属性.

你可以使用 UIPopoverPresentationController 对象根据需要来设置 popover 的表现. Popover presentation controller 也支持设置一个代理来响应 popover 的 appear, disappear 或者是在屏幕上位置的重置. 如果想要知道更多关于这个对象的信息, 参见: [UIPopoverPresentationController Class Reference](https://developer.apple.com/documentation/uikit/uipopoverpresentationcontroller)

## *Dismissing a Presented View Controller*
Presenting view controller 调用 dismissViewControllerAnimated:completion: 方法可以 dismiss presented view controller. 你也可以使用 presented view controller 自己来调用这个方法. 当你使用 presented view controller 调用这个方法时, UIKit 会自动将这个方法转发至 presenting view controller 执行.

务必记住, 在 dismiss 一个 view controller 之前, 一定要保存好重要的信息. Dismiss 一个 view controller 会将它从 view controller 层级中移除然后把它的 view 从屏幕移除. 如果你不在其他地方强引用这个 view controller, 那么 dimiss 会释放与之相关的内存.

如果 presented view controller 必须对 presenting view controller 返回数据, 使用代理的设计模式去处理这种数据转移. 代理可以使得在 app 的任意地方复用这个 view controller 变得更加方便. 有了代理, 这个 presented view controller 保存了对代理对象的引用, 而代理对象也会实现正式协议的方法. 在通常情况下, presenting view controller 将自己设置为 presented view controller 的代理.

## *Presenting a View Controller Defined in a Different Storyboard*
虽然你可以自己在同一个 storyboard 中的不同 view controllers 之间创建 segues, 但是你不能在不同的 storyboards 之间创建. 当你想要展示不同 storyboard 中的一个 view controller 时, 你必须在 present 那个 view controller 之前显式地实例化它, 就像下面代码写的那样. 例子中 modally present 了 view controller, 但是你可以使用 navigation controller 用 push 的方式, 或者其它的方式将它呈现出来.

```
// Loading a view controller from a storyboard

UIStoryboard* sb = [UIStoryboard storyboardWithName:@"SecondStoryboard" bundle:nil];
MyViewController* myVC = [sb instantiateViewControllerWithIdentifier:@"MyViewController"];
 
// Configure the view controller.
 
// Display the view controller
[self presentViewController:myVC animated:YES completion:nil];
```

没有说必须要在 app 中创建多个 storyboards. 这里有几个例子, 在这些例子所在的情形下, 可能需要创建多个 storyboards :
- 你有一个大型的开发团队, 团队分成多个小组, 每个小组负责管理不同部分的用户界面. 每个小组使用自己部分的 storyboard 文件可以减小冲突.
- 你购买或者创建了一个 library, 这个库预定义了一些 view controller 的类型; 那些 view controllers 的内容在 library 提供的 storyboard 中被定义.
- 你需要在外置显示器展示一些内容. 在这种情况下, 你可能需要将所有需要在外部显示器显示的相关的 view controllers 放在一个单独的 storyboard 文件中. 另外一种做法则是写一个自定义的 segue.

---

# Using Segues

使用 segues 去定义你 app 的界面流. 在你 app storyboard 文件中, Segue 定义了两个 view controllers 之间的转场( transition ). Segue 可以从 button, table row, 或者初始化了 segue 的 gesture recognizer 开始. 而终点则是那个你想要呈现的 view controller. Segue 通常用来 present 一个新的 view controller, 但是你也可以使用 unwind segue 来 dismiss 一个 view controller.

![image](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/segue_defined_2x.png)

你并不需要使用程序去触发 segue. 在运行时阶段, UIKit 加载与 view controller 有关的 segue 并且将它们与对应的元素进行连接. 当用户与该元素进行交互时, UIKit 会加载合适的 view controller, 通知你的 app 这个 segue 即将要被触发, 然后执行 transition. 你可以通过 UIKit 发送通知来给这个新的 view controller 传递数据, 或者完全阻止这个 segue 的发生.

## *Creating a Segue Between View Controllers*

欲在同一个 storyboard 的 view controllers 之间创建一个 segue, 按住 control 然后点击合适的元素, 然后拖动鼠标至新的 view controller. segue 的起始点必须是一个 view 或者是一个有着预定义动作的对象, 比如一个 control, bar button item, 或者是 gesture recognizer. 你也可以通过 cell-based views 来创建, 比如 tables 和 collection views. 下图展示了 segue 的创建和展示, 它能够在一个 table row 被点击的时候展示一个新的 view controller.

![Creating the segue relationship](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/segue_creating_2x.png)

> 注意:  
一些元素支持多个 segue. 举个栗子, table row 允许你为不同的点击配置不同的 segue. 比如一个是点击 row 的, 另外一个是点击 row 里面的 accessory button 的.

当你放开鼠标按键的时候, Interface Builder 会提示你, 让你去选择关系种类, 这个关系是指你希望在两个 view controllers 之间建立起的关系, 如下图所示. 可以选择 segue 来对应你希望要的 transition 方式.

![Selecting the type of segue to create](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/segue_creating_relationships_2x.png)

当你想为 segue 选择关系种类, 那么就应该尽可能选择适合的. 合适的 segues 会根据当前所处的环境自动调整他们的行为. 举例, "Show" segue 会根据 presenting view controller 来改变它的行为. 非适应性的( nonadaptive ) segues 只支持在 iOS 7 上运行 app, 因为 iOS 7 不支持可适应性( adaptive ) segue. 下面列出了可适应性 segues 以及他们在 app 中是如何表现的:
- ***Show(Push)***:  
这个 segue 会调用目标控制器的 showViewController:sender: 方法去展示新的内容. 对于大多数 view controllers 来说, 这个 segue 会在源控制器上 modally present 新的内容. 一些 view controllers 明确重写了方法然后实现不同的行为. 举例, navigation controller push 新的 view controller 到它的 navigation stack 上. UIKit 使用 targetViewControllerForAction:sender: 方法去定位源控制器( source view controller ).
- ***Show Detail (Replace)***:  
这个 segue 会调用目标控制器的 showViewController:sender: 方法去展示新的内容. 这个 segue 只会与那些嵌入在 [UISplitViewController](https://developer.apple.com/documentation/uikit/uisplitviewcontroller) 对象中的 view controllers 有关. 有了这个 segue, split view cntroller 可以将它的第二个 child view controller( the detail controller )使用新的内容替换掉. 大多数其他的 view controllers 只会 modally present 新的内容. UIKit 使用 targetViewControllerForAction:sender: 方法去定位源控制器( source view controller ).
- ***Present Modally***:  
这个 segue 可以使用指定的 presentation 和 transition styles 来 modally 展示 view controller. 这个 view controller 定义了合适的 presentation context 来处理实际的 presentation.
- ***Present as Popover***:  
在水平 regular 场景中, view controller 以 popover 的形式呈现. 在水平 compact 场景中, view controller 以全屏的 modal presentation 的方式呈现.

在创建 segue 之后, 选择一个 segue 对象然后使用 '属性观察器( attributes inspector )' 来修改 identifier. 你可以使用 identifier 决定该触发那一个 segue, 这对于支持多个 segues 的 view controller 来说是非常有用的. identifier 被包含在 [UIStoryboardSegue](https://developer.apple.com/documentation/uikit/uistoryboardsegue) 对象中, 这个对象当 segue 执行的时候会递交给你的 view controller.

## *Modifying a Segue’s Behavior at Runtime*

下图展示了当一个 segue 触发时, 究竟发生了什么. 大多数工作发生在 presenting view controller 中, 这个控制器管理着新的 view controller 的 transition. 对新的 view controller 的配置本质上还是跟随着同样的步骤, 这个步骤就是当你创建一个新的 view controller 然后 present 它. 由于 segues 的配置是在 storyboards 中的, 所以 segue 连接着的两个控制器都应该在同一个 storyboard 中.

![Displaying a view controller using a segue](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_displaying-view-controller-using-segue_9-4_2x.png)

在 segue 过程中, UIKit 会调用当前的 view controller 的一些方法, 这让你有机会去影响 segue 后的结果.
- shouldPerformSegueWithIdentifier:sender: 方法给你一个机会去防止 segue 的发生. 返回 NO 可以让 segue 过程静悄悄地失败, 但是却不能阻止其它的动作. 举例, 在 talbe row 中的 tap 仍然可以让 table 去调用代理方法.
- source view controller 的 prepareForSegue:sender: 方法允许你从 source view controller 向 destination view controller 传递数据. 传递到这个方法的参数 —— UIStoryboardSegue 对象持有了一个对 destination view controller 的引用以及其他与 segue 有关的信息.

## *Creating an Unwind Segue*
Unwind segues 允许你 dismiss 一个已经被 present 出来的 view controller. 你可以在 Interface Builder 中通过连接一个按钮或者其它合适的对象至当前 view controller 的 Exit 对象来创建 unwind segues. 当用户点击了 button 或者在与合适的对象进行交互时, UIKit 会在 view controller 的层级当中查找能够处理 unwind segue 的对象. 然后这个对象会 dismiss 当前的 view controller 和任何中间的 view controllers 然后呈现 unwind segue 的 target.

创建一个 unwind segue 的步骤:
1. 选择 unwind segue 执行结束时应该展现在屏幕上的 view controller.
2. 在你选择的 view controller 上定义一个 unwind action.  
    The Swift syntax for this method is as follows:
    ```
    @IBAction func myUnwindAction(unwindSegue: UIStoryboardSegue)
    ```  
    The Objective-C syntax for this method is as follows:
    ```
    - (IBAction)myUnwindAction:(UIStoryboardSegue*)unwindSegue
    ```
3. 找到你想初始化 unwind action 的 view controller.
4. 按住 Contorl 并点击那个需要初始化 unwind segue 的按钮或者对象. 这个元素应该在你想要 dismiss 的 view controller 里.
5. 把它拖到 Exit 对象中.  
![image](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/segue_unwind_linking_2x.png)
6. 在关系面板选择你需要的 unwind action.

在 Interface Builder 中创建对应的 unwind segue 之前, 你必须在其中一个 view controller 中定义一个 unwind action 方法. 这个方法必须存在, 因为这样可以告诉 Interface Builder 对于 unwind segue 已经有一个可用的 target.

使用你的 unwind action 方法去实现你的 app 独有的工作. 你不需要把包含 segue 本身的 view controllers 给 dismiss 掉, 因为 UIKit 会帮你做. 你应该使用 segue 对象去获取正在被 dismiss 的 view controller, 这样你就可以从它那获取数据. 你也可以使用 unwind action 在 unwind segue 结束前更新当前 view controller.

## *Initiating a Segue Programmatically*

由于你在 storyboard 文件中创建了连接, segues 会被经常触发. 然而, 在有些情况下, 你不能在 storyboard 中创建 segues, 可能是因为 destination view controller 还是未知的. 举栗子, 一个 游戏 app 可能要根据游戏的结果转场到不同的场景. 在那些情况下, 你可以使用代码触发 segues -- 调用当前视图控制器的 performSegueWithIdentifier:sender: 方法.

下面的代码展示了当设备从 portrait 旋转到 landscape 的状态下, 会 present 一个指定的 view controller 的 segue. 由于通知对象在这个情形中没有为 segue 的执行提供任何有用的信息, 于是这个 view controller 指定它自己为 segue 的 sender.

```
// Triggering a segue programmatically

- (void)orientationChanged:(NSNotification *)notification {
    UIDeviceOrientation deviceOrientation = [UIDevice currentDevice].orientation;
    if (UIDeviceOrientationIsLandscape(deviceOrientation) &&
             !isShowingLandscapeView) {
        [self performSegueWithIdentifier:@"DisplayAlternateView" sender:self];
        isShowingLandscapeView = YES;
    }
// Remainder of example omitted.
}
```

## *Creating a Custom Segue*
Interface Builder 为所有从一个 view controller 转场至另外一个 view controller 的标准方法提供了 segues -- 无论是 present view controller 还是通过 popover 的形式展示 controller. 然而, 如果那些 segues 不能满足你的需求, 你也可以自定义 segue.

### The Life Cycle of a Segue
为了理解自定义 segues 是如何工作的, 你需要理解 segue 对象的生命周期. Segue 对象是 [UIStoryboardSegue](https://developer.apple.com/documentation/uikit/uistoryboardsegue) 类或其子类的实例. 你的 app 绝不应该直接创建 segue 对象; UIKit 会在 segue 触发的时候创建他们. 下面描述了事情的经过...:
1. 创建以及初始化需要被 present 的 view controller；
2. 创建 segue 对象，调用 initWithIdentifier:source:destination: 方法。其中的 identifier 是你在 Interface Builder 中给的特定字符串。其他的两个参数表示参与 Transition 的两个 view controller 对象；
3. 调用 presenting view controller 的 prepareForSegue:sender: 方法。 详见[ Modifying a Segue’s Behavior at Runtime](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/UsingSegues.html#//apple_ref/doc/uid/TP40007457-CH15-SW11)
4. 调用 segue 对象的 [perform](https://developer.apple.com/documentation/uikit/uistoryboardsegue/1621912-perform) 方法。这个方法会执行 transition 然后把新的 view controller 展示在屏幕上；
5. 指向 segue 对象的引用被释放；

### Implementing a Custom Segue
要实现一个自定义的 segue，需要继承自 UIStoryboardSegue 然后实现以下方法：
- 重写 initWithIdentifier:source:destination: 方法，用它去初始化你的 segue 对象。永远记得要先调用 super。
- 实现 perform 方法去配置 transition 动画；

> 注意：  
如果你的实现为 segue 增加了属性，但是你不能在 Interface Builder 去配置这些属性。作为替代，你只能使用代码来配置。你需要在 source view controller （也就是要出发 segue 的那个控制器）的 prepareForSegue:sender: 方法中配置那些额外属性。

下面的代码实现了一个非常简单的自定义 segue。这个例子仅仅展示了没有任何动画的 present，但是如果需要的话，你可以把你自己的动画加入进来。

```
// A custom segue

- (void)perform {
    // Add your own animation code here.
 
    [[self sourceViewController] presentViewController:[self destinationViewController] animated:NO completion:nil];
}
```
----

# Customizing the Transition Animations
转场动画为用户提供 app 界面变化的反馈. UIKit 提供了在 present view controller 时的 一系列标准的转场方式, 而且你也可以使用自定义的转场动画.

## *The Transition Animation Sequence*
转场动画交换了两个 view controller 的内容. transition 有两种: 呈现和消失( presentation and dismissal ). 一个 presentation transition 在你 app 的 view controller 层级添加了一个新的 view controller, 相反地, dismissal transition 从你的 view controller 层级移除一个或者更多的 view controller.

实现一个转场动画需要用到的对象有很多. UIKit 提供了默认需要使用的所有对象, 当然你也可以在自定义的时候使用全部或者只是一个子集. 如果你选择了正确的对象, 你应该只需要使用少量代码便可以创造出属于你的动画效果. 就算你的动画包括一些交互, 你仍然可以只使用 UIKit 提供的一些现成代码就能够轻易实现.

## The Transitioning Delegate
转场代理是转场动画和自定义 presentation 的开始. 转场代理是一个对象, 你可以自己定义这个对象但是必须遵守 [UIViewControllerTransitioningDelegate](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioningdelegate) 协议. 它的任务是为 UIKit 提供以下几个对象:
- **动画执行对象( Animator objects ).** 动画执行对象负责创建动画, 这个动画用来呈现或者隐藏 view controller 的 view. 转场代理可以提供单独的动画执行对象来 present 或者 dismiss view controller. 动画执行对象遵守 [UIViewControllerAnimatedTransitioning](https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning) 协议.
- **交互动画执行对象( Interactive animator objects ).** 交互动画执行对象根据点击事件或者 gesture recognizers 来控制自定义动画的时机. 交互动画执行对象遵守 [UIViewControllerInteractiveTransitioning](https://developer.apple.com/documentation/uikit/uiviewcontrollerinteractivetransitioning) 协议.  
最简单的创建交互动画执行对象的方式是继承 [UIPercentDrivenInteractiveTransition](https://developer.apple.com/documentation/uikit/uipercentdriveninteractivetransition) 类, 然后在子类中添加事件处理的代码. 这个类使用你现有的动画执行对象来控制动画的执行时机. 如果你创建了你自己的交互动画执行对象, 那么你必须自己去渲染每一帧动画.
- **Presentation Controller.** Presentation controller 管理着 view controller 呈现时的 presentation style. 系统提供了 presentation controller 内置的 presentation styles, 不过你可以为你自己的 presentation style 提供自定义的 presentation controller. 想知道更多关于如何创建一个自定义的 presentation controller, 见: [Creating Custom Presentations](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/DefiningCustomPresentations.html#//apple_ref/doc/uid/TP40007457-CH25-SW1)

将一个转场代理赋值给 view controller 的 [transitioningDelegate](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621421-transitioningdelegate) 属性告诉 UIKit 你想要执行一个自定义的 transition 或者 presentation. 你的代理应该可以选择提供的对象. 如果你不提供动画执行对象( Animator objects ), UIKit 会在 view controller 的 [modalTransitionStyle](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621388-modaltransitionstyle) 属性中使用标准的转场动画.

下图展示了转场代理和动画执行对象对于 presented view controller 的关系. Presentation controller 仅仅在 view controller 的 [modalTransitionStyle](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621388-modaltransitionstyle) 属性设置为 [UIModalPresentationCustom](https://developer.apple.com/documentation/uikit/uimodalpresentationstyle/1621375-custom) 的时候会被使用到.

![The custom presentation and animator objects](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_custom-presentation-and-animator-objects_10-1_2x.png)

想要得到更多关于如何实现转场代理的信息, 见 [Implementing the Transitioning Delegate](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/CustomizingtheTransitionAnimations.html#//apple_ref/doc/uid/TP40007457-CH16-SW15). 更多关于转场代理对象的方法, 见: [UIViewControllerTransitioningDelegate Protocol Reference](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioningdelegate).

## The Custom Animation Sequence
当一个 presented view controller 的 [transitioningDelegate](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621421-transitioningdelegate) 属性包含一个可用的对象, UIKit 会使用你提供的自定义动画执行对象来 present 这个 view controller. 当它准备 presentation 的时候, UIKit会调用转场代理的 [animationControllerForPresentedController:presentingController:sourceController:](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioningdelegate/1622037-animationcontrollerforpresentedc) 方法来获取自定义动画执行对象. 如果对象可用, 那么 UIKit 会执行以下几个步骤:
1. UIKit 调用转场代理的 [interactionControllerForPresentation:](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioningdelegate/1622050-interactioncontrollerforpresenta) 方法来检查交互动画执行对象是否可用. 如果这个方法返回了 nil, 那么 UIKit 会执行不带任何用户交互的动画.
2. UIKit 调用动画执行对象的 [transitionDuration:](https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning/1622032-transitionduration) 方法来获取动画的持续时间.
3. UIKit 调用合适的方法来开始动画:
    - 对于无交互的动画, UIKit 调用动画执行对象的 [animateTransition:](https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning/1622061-animatetransition) 方法.
    - 对于有交互的动画, UIKit 调用交互动画执行对象的 [startInteractiveTransition:](https://developer.apple.com/documentation/uikit/uiviewcontrollerinteractivetransitioning/1622028-startinteractivetransition) 方法.
4. UIKit 等待一个动画执行对象调用上下文转场对象的 [completeTransition:](https://developer.apple.com/documentation/uikit/uiviewcontrollercontexttransitioning/1622042-completetransition) 方法.  
    你的自定义动画执行对象在多花结束后会调用这个方法, 一般来说会在动画结束的 completion block 中调用. 调用这个方法会结束动画并且让 UIKit 知道它可以调用 [presentViewController:animated:completion:](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621380-presentviewcontroller) 方法的 completion handler, 然后调用动画执行对象自己的 [animationEnded:](https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning/1622059-animationended) 方法.

当 dismiss 一个 view controller 时, UIKit 调用转场代理的 [animationControllerForDismissedController:](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioningdelegate/1622047-animationcontroller) 方法然后执行下列步骤:
1. UIKit 调用转场代理的 [interactionControllerForDismissal:](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioningdelegate/1622030-interactioncontrollerfordismissa) 方法来查看是否有可用的交互动画执行对象. 如果这个方法返回 nil, 那么 UIKit 会执行不带任何用户交互的动画.
2. UIKit 调用动画执行对象的 [transitionDuration:](https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning/1622032-transitionduration) 方法来获取动画的持续时间.
3. UIKit 调用合适的方法来开始动画:
    - 对于无交互的动画, UIKit 调用动画执行对象的 [animateTransition:](https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning/1622061-animatetransition) 方法.
    - 对于有交互的动画, UIKit 调用交互动画执行对象的 [startInteractiveTransition:](https://developer.apple.com/documentation/uikit/uiviewcontrollerinteractivetransitioning/1622028-startinteractivetransition) 方法.
4. UIKit 等待一个动画执行对象调用上下文转场对象的 [completeTransition:](https://developer.apple.com/documentation/uikit/uiviewcontrollercontexttransitioning/1622042-completetransition) 方法.  
    你的自定义动画执行对象在多花结束后会调用这个方法, 一般来说会在动画结束的 completion block 中调用. 调用这个方法会结束动画并且让 UIKit 知道它可以调用 [presentViewController:animated:completion:](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621380-presentviewcontroller) 方法的 completion handler, 然后调用动画执行对象自己的 [animationEnded:](https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning/1622059-animationended) 方法.

> **重要!!!**  
必须在动画的末尾调用 [completeTransition:](https://developer.apple.com/documentation/uikit/uiviewcontrollercontexttransitioning/1622042-completetransition) 方法. 由于 UIKit 不会结束转场过程, 因此它把控制权交给了 app, 直到你调用这个方法, 才会结束转场.

## The Transitioning Context Object
在转场动画开始前, UIKit 会创建一个转场上下文对象( transitioning context object ), 这个对象包含了一些如何演示动画的的信息. 转场上下文对象是一个非常重要的部分. 它实现了 [UIViewControllerContextTransitioning](https://developer.apple.com/documentation/uikit/uiviewcontrollercontexttransitioning) 协议并保存了指向 view controller 和转场过程中使用的 views 的引用. 不仅如此, 它还保存了应如何进行转场的信息, 包括动画是否支持交互. 你的动画执行对象需要所有这些信息来配置和执行实际的动画.

> **重要!!!**  
在配置自定义动画的时候, 必须使用转场上下文对象中的对象和数据, 而不是你自己管理的缓存信息. 转场会在很多的情况下发生, 其中一些可能会改变动画的参数. 转场上下文对象中一定会有你展示动画时所需要的正确信息, 然而你自己的缓存信息可能会随着 animator 的方法被调用而过时.

下图展示了转场上下文对象是如何与其他对象进行交互的. 你的动画执行对象从 [animateTransition:](https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning/1622061-animatetransition) 方法中获取转场上下文对象. 你创建的动画应该运行在已经提供好的 container view 中. 例如当 present 一个 view controller, 把这个 view controller 的 view 添加为 container view 的子视图的时候. container view 应该作为 window 或者一个普通的 view, 但是在运行动画的时候它总是需要配置.

![The transitioning context object](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_transitioning-context-object_10-2_2x.png)

想要知道更多关于转场上下文对象的信息, 详见: [UIViewControllerContextTransitioning Protocol Reference](https://developer.apple.com/documentation/uikit/uiviewcontrollercontexttransitioning).

## The Transition Coordinator ( 转场调度者 )
对于原生的转场和你自定义的转场, UIKit 会创建一个转场调度者, 它能更加方便地执行额外动画. view controller 除了 presentation 和 dismissal 之外, 转场会在很多情况下发生, 比如旋转界面或者当 view controller 的 frame 变化的时候. 所有的这些转场意味着 view 层级的变化. 转场调度者能在追踪那些变化的同时, 还能让你的动画执行. 如需访问转场调度者, 可以从相关 view controller 的 [transitionCoordinator](https://developer.apple.com/documentation/uikit/uiviewcontroller/1619294-transitioncoordinator) 属性获取. 转场调度者仅仅存在于转场的过程中.

下图展示了在 presentation 过程中, 转场调度者与 view controllers 之间的关系.  在转场动画发生的时候, 使用转场调度者获取转场信息的同时注册你想要执行的 animation block. 转场调度者遵守 [UIViewControllerTransitionCoordinatorContext](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioncoordinatorcontext) 协议, 这个协议提供了关于时间点的信息, 动画当前状态的信息, 还有转场中包含的 views 和 view
controllers . 当你的 animation blocks 执行时, 他们基本上接收到一个有着相同信息的上下文对象.

![The transition coordinator objects](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_transition-coordinator-objects_10-3_2x.png)

更多关于转场调度者的信息, 见: [UIViewControllerTransitionCoordinator Protocol Reference](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioncoordinator). 想了解更多关于能用于配置动画的上下文信息, 见: [UIViewControllerTransitionCoordinatorContext Protocol Reference](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioncoordinatorcontext).

## *Presenting a View Controller Using Custom Animations*
如果要使用自定义的动画去 present 一个 view controller, 需要在你现有的 view controllers 中的 action method 中, 做以下几件事:
1. 创建你想要 present 的 view controller.
2. 创建你的自定义转场代理对象, 然后赋值给 view controller 的 transitioningDelegate 属性. 在转场代理的方法中应该在需要的时候创建并返回你自定义的动画执行对象.
3. 调用 presentViewController:animated:completion: 方法去 present view controler.

当你调用 presentViewController:animated:completion: 方法的时候, UIKit 会初始化 presentation 流程. Presentations 会在下一个 runloop 循环开始的时候开始直到你的自定义执行对象调用 completeTransition: 方法. 能够交互的转场允许你在转场正在进行的时候执行触摸事件, 但是不允许交互的转场会在动画执行对象规定的时间内完成.

## *Implementing the Transitioning Delegate*
设置转场代理的目的是创建然后返回你的自定义对象. 下列代码展示了转场的方法可以能够简单到什么程度. 这个例子创建了自定义动画执行对象且返回. 实际上, 大多数的工作是在动画执行对象中完成的.

```
// Creating an animator object
- (id<UIViewControllerAnimatedTransitioning>)
    animationControllerForPresentedController:(UIViewController *)presented
                         presentingController:(UIViewController *)presenting
                             sourceController:(UIViewController *)source {
    MyAnimator* animator = [[MyAnimator alloc] init];
    return animator;
}
```
转场代理中其他的方法其实也可以像上面的代码一样简单. 你可以根据 app 当前状态加入自己的逻辑来返回不同的动画执行对象. 更多关于转场代理方法的信息, 见: [UIViewControllerTransitioningDelegate Protocol Reference](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioningdelegate).

## *Implementing Your Animator Objects*
动画执行对象可以是任何遵守 UIViewControllerAnimatedTransitioning 协议的对象. 一个动画执行对象可以创建一段动画. 动画执行对象的关键在于它的 animateTransition: 方法, 你可以使用这个方法来创建实际的动画. 动画的过程大致可以分成以下几段:
1. 获取动画参数.
2. 使用 Core Animation 或者 UIView 的方法来创建动画.
3. 完成转场并做相应清除.

## Getting the Animation Parameters
通过 animateTransition: 方法传递的上下文转场对象包含了你执行动画时所需的数据. 在你能够从上下文转场对象获取最新信息的时候, 千万不要使用你自己的缓存信息或者从 view controllers 得来的信息. Presenting 和 dismissing view controllers 有时候会包含一些在 view controllers 之外的对象. 举例, 一个自定义的 presentation controller 可能会加一个 background view 作为 presentation 的一部分. 上下文转场对象会把额外的 views 和对象引入, 然后为你的动画提供正确的 views.
- 调用 [viewControllerForKey:](https://developer.apple.com/documentation/uikit/uiviewcontrollercontexttransitioning/1622043-viewcontroller) 方法两次来获取转场中 "from" 和 "to" view controller. 不要去假设你知道有哪些 view controllers 参与了转场. UIKit 可能会在适配新的 trait environment 或者在相应你 app 的请求的时候改变 view controllers.
- 调用 containerView 方法来为动画获取 superview. 你需要把所有关键的 subviews 添加到这个 superview 中. 举例, 在 presentation 的过程中, 把 presented view controller 的 view 加入到这个 view 中.
- 调用 viewForKey: 方法来获取需要添加或者移除的 view. view controller 的 view 可能不是转场过程中唯一需要添加或者移除的 view. Presentation controller 可能需要将 views 加入到层级中, 这些 views 可能也需要被添加或者移除. viewForKey: 方法返回 root view, 这个 root view 包含了你需要添加或者移除的所有 views.
- 调用 finalFrameForViewController: 方法来获取正在添加或者移除的 view 的最终的 frame.

上下文转场对象使用 "from" 和 "to" 命名系统来标示 view controllers, views, 还有转场中包含的 frame. 'from' view controller 指的是在转场开始时刻处在屏幕上的那个控制器, 那么 'to' view controller 则是在转场结束时刻处在屏幕上的 view controller. 就像下图中显示的那样, 'from' 和 'to' view controllers 在 presentation 和 dismissal 中交换了位置.

![The from and to objects](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_from-and-to-objects_10-4_2x.png)

值的交换可以更方便地让单独的一个 animator 处理 presentation 和 dismissal. 你在设计 animator 时, 你所需要做的全部只是添加一个属性记录正在执行的动画是 presentation 还是 dismissal. 他们俩之间唯一的区别在下面列出:
- 对于 presentation, 向 container view 层级添加 'to' view.
- 对于 dismissal, 从 container view 层级移除 'from' view.

## Creating the Transition Animations
在一个典型的 presentation 中, 属于 presented view controller 的 view 会在动画后处于适当的位置. 其他的 views 的动画可能只作为 presentation 的一部分, 但是动画的主要目标一定是被添加到 view 层级的那个 view.

当 main view 处在动画中时, 配置你的动画的基本动作是一样的. 你从转场上下文对象获取所需的对象和数据, 然后使用这些信息来创建实际的动画.
- Presentation animations:
    - 使用 viewControllerForKey: 和 viewForKey: 方法来获取 view controllers 和转场中包含的 views.
    - 设置 'to' view 的起始点. 同时也要初始化其他的属性的值.
    - 从上下文转场对象的 finalFrameForViewController: 方法中获取 'to' view 的结束位置.
    - 把 'to' view 添加为 container view 的 subview.
    - 创建动画.
        - 在你的 animation block 中, 把 'to' view 动画至其在 container view 中的结束位置. 并且把其他相关的属性也设置为结束时的值.
        - 在 completion block, 调用 completeTransition: 方法, 然后执行清理.

- Dismissal animations：
    - 使用 viewControllerForKey: 和 viewForKey: 方法来获取 view controllers 和转场中包含的 views.
    - 计算 'from' view 的结束位置. 这个 view 属于正在被 dismiss 的 presented view controller.
    - 把 'to' view 添加为 container view 的 subview.  
      在 presentation 过程中, 属于 presenting view controller 的 view 在转场结束时被移除. 结果, 你在 dismissal 过程中必须把那个 view 加回到 container.
    - 创建动画.
        - 在你的 animation block 中, 把 'from' view 动画至其在 container view 中的结束位置. 并且把其他相关的属性也设置为结束时的值.
        - 在你的 completion block 中, 从 view 的层级中移除 'from' view 并调用 completeTransition: 方法. 然后进行清理.

下图展示了一个自定义的 presentation 和 dismissal 转场, 这两个转场动画使 view 的动画在斜线方向运动. 在 presentation 中, presented view 在屏幕外开始然后向左上方的斜线运动直到它全部可见才停止. 在 dismissal 中, 运动轨迹则相反.

![A custom presentation and dismissal](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_custom-presentation-and-dismissal_10-5_2x.png)

下面代码展示了你应该如何实现上图的转场. 在获得了转场所需要的对象之后, animateTransition: 方法会计算相关 view 的 frame 区域. 在 presentation 过程中, toView 这个变量代表 presented view . 在 dismissal 过程中,  fromView 变量代表 dismissed view. presenting 属性是一个自定义的属性, 它属于动画执行对象. 转场代理会在创建动画执行对象的时候将合适的值赋值给 presenting 属性.

```
// Animations for implementing a diagonal presentation and dismissal

- (void)animateTransition:(id<UIViewControllerContextTransitioning>)transitionContext {
    // Get the set of relevant objects.
    UIView *containerView = [transitionContext containerView];
    UIViewController *fromVC = [transitionContext
            viewControllerForKey:UITransitionContextFromViewControllerKey];
    UIViewController *toVC   = [transitionContext
            viewControllerForKey:UITransitionContextToViewControllerKey];
 
    UIView *toView = [transitionContext viewForKey:UITransitionContextToViewKey];
    UIView *fromView = [transitionContext viewForKey:UITransitionContextFromViewKey];
 
    // Set up some variables for the animation.
    CGRect containerFrame = containerView.frame;
    CGRect toViewStartFrame = [transitionContext initialFrameForViewController:toVC];
    CGRect toViewFinalFrame = [transitionContext finalFrameForViewController:toVC];
    CGRect fromViewFinalFrame = [transitionContext finalFrameForViewController:fromVC];
 
    // Set up the animation parameters.
    if (self.presenting) {
        // Modify the frame of the presented view so that it starts
        // offscreen at the lower-right corner of the container.
        toViewStartFrame.origin.x = containerFrame.size.width;
        toViewStartFrame.origin.y = containerFrame.size.height;
    }
    else {
        // Modify the frame of the dismissed view so it ends in
        // the lower-right corner of the container view.
        fromViewFinalFrame = CGRectMake(containerFrame.size.width,
                                      containerFrame.size.height,
                                      toView.frame.size.width,
                                      toView.frame.size.height);
    }
 
    // Always add the "to" view to the container.
    // And it doesn't hurt to set its start frame.
    [containerView addSubview:toView];
    toView.frame = toViewStartFrame;
 
    // Animate using the animator's own duration value.
    [UIView animateWithDuration:[self transitionDuration:transitionContext]
                     animations:^{
                         if (self.presenting) {
                             // Move the presented view into position.
                             [toView setFrame:toViewFinalFrame];
                         }
                         else {
                             // Move the dismissed view offscreen.
                             [fromView setFrame:fromViewFinalFrame];
                         }
                     }
                     completion:^(BOOL finished){
                         BOOL success = ![transitionContext transitionWasCancelled];
 
                         // After a failed presentation or successful dismissal, remove the view.
                         if ((self.presenting && !success) || (!self.presenting && success)) {
                             [toView removeFromSuperview];
                         }
 
                         // Notify UIKit that the transition has finished
                         [transitionContext completeTransition:success];
                     }];
 
}

```

## Cleaning Up After the Animations
在转场动画的最后, 调用 completeTransition: 方法是一个非常关键的步骤. 调用这个方法会告诉 UIKit 转场已经结束, 用户可以开始使用 presented view controller 了. 调用这个方法也会触发一系列其他的 completion handlers, 包括 presentViewController:animated:completion: 方法和动画执行对象拥有的 animationEnded: 方法. 最佳调用 completeTransition: 方法的地方是在你的 animation block 的 completion handler 中.

由于转场可以被取消, 所以你应该使用上下文对象的  [transitionWasCancelled](https://developer.apple.com/documentation/uikit/uiviewcontrollercontexttransitioning/1622039-transitionwascancelled) 方法的返回值去决定需要清理哪些东西. 当一个 presentation 被取消了, 你的 animator 必须回复任何对 view 层级做出的修改. 一个成功的 dismissal 也需要类似的操作.

## *Adding Interactivity to Your Transitions*
是你的动画能够交互的最简单的方法时使用 [UIPercentDrivenInteractiveTransition](https://developer.apple.com/documentation/uikit/uipercentdriveninteractivetransition) 对象. UIPercentDrivenInteractiveTransition 对象与你现有的动画执行对象协作, 通过你提供的完成度 (completion percentage value), 来完成控制动画时机的任务. 你所需要做的是建立好所需的处理事件的代码来计算完成度, 并且在每一个新的事件到来的时候更新它.

你可以使用 UIPercentDrivenInteractiveTransition 类, 本类和子类均可. 如果你使用子类, 那么你需要使用子类的 init 方法 (或者 startInteractiveTransition: 方法) 来执行一次性的建立事件处理的代码. 在那之后, 使用你自定义的时间处理代码来计算一个新的完成度, 然后调用 updateInteractiveTransition: 方法. 当你的代码决定这个转场应该结束时, 调用 finishInteractiveTransition 方法.

下面代码展示了一个自定义实现的 startInteractiveTransition: 方法, 使用的是 UIPercentDrivenInteractiveTransition 的子类. 这个方法设置了一个 pan 手势去跟踪触摸事件, 然后将该手势识别器放置在了 container view 中为动画做准备. 这个方法也为接下来需要使用的转场上下文对象保存了一个引用.

```
// Configuring a percent-driven interactive animator

- (void)startInteractiveTransition:(id<UIViewControllerContextTransitioning>)transitionContext {
   // Always call super first.
   [super startInteractiveTransition:transitionContext];
 
   // Save the transition context for future reference.
   self.contextData = transitionContext;
 
   // Create a pan gesture recognizer to monitor events.
   self.panGesture = [[UIPanGestureRecognizer alloc]
                        initWithTarget:self action:@selector(handleSwipeUpdate:)];
   self.panGesture.maximumNumberOfTouches = 1;
 
   // Add the gesture recognizer to the container view.
   UIView* container = [transitionContext containerView];
   [container addGestureRecognizer:self.panGesture];
}
```

手势识别器 (gesture recognizer) 对每一个新到来的事件调用了它的 action 中的方法. action 方法的实现可以使用手势识别器的状态信息来判断手势是否成功, 失败, 或者是还在进行当中. 同时, 你可以使用最近的触摸事件信息来为一个手势计算新的百分比值.

下面的代码展示了在上面代码中配置的 pan 手势识别器调用的方法. 当新事件来临时, 这个方法使用了垂直移动距离( vertical travel distance )来计算动画的完成度. 当手势结束后, 这个方法会终止转场. 

```
// Using events to update the animation progress

-(void)handleSwipeUpdate:(UIGestureRecognizer *)gestureRecognizer {
    UIView* container = [self.contextData containerView];
 
    if (gestureRecognizer.state == UIGestureRecognizerStateBegan) {
        // Reset the translation value at the beginning of the gesture.
        [self.panGesture setTranslation:CGPointMake(0, 0) inView:container];
    }
    else if (gestureRecognizer.state == UIGestureRecognizerStateChanged) {
        // Get the current translation value.
        CGPoint translation = [self.panGesture translationInView:container];
 
        // Compute how far the gesture has travelled vertically,
        //  relative to the height of the container view.
        CGFloat percentage = fabs(translation.y / CGRectGetHeight(container.bounds));
 
        // Use the translation value to update the interactive animator.
        [self updateInteractiveTransition:percentage];
    }
    else if (gestureRecognizer.state >= UIGestureRecognizerStateEnded) {
        // Finish the transition and remove the gesture recognizer.
        [self finishInteractiveTransition];
        [[self.contextData containerView] removeGestureRecognizer:self.panGesture];
    }
}
```

> 注意!!!  
你所计算的值代表了动画全程的完成度. 对于交互动画来说, 你可能想要避免非线性的效果, 比如初始速度( initial velocities ), 阻尼值( damping values ) , 还有动画本身的非线性完成曲线( nonlinear completion curves ). 这种效果往往使事件的触摸位置与任何底层视图的移动分离. (Such effects tend to decouple the touch location of events from the movement of any underlying views.)

## *Creating Animations that Run Alongside a Transition*
转场所包含的 view controllers 可以在 presentation 或者转场动画之上执行额外动画. 举例, 一个 presented view controller 可能会在转场过程中在它自己的 view 层级上做动画, 也可能会在转场发生时添加运动效果或其他的世界反馈. 任何对象都可以创建动画, 只要它可以访问 presented 或 presenting view controller 的 transitionCoordinator 属性. 转场调度者只会在转场进行的过程中存在.

如果要创建动画, 需要调用转场调度者的 [animateAlongsideTransition:completion:](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioncoordinator/1619300-animate) 或 [animateAlongsideTransitionInView:animation:completion:](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioncoordinator/1619295-animatealongsidetransition) 方法. 你提供的 blocks 会被储存起来直到动画开始，他们会在剩余的动画过程中一直被执行下去。

## *Using a Presentation Controller with Your Animations*
对于自定义 presentation 来说，你可以提供你自己的 presentation controller 来为 presented view controller 提供自定义的展示。 Presentation controllers 管理任何与 view controller 和它的内容分离开来的自定义物件（chrome）。举例， 一个放置在 view controller 背后的 dimming view 会由 presentation controller 来管理。 presentation controller 不负责管理特定的 view controlelr 的 view 的这个事实说明了你可以在你的 app 中使用相同的 presentation controller 和 任意的 view controller。

你为 presented view controller 的转场代理提供一个自定义的 presentation controller (view controller 的 modalTransitionStyle 属性必须得是 UIModalPresentationCustom)。Presentation controller 会与 animator 对象同时运作。当动画执行对象把 view controller 的 view 移动到了对应的地方， Presentation controller 会把额外的 views 移动到对应的位置。在转场结束的时候，presentation controller 有一个机会对 view 层级做最后的调整。想要知道更多关于如何创建自定义 presentation controller，见[ Creating Custom Presentations](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/DefiningCustomPresentations.html#//apple_ref/doc/uid/TP40007457-CH25-SW1)

---

# Creating Custom Presentations

UIKit 通过将 present 内容和 display 内容这种方式来将 view controllers 的内容分开. Presented view controllers 被位于其之下的 presentation controller 对象管理着, presentation controller 管理着展示 view controller 的 view 的视觉风格. presentation controller 可能会做下面列举的事情:
- 设置 presented view controller 的尺寸.
- 在 presented content 中添加自定义 views 来改变视觉效果.
- 为其自定义的任何视图提供转场动画.
- 当 app 的环境发生变化的时候, 调整 presentation 的视觉效果.

UIKit 为标准的 presentation 方式提供 presentation controller. 当你把一个 view controller 的 presentation style 设置为 UIModalPresentationCustom 然后提供一个合适的转场代理, UIKit 则会使用你自定义的 presentation controller.

## *The Custom Presentation Process*

当你需要 present 一个 presentation style 为 UIModalPresentationCustom 的 view controller, UIKit 会去查找一个自定义的 presentation controller 用于管理 presentation 的过程. 当 presentation 正在进行时, UIKit 会调用 presentation controller 的方法, 给一个机会让 presentation controller 设置自定义 views 以及将他们动画至相应位置.

presentation controller 与动画执行对象协同工作来完成一个完整的转场过程. 动画执行对象只需要负责将 view controller 的 contents 添加动画效果, 其余的工作全部交给 presentation controller 就可以了. 通常来说, presentation controller 只会为它自己的 views 添加动画效果, 但是你可以重写 presentation controller 的 presentedView 方法来让动画执行对象为所有或者部分 views 添加动画效果.

在 presentation 过程中, UIKit:
1. 调用转场代理的 presentationControllerForPresentedViewController:presentingViewController:sourceViewController: 方法来获取你的自定义 presentation controller.
2. 要求转场代理提供动画执行对象和交互动画执行对象, 如果有的话.
3. 调用 presentation controller 的 presentationTransitionWillBegin 方法  
   在这个方法的实现中, 你应该在 view 层级中添加自定义 views 然后为那些 views 的动画效果进行配置.
4. 从你的 presentation controller 中获取 *presentedView*  
   这个方法返回的 view 会被动画执行对象动画之对应的位置. 一般来说, 这个方法会返回 presented view controller 的 root view. 你的 presentation controller 可以根据需要来使用一张自定义的 background view 去替换那个 view. 如果你确实指定了一个不同的 view, 那么你必须把 presented view controller 的 root view 添加到你的 view 层级中.
5. 执行转场动画  
   转场动画包括了由动画执行对象创建的动画和任何你配置好的动画, 这些动画与主动画一同被执行.  
   在动画过程中, UIKit 会调用 presentation controller 的 containerViewWillLayoutSubviews 和 containerViewDidLayoutSubviews 这两个方法, 这样, 你就可以根据需要去调整你自定义 views 的布局了.
6. 当转场动画结束时, 调用 presentationTransitionDidEnd: 方法.

在 dismissal 过程中, UIKit:
1. 从当前可见的 view controller 获取你自定义的 presentation controller.
2. 要求转场代理提供动画执行对象和交互动画执行对象, 如果有的话.
3. 调用 presentation controller 的 dismissalTransitionWillBegin 方法  
   在这个方法的实现中, 你应该在 view 层级中添加自定义 views 然后为那些 views 的动画效果进行配置.
4. 从 presentation controller 获取已经存在的 *presentedView*.
5. 执行转场动画  
   转场动画包括了由动画执行对象创建的动画和任何你配置好的动画, 这些动画与主动画一同被执行.  
   在动画过程中, UIKit 会调用 presentation controller 的 containerViewWillLayoutSubviews 和 containerViewDidLayoutSubviews 这两个方法, 这样, 你就可以移除任何自定义的约束.
6. 在转场动画结束时, 调用 dismissalTransitionDidEnd: 方法.

在 presentation 过程中, presentation controller 的 frameOfPresentedViewInContainerView 和 presentedView 方法可能会被多次调用, 所以在方法的实现中, 你应该快速地 return 返回值. 还有, 在 presentedView 的实现中, 不应该试图去设置 view 层级. view 层级在这个方法调用之前, 就应该已经被设置好了.

## *Creating a Custom Presentation Controller*
如需实现一个自定义的 presentation style, 子类化 UIPresentationController 然后为 presentation 创建 views 和 animations. 当你在创建一个自定义的 presentation controller 时, 你应该考虑一下问题:
- 需要添加什么 views?
- 你想如何在屏幕上为额外的 views 添加动画效果?
- presented view controller 的尺寸应该是多少?
- presentation 应该如何在适应水平 regular 和水平 compact 的情况?
- 当 presentation 结束时, presenting view controller 应该被移除吗?

以上所有的问题需要在 UIPresentationController 类中的不同方法中去解决.