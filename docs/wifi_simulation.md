# 模拟 Wi-Fi 连接功能设计

本文档提供在 Android App 中实现“模拟 Wi-Fi 连接”演示功能的参考方案，适用于教学、演示或仿真场景（不实际修改系统 Wi-Fi 连接状态）。

## 需求拆解
- 扫描附近 Wi-Fi 列表并展示 SSID、信号强度、加密类型。
- 用户点击某个 Wi-Fi 时播放“连接中”动画/进度，随后显示“连接成功”状态。
- 连接成功后显示模拟的网络信息（如 IP、MAC、信号强度），并支持断开/重试。
- 可扩展：模拟网络请求（如 ping）、断开事件、信号变化。

## 执行步骤
1. **获取 Wi-Fi 列表**
   - 动态申请 `ACCESS_FINE_LOCATION`、`ACCESS_COARSE_LOCATION`、`ACCESS_WIFI_STATE`、`CHANGE_WIFI_STATE` 权限（Android 10+ 需要前台/后台定位权限）。
   - 确保 Wi-Fi 已开启后调用 `WifiManager.startScan()` 与 `getScanResults()`，生成展示列表。
2. **展示 Wi-Fi 列表**
   - 使用 `RecyclerView` + `ListAdapter` 显示 SSID、信号强度、加密类型。
   - 点击某个 item 进入“模拟连接”界面或弹出 BottomSheet。
3. **模拟连接过程**
   - 切换到“正在连接”状态（进度条/动画）。
   - `Handler.postDelayed` 或 `Coroutine` 延时 2–3 秒，随后将状态切换为“连接成功”。
   - 生成模拟网络数据（伪造的 IP、MAC、速率、信号等级），渲染到 UI。
4. **后续交互（可选）**
   - 提供“断开连接”按钮，将状态重置为初始。
   - 模拟网络请求：例如延时后显示“已完成 ping www.example.com 120ms”。

## 参考实现（Kotlin 伪代码）
```kotlin
class WifiSimulationActivity : AppCompatActivity() {
    private val wifiManager by lazy { applicationContext.getSystemService(WIFI_SERVICE) as WifiManager }
    private val adapter = WifiListAdapter(::onWifiSelected)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_wifi_simulation)
        recyclerView.adapter = adapter
        requestPermissions()
        loadWifiList()
    }

    private fun loadWifiList() {
        if (!wifiManager.isWifiEnabled) wifiManager.isWifiEnabled = true
        val results = wifiManager.scanResults
        adapter.submitList(results)
    }

    private fun onWifiSelected(result: ScanResult) {
        showConnectingState(result.SSID)
        lifecycleScope.launch {
            delay(2500)
            val mockInfo = MockWifiInfo.from(result)
            showConnectedState(mockInfo)
        }
    }
}
```

## 布局与状态建议
- `RecyclerView` item：SSID、信号图标、加密标签、点击区域。
- “连接中”视图：圆形进度或连接动画，展示“正在连接 <SSID>…”。
- “连接成功”视图：显示伪造 IP/MAC/速率/信号等级，附“断开连接”“重新选择”按钮。

## 注意事项
- 该功能不修改系统 Wi-Fi 连接状态，仅提供 UI 层面的仿真。
- 需要在应用内解释“此为演示功能，不会更改真实网络连接”。
- 如果需要配合教程/演示，可在成功页面显示提醒或引导文案。
