# 从v3版本迁移到v4版本

<p class="description">是的，我们已经发布了v4版本！</p>

您在找v3版本的文档吗？ [您可以在这里找到它们](https://material-ui.com/versions/) 。

> 此文档尚未完成。 您是否已经升级了站点并且遇到了一些并没有在此涉及的问题？ [Add your changes on GitHub](https://github.com/mui-org/material-ui/blob/master/docs/src/pages/guides/migration-v3/migration-v3.md).

## 简介

当您将站点从 Material-UI 的v3版本升级到v4版本时，此篇会给您提供一些参考。 您可能不会将这里所有覆盖的内容运用到你的站点上。 我们会尽我们最大的努力让文档简单易懂，并尽可能有序地介绍，这样您可以迅速对v4版本游刃有余。

## 为什么您需要迁移

此文档介绍了*h如何*从v3版本迁移到v4版本。 关于迁移的*原因*，我们则在 [Medium上发布了一篇博客](https://medium.com/material-ui/material-ui-v4-is-out-4b7587d1e701)来详细解说。

## 升级你的依赖包

The very first thing you will need to do is to update your dependencies.

### 升级 Material-UI 的版本

若想要使用最新版本的 Material-UI，您必须更新 `package.json`。

```json
"dependencies": {
  "@material-ui/core": "^4.0.0"
}
```

或者运行

```sh
npm install @material-ui/core@next

或者

yarn add @material-ui/core@next
```

### 更新 React 的版本

对于 React 版本的最低要求是从 `react@^16.3.0` 升级到 `react@^16.8.0`。 这样一来我们能够依赖 [Hooks](https://reactjs.org/docs/hooks-intro.html) 的功能（我们已经不再使用 class API）。

### 更新 Material-UI Styles 的版本

If you were previously using `@material-ui/styles` with v3 you need to update your `package.json` to use the latest version of Material-UI Styles.

```json
"dependencies": {
  "@material-ui/styles": "^4.0.0"
}
```

或者运行

```sh
npm install @material-ui/styles@next

或者

yarn add @material-ui/styles@next
```

## 处理变化带来的系统崩溃

### Core

- 每个组件会提供他们的 ref。 这是通过使用 `React.forwardRef()` 实现的。 这回影响到内部的组件树和显示的名称，进而会使得 shallow 或者 snapshot 测试崩溃。 `innerRef` 不再返回一个实例的 ref（或者当内部组件是一个函数组件时，什么都不返回），而是返回一个它根组件的 ref。 相应的 API 文档在根组件中列出了。

### Styles（样式表单）

- ⚠️ Material-UI 依赖于 JSS v10版本。 JSS v10版本与v9版本不向后兼容。 请保证您的开发环境中未安装 JSS v9版本。 Removing `react-jss` from your `package.json` can help. StylesProvider 组件替代了 JssProvider 组件。
- 请移除 `withTheme()` 中的第一个可选的参数。 第一个参数本是作为未来的可能的选项的一个占位符。 我们从未发现有需要它的情况。 该是删除这个参数的时候了。 It matches the [emotion API](https://emotion.sh/docs/introduction) and the [styled-components API](https://www.styled-components.com).
  
  ```diff
  -const DeepChild = withTheme()(DeepChildRaw);
  +const DeepChild = withTheme(DeepChildRaw);
  ```

- 仔细研究 [keyframes 的 API](https://cssinjs.org/jss-syntax/#keyframes-animation)。 您应该在您的代码中做出以下改变。 它能帮助分离动画效果的逻辑：
  
  ```diff
    rippleVisible: {
      opacity: 0.3,
  -   animation: 'mui-ripple-enter 100ms cubic-bezier(0.4, 0, 0.2, 1)',
  +   animation: '$mui-ripple-enter 100ms cubic-bezier(0.4, 0, 0.2, 1)',
    },
    '@keyframes mui-ripple-enter': {
      '0%': {
        opacity: 0.1,
      },
      '100%': {
        opacity: 0.3,
      },
    },
  ```

### Theme（主题）

- `theme.palette.augmentColor()` 方法不再对输入框的颜色产生副作用。 若想要正确地使用它，您必须使用返回的值。
  
  ```diff
  -const background = { main: color };
  -theme.palette.augmentColor(background);
  +const background = theme.palette.augmentColor({ main: color });
  
  console.log({ background });
  ```

- 您可以从创建主体中安全地删除下一个变体：
  
  ```diff
  typography: {
  - useNextVariants: true,
  },
  ```

- 我们已经不再使用 `theme.spacing.unit`，您可以用新的 API 了：
  
  ```diff
  label: {
    [theme.breakpoints.up('sm')]: {
  -   paddingTop: theme.spacing.unit * 12,
  +   paddingTop: theme.spacing(12),
    },
  }
  ```
  
  *Tip: you can provide more than 1 argument: `theme.spacing(1, 2) // = '8px 16px'`*.
  
  您可以在项目中使用[迁移小帮手](https://github.com/mui-org/material-ui/tree/master/packages/material-ui-codemod/README.md#theme-spacing-api)来让您的迁移流程更加顺畅。

### Layout（布局）

- [Grid] 为了支持任意的间距值，并且移除每次心算都需要数8，我们改变了间距的 API：
  
  ```diff
    /**
     * 定义了类型为 `item` 组件之间的距离。
     * 它只能用于类型为 `container` 的组件。
     */
  -  spacing: PropTypes.oneOf([0, 8, 16, 24, 32, 40]),
  +  spacing: PropTypes.oneOf([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]),
  ```
  
  接下来，您可以用主题来实现[一个自定义网格间距的转换功能](https://material-ui.com/system/spacing/#transformation)。

- [Container] Moved from `@material-ui/lab` to `@material-ui/core`.
  
  ```diff
  -import Container from '@material-ui/lab/Container';
  +import Container from '@material-ui/core/Container';
  ```

### Buttons（按钮）

- [Button] 删除不推荐使用的按钮变体（flat，raised 和 fab）：
  
  ```diff
  -<Button variant="raised" />
  +<Button variant="contained" />
  ```
  
  ```diff
  -<Button variant="flat" />
  +<Button variant="text" />
  ```
  
  ```diff
  -import Button from '@material-ui/core/Button';
  -<Button variant="fab" />
  +import Fab from '@material-ui/core/Fab';
  +<Fab />
  ```
  
  ```diff
  -import Button from '@material-ui/core/Button';
  -<Button variant="extendedFab" />
  +import Fab from '@material-ui/core/Fab';
  +<Fab variant="extended" />
  ```

- [ButtonBase] 传递给`组件`的属性的组件需要能接受一个 ref。 [组合指南](/guides/composition/#caveat-with-refs)解释了迁移的策略。
  
  This also applies to `BottomNavigationAction`, `Button`, `CardActionArea`, `Checkbox`, `ExpansionPanelSummary`, `Fab`, `IconButton`, `MenuItem`, `Radio`, `StepButton`, `Tab`, `TableSortLabel` as well as `ListItem` if the `button` prop is true.

### Cards（卡片）

- [CardActions] Rename the `disableActionSpacing` prop to `disableSpacing`.
- [CardActions] 移除 CSS类中的 `disableActionSpacing`。
- [CardActions] Rename the `action` CSS class to `spacing`.

### ClickAwayListener（他处点击监听器）

- [ClickAwayListener] 隐藏 react-event-listener 的属性。

### 对话框 (Dialog)

- [DialogActions] Rename the `disableActionSpacing` prop to `disableSpacing`.
- [DialogActions] Rename the `action` CSS class to `spacing`.
- [DialogContentText] 不使用文字铸排变体 `subtitle1`，而使用 `body1`。
- [Dialog] 子组件能够接受一个 ref。 The [composition guide](/guides/composition/#caveat-with-refs) explains the migration strategy.

### Dividers（分隔线）

- [Divider] Remove the deprecated `inset` prop.
  
  ```diff
  -<Divider inset />
  +<Divider variant="inset" />
  ```

### ExpansionPanel（扩展面板）

- [ExpansionPanelActions] Rename the `action` CSS class to `spacing`.
- [ExpansionPanel] 加强 `disabled` 样式规则的 CSS 特性。

### Lists（列表）

- [List] 为了符合规范，我们重新在列表组件上做了调整 ：
  
  - The `ListItemAvatar` component is required when using an avatar`.
  - The `ListItemIcon` component is required when using a left checkbox.
  - 您必须要在图标按钮上设置 `edge` 属性。

- [ListItem] 加强 `disabled` 和 `focusVisible` 样式规则的 CSS 特性。

### Menu（菜单）

- [MenuItem] 删除 MenuItem 的固定高度。 浏览器将会自行根据间距和行高来计算高度。

### Modal（模态框）

- [Modal] 子组件能够接受一个 ref。 [组合指南](/guides/composition/#caveat-with-refs)解释了迁移的策略。
  
  这也适用于 `Dialog` 和 `Popover` 。

- [Modal] Remove the classes customization API for the Modal component(-74% bundle size reduction when used standalone).

- [Modal] 现在忽略了 event.defaultPrevented。 即使当向下离开事件调用了 `event.preventDefault()`，新的逻辑也会关闭模态框。 `event.preventDefault()` 旨在禁用一些默认的行为，如单击一个复选框来选中它；点击按钮来提交表单；以及点击左键来移除文本输入框的光标等等。 只有一些特殊的HTML元素才具有这些默认的行为。 若您不想触发模态框的 `onClose` 事件，您需要使用 `event.stopPropagation()`。

### Paper（纸张）

- [Paper] 减小默认的 elevation（阴影高度）。 为了适配卡片组件和扩展面板组件，请更改默认纸张的阴影高度：
  
  ```diff
  -<Paper />
  +<Paper elevation={2} />
  ```
  
  这也会影响 `扩展面板`。

### Portal（传送门）

- [Portal] 当使用 `disablePortal`属性的时候，子元素需要能够接受一个 ref。 [组合指南](/guides/composition/#caveat-with-refs)解释了迁移的策略。

### Slide（滑块）

- [Slide] 子组件能够接受一个 ref。 [组合指南](/guides/composition/#caveat-with-refs)解释了迁移的策略。

### Switch （开关）

- [Switch] 重新编写实施的代码能够更容易覆盖样式表。 请重命名类的名字以匹配规范的用词：
  
  ```diff
  -icon
  -bar
  +thumb
  +track
  ```

### Snackbar（消息条）

- [Snackbar] 匹配新的规范。
  
  - 更改尺寸。
  - 将默认的过渡动画从`Slide`改成`Grow`。

### SvgIcon（Svg图标）

- [SvgIcon] 重命名nativeColor - > htmlColor。 React 在 `for` 这个 HTML 属性上也遇到了同样的问题，他们选择命名这个属性为`htmlFor`。 此变化的原因大同小异。
  
  ```diff
  -<AddIcon nativeColor="#fff" />
  +<AddIcon htmlColor="#fff" />
  ```

### Tabs（选项卡）

- [Tab] 为了简单起见，删除了` labelContainer `，`label` 和 `labelWrapped`等类的 key。 这使得我们可以移走中间的两个 DOM 元素。 您应该将自定义的样式已到`根元素`的类的 key 上。
  
  ![一个更简单的标签项的 DOM 结构](https://user-images.githubusercontent.com/3165635/53287870-53a35500-3782-11e9-9431-2d1a14a41be0.png)

- [Tabs] 移除了弃用的 <0>fullWidth</0> 和 <0>scrollable</0> 属性：:
  
  ```diff
  -<Tabs fullWidth scrollable />
  +<Tabs variant="scrollable" />
  ```

### Table（表格）

- [TableCell] Remove the deprecated `numeric` property:
  
  ```diff
  -<TableCell numeric>{row.calories}</TableCell>
  +<TableCell align="right">{row.calories}</TableCell>
  ```

- [TableRow] 删除了 CSS 属性中的固定高度。 浏览器将会自行根据间距和行高来计算单元格的高度。
- [TableCell] 将 `dense` 模式移至一个不同的属性：
  
  ```diff
  -<TableCell padding="dense" />
  +<TableCell size="small" />
  ```

- [TablePagination] 此组件不再修复无效的属性（`page`，`count`，`rowsPerPage`）组合。 相反的，它会给出一个警告。

### 文本字段

- [InputLabel] 凭借 InputLabel 组件的类 API，您应该可以覆盖 FormLabel 组件所有的样式表。 我们移除了 `FormLabelClasses` 属性。
  
  ```diff
  <InputLabel
  - FormLabelClasses={{ asterisk: 'bar' } }
  + classes={{ asterisk: 'bar' } }
  >
    Foo
  </InputLabel>
  ```

- [InputBase] 改变了默认的盒子模型的大小。 现如今它则使用以下的 CSS：
  
  ```css
  box-sizing: border-box;
  ```
  
  与 `fullWidth` 属性有关的问题迎刃而解。

- [InputBase] 从 `InputBase` 中移走了 `inputType` 类。

### Tooltip（提示）

- [Tooltip] 子组件能够接受一个 ref。 [组合指南](/guides/composition/#caveat-with-refs)解释了迁移的策略。
- [Tooltip] 相比以前任何聚焦都会出现，现在只会在 focus-visible 聚焦的时候出现。

### Typography（文字铸排）

- [Typography] 移除了各种弃用的铸排变体。 您可以通过执行以下的替换来升级： 
  - display4 => h1
  - display3 => h2
  - display2 => h3
  - display1 => h4
  - headline => h5
  - title => h6
  - subheading => subtitle1
  - body2 => body1
  - body1 (default) => body2 (default)
- [Typography] 移除了固定的 `display: block` 这个默认的铸排样式。 您现在可以使用新的 `display?: 'initial' | 'inline' | 'block';` 属性。
- [Typography] 为了达到更好的排版效果，请重命名属性 `headlineMapping` 为 `variantMapping`。
  
  ```diff
  -<Typography headlineMapping={headlineMapping}>
  +<Typography variantMapping={variantMapping}>
  ```

- [Typography] 将默认的字体从 `body2` 换成 `body1`。 默认为16px的字体大小比默认为14px好。 Bootstrap，material.io，甚至我们的文档都将16px作为默认字体大小。 像 Ant Design 一样使用14px是可以理解的，因为中国的用户使用了不同的字母表。 我们建议使用12px作为日语的默认字体大小。
- [Typography] 移除了铸排变体的默认颜色。 大多数情况下，字体颜色应该是继承而来的。 这是网站的默认行为。
- [Typography] 遵循 #13028的逻辑，将 `color="default"` 重命名为 `color="initial"`。 不应该再使用*default*，它缺少明确的语义。

### Node

- [是时候放弃对 node 6 的支持](https://github.com/nodejs/Release/blob/eb91c94681ea968a69bf4a4fe85c656ed44263b3/README.md#release-schedule)，而升级到 node 8 啦。

### UMD

- 此更改简化了 Material-UI 与 CDN 的使用：
  
  ```diff
  const {
    Button,
    TextField,
  -} = window['material-ui'];
  +} = MaterialUI;
  ```
  
  它与其他 React 的项目保持一致：
  
  - material-ui => MaterialUI
  - react-dom => ReactDOM
  - prop-types => PropTypes