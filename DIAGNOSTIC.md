# HeadsetPlay #10 播放诊断版 diag1

这是 `/var/minis/shared/headsetplay` 的副本；其播放控制器与此前比对的 #10 控制器相同。仅为排查首次连接而在控制器加 NSLog，保留原包 ID `com.axs.headsetplay`、配置、SpringBoard 注入及播放决策；故这是**替换版，不是可并装的旁路插件**。日志会带来开销并可能改变时序；不能保证与 #10 二进制完全等价，也不是修复版。不处理弹窗。

版本为 `5.2.0-10+diag1`，便于识别；请预先保留原版 deb。不要同时装其他改动播放的 HeadsetPlay 测试插件。覆盖安装前请确认包管理器识别为同一包并保留设置；配置是否被包管理器操作保留依赖具体操作，可截图备份配置，卸载可能影响配置。

## 手动 GitHub 构建

请用新建的独立仓库，不要修改原仓库。将此 ZIP **解压**，上传 `headsetplay-playback-diag` 目录内部所有内容到仓库根目录，包括隐藏的 `.github` 文件夹；不是上传 zip 本身或套一层目录。检查 GitHub 网页根目录同级应能看到 `Makefile`、`control`、`build.sh`、`Sources/`、`Hooks/`、`.github/`，且文件 `.github/workflows/build.yml` 必须存在且非空。不能让 `.github` 是普通文件；不能把本工程混入旧项目。

GitHub → Actions → Build HeadsetPlay playback diagnostic → Run workflow，选上传代码的分支并运行。完成后打开该次运行底部 Artifacts，下载 `headsetplay-10-diag1-rootless`，解压得到 deb。只上传构建产物，不创建 Release、不自动推送、不修改任何远程仓库。该流程尚未在 GitHub 实测，失败请保留完整 Actions 构建日志。iOS 16.3 Dopamine 用 rootless deb。安装前确认显示包名 `com.axs.headsetplay`、版本 `5.2.0-10+diag1`；不要误装其他项目或旧版本。安装替换后按包管理器提示重启 SpringBoard/用户空间，保持原设置的插件启用、网易云、自动播放等条件，记下“播放中跳过”当前状态，不为测试任意改变。

## 采集一组首次失败和二次成功

用 Mac「控制台」连接 iPhone，选中该设备并开始实时收集，在搜索栏选择消息包含 `HSPPLAYDIAG`；显示 Info/Debug（若可选择）。先观察是否有 `HSPPLAYDIAG START diagnostic 10+diag1 SpringBoard`，有时需在开始收集后重启 SpringBoard/用户空间才能看到。没看到不能直接判定没注入，也可能是日志过滤/连接/采集时机问题。保留带时间戳的完整日志，勿只截最后一行。

保持耳机在盒内，手机与控制台连接并收集；执行用户空间重启，等待 18 秒后第一次取出耳机，保持原先的佩戴、弹窗及网易云状态，不手动点击播放，额外等待至少 20 秒。记下取出时间、是否有声音、是否手动点过。放回耳机，确认断开并等待至少 10 秒；第二次取出，按同样条件再等 20 秒，记录是否播放。可在完成这两轮后另做手动播放作为对照，标注时间，避免污染自动播放结果。将从 START/基线建立到第二轮结束的所有 HSPPLAYDIAG 行导出或连续截图（包括时间和进程），发送并标注每次取出/放回、结果与时间，说明其他连接的蓝牙音频设备及使用的开关。不要发送未筛选的全部系统日志、账号或设备地址；本埋点不主动记录地址、名称或播放内容，但系统原始日志可能含敏感信息，分享前检查。

如果重启期间 Mac 断开，重连并重新开始收集。若没看到 START 或 baseline，把能看到的行、重连时间和操作结果一并提供；实时日志无法补回断线期间的事件，不要把日志缺失等同于代码没运行。用完后可安装保留的 `5.2.0-10` 原始 deb；因为是降级，可能需包管理器确认或选择降级。恢复后检查版本、设置并依提示重启。不要删除诊断日志直到排查完成。

日志判读只作定位：baseline/status/newAudio 对应事件；run route timeout 表示等待耗尽；out of ear 表示当时设备状态拦截；query callback playing=1 是系统全局播放状态而非证明网易云播放；launch result 仅是 API 返回，send Play 仅说明调用、不是指定 App 已接收。query 有发起但无 callback 时也可观察日志中断位置，系统采集丢行则需重测。日志不能单独证明根因。
