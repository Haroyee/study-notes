启用 **Hyper-V** 功能

```cmd
dism.exe /Online /Enable-Feature:Microsoft-Hyper-V /All
```

设置 Hypervisor 的启动类型为 自动，系统会在启动时自动加载 Hypervisor，从而使 Hyper-V 功能可用

```cmd
bcdedit /set hypervisorlaunchtype auto
```

关闭Hypervisor

```cmd
bcdedit /set hypervisorlaunchtype off
```

