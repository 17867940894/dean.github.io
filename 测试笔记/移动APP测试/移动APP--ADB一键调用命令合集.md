# 一、ADB一键调用命令合集（自动化脚本直接用）

## 基础导航

adb shell [[input]] keyevent 3  # 返回桌面Home

adb shell input keyevent 4  # 返回上一页

adb shell input keyevent 187 # 调出多任务后台

adb shell input keyevent 26 # 电源键（亮屏/锁屏）

## 音量控制

adb shell input keyevent 24 # 音量+

adb shell input keyevent 25 # 音量-

adb shell input keyevent 164 # 静音切换

## 输入编辑

adb shell input keyevent 62 # 空格

adb shell input keyevent 66 # 回车确认

adb shell input keyevent 67 # 退格删除

adb shell input keyevent 112 # 删除光标后文字

## 方向键操作

adb shell input keyevent 19 # 上

adb shell input keyevent 20 # 下

adb shell input keyevent 21 # 左

adb shell input keyevent 22 # 右

adb shell input keyevent 23 # 确认OK

## 多媒体控制

adb shell input keyevent 85 # 播放/暂停

adb shell input keyevent 86 # 停止播放

adb shell input keyevent 87 # 下一首

adb shell input keyevent 88 # 上一首

adb shell input keyevent 89 # 快退

adb shell input keyevent 90 # 快进

## 相机&通话

adb shell input keyevent 27 # 拍照

adb shell input keyevent 5  # 拨号

adb shell input keyevent 6  # 挂断

## 数字输入示例

adb shell input keyevent 7  # 0

adb shell input keyevent 8  # 1

adb shell input keyevent 9  # 2

adb shell input keyevent 10 # 3

adb shell input keyevent 11 # 4

adb shell input keyevent 12 # 5

adb shell input keyevent 13 # 6

adb shell input keyevent 14 # 7

adb shell input keyevent 15 # 8

adb shell input keyevent 16 # 9

## 自动化批处理脚本

###  基础操作脚本：ADB_QuickCtrl.bat

新建文本文档，复制全部内容后，后缀改为 `.bat`，手机开启USB调试连接电脑后双击运行

```Plain
@echo off
chcp 65001 >nul
title 安卓ADB按键快捷控制器
:menu
cls
echo ==============================================
echo          Android KeyCode 按键工具菜单
echo ==============================================
echo 【系统导航】
echo 1. 返回桌面Home      keyevent 3
echo 2. 返回上一页        keyevent 4
echo 3. 打开多任务后台    keyevent 187
echo 4. 电源键(亮屏/锁屏) keyevent 26
echo.
echo 【音量控制】
echo 5. 音量加            keyevent 24
echo 6. 音量减            keyevent 25
echo 7. 静音切换          keyevent 164
echo.
echo 【输入编辑】
echo 8. 空格              keyevent 62
echo 9. 回车确认          keyevent 66
echo 10. 退格删除文字     keyevent 67
echo.
echo 【多媒体播放】
echo 11. 播放/暂停切换    keyevent 85
echo 12. 下一曲           keyevent 87
echo 13. 上一曲           keyevent 88
echo.
echo 【相机通话】
echo 14. 拍照             keyevent 27
echo 15. 拨号键           keyevent 5
echo 16. 挂断电话         keyevent 6
echo.
echo 【方向键】
echo 17. 上  18. 下  19. 左  20. 右  21. 确认OK
echo.
echo 0. 退出程序
echo ==============================================
set /p opt=请输入功能序号执行：
if "%opt%"=="1" adb shell input keyevent 3
if "%opt%"=="2" adb shell input keyevent 4
if "%opt%"=="3" adb shell input keyevent 187
if "%opt%"=="4" adb shell input keyevent 26
if "%opt%"=="5" adb shell input keyevent 24
if "%opt%"=="6" adb shell input keyevent 25
if "%opt%"=="7" adb shell input keyevent 164
if "%opt%"=="8" adb shell input keyevent 62
if "%opt%"=="9" adb shell input keyevent 66
if "%opt%"=="10" adb shell input keyevent 67
if "%opt%"=="11" adb shell input keyevent 85
if "%opt%"=="12" adb shell input keyevent 87
if "%opt%"=="13" adb shell input keyevent 88
if "%opt%"=="14" adb shell input keyevent 27
if "%opt%"=="15" adb shell input keyevent 5
if "%opt%"=="16" adb shell input keyevent 6
if "%opt%"=="17" adb shell input keyevent 19
if "%opt%"=="18" adb shell input keyevent 20
if "%opt%"=="19" adb shell input keyevent 21
if "%opt%"=="20" adb shell input keyevent 22
if "%opt%"=="21" adb shell input keyevent 23
if "%opt%"=="0" exit
echo.
echo 操作执行完成，按任意键返回菜单
pause >nul
goto menu
```

### 批量连续操作脚本：Batch_Action.bat

适合自动化流程，示例：解锁→返回桌面→打开后台→返回

```Plain
@echo off
chcp 65001 >nul
echo 开始执行批量按键操作...
:: 亮屏
adb shell input keyevent 26
timeout /t 1 /nobreak >nul
:: 返回桌面
adb shell input keyevent 3
timeout /t 0.5 /nobreak >nul
:: 打开多任务后台
adb shell input keyevent 187
timeout /t 1 /nobreak >nul
:: 返回上一页
adb shell input keyevent 4
echo 批量操作全部完成！
pause
```

### 文本快速输入脚本：Input_Text.bat

不用逐个KeyCode输入字母，直接输入自定义文字

```Plain
@echo off
chcp 65001 >nul
set /p text=请输入要填充到手机的文字：
adb shell input text "%text%"
echo 文字输入完成
pause
```

### 使用前置条件

1. 电脑配置ADB环境，adb命令可在cmd直接调用；
2. 安卓手机开启：设置-开发者选项-USB调试；
3. USB数据线连接手机与电脑，手机弹出「允许USB调试」点确认；
4. 验证连接：cmd输入 `adb devices` 能看到设备编号即正常。



## 相关笔记
- [[移动APP测试]]
