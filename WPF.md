# WPF



# 一、布局容器

### 1. **Grid**（最常用，表格布局）

```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="100"/>
        <ColumnDefinition Width="*"/>
    </Grid.ColumnDefinitions>
</Grid>
```

### 2. **StackPanel**（垂直 / 水平排列）

```xml
<StackPanel Orientation="Vertical"/>
<StackPanel Orientation="Horizontal"/>
```

### 3. **WrapPanel**（自动换行）

### 4. **DockPanel**（停靠布局）

### 5. **Canvas**（绝对坐标定位）

### 6. **ScrollViewer**（滚动容器）

```xml
<ScrollViewer>
    <StackPanel/>
</ScrollViewer>
```

# 二、基础输入控件

### 1. **TextBox** 文本输入

```xml
<TextBox Text="{Binding Name}" PlaceholderText="请输入"/>
```

### 2. **PasswordBox** 密码框

```xml
<PasswordBox PasswordChar="*"/>
```

### 3. **Button** 按钮

```xml
<Button Content="确定" Command="{Binding SaveCommand}"/>
```

### 4. **ToggleButton** 开关按钮

### 5. **RepeatButton** 长按重复触发

# 三、选择类控件

### 1. **CheckBox** 复选框

```xml
<CheckBox Content="已同意" IsChecked="{Binding IsAgree}"/>
```

### 2. **RadioButton** 单选框

```xml
<RadioButton Content="男" GroupName="Gender"/>
<RadioButton Content="女" GroupName="Gender"/>
```

### 3. **ComboBox** 下拉框

```xml
<ComboBox ItemsSource="{Binding CityList}"
          SelectedItem="{Binding SelectedCity}"/>
```

### 4. **ListBox** 列表选择

### 5. **ListView** 带视图的列表

### 6. **DataGrid** 表格（超级常用）

```xml
<DataGrid ItemsSource="{Binding UserList}" AutoGenerateColumns="False">
    <DataGrid.Columns>
        <DataGridTextColumn Header="姓名" Binding="{Binding Name}"/>
        <DataGridTextColumn Header="年龄" Binding="{Binding Age}"/>
    </DataGrid.Columns>
</DataGrid>
```

# 四、文本显示

### 1. **TextBlock** 普通文本（推荐）

```xml
<TextBlock Text="Hello WPF" FontSize="16" Foreground="Red"/>
```

### 2. **Label** 带标签功能

### 3. **ToolTip** 提示

```xml
<Button ToolTip="点击保存"/>
```

# 五、图片与媒体

### 1. **Image** 图片

```xml
<Image Source="/Images/logo.png" Stretch="Uniform"/>
```

### 2. **MediaElement** 视频 / 音频

```xml
<MediaElement Source="test.mp4"/>
```

# 六、进度与状态

### 1. **ProgressBar** 进度条

```xml
<ProgressBar Value="50" Maximum="100"/>
```

### 2. **Slider** 滑块

```xml
<Slider Minimum="0" Maximum="100" Value="{Binding Volume}"/>
```

### 3. **ProgressRing** 加载圈（需 WinUI 或库）

# 七、容器与分组

### 1. **GroupBox** 分组框

```xml
<GroupBox Header="基本信息">
    <StackPanel/>
</GroupBox>
```

### 2. **Expander** 可展开面板

```xml
<Expander Header="高级选项">
    <StackPanel/>
</Expander>
```

### 3. **TabControl** 选项卡

```xml
<TabControl>
    <TabItem Header="用户"/>
    <TabItem Header="订单"/>
</TabControl>
```

# 八、菜单与工具栏

### 1. **Menu** 菜单

```xml
<Menu>
    <MenuItem Header="文件">
        <MenuItem Header="打开"/>
    </MenuItem>
</Menu>
```

### 2. **ToolBar** 工具栏

### 3. **ContextMenu** 右键菜单

```xml
<Button ContextMenu="{StaticResource MyMenu}"/>
```

# 九、弹窗与窗口

### 1. **Window** 主窗口

### 2. **MessageBox** 消息框

```csharp
MessageBox.Show("保存成功");
```

### 3. **OpenFileDialog / SaveFileDialog** 文件对话框

```csharp
var dlg = new OpenFileDialog();
dlg.ShowDialog();
```

### 4. **Popup** 悬浮弹窗

# 十、特殊高级控件

### 1. **DatePicker** 日期选择

```xml
<DatePicker SelectedDate="{Binding CreateTime}"/>
```

### 2. **Calendar** 日历

### 3. **TreeView** 树形控件

```xml
<TreeView ItemsSource="{Binding MenuTree}"/>
```

### 4. **StatusBar** 状态栏

### 5. **ItemsControl** 万能列表控件

# 十一、用于 MVVM 绑定的关键属性

```xml
Text="{Binding Name}"
IsChecked="{Binding IsEnabled}"
SelectedItem="{Binding SelectedItem}"
ItemsSource="{Binding List}"
Command="{Binding SaveCommand}"
```