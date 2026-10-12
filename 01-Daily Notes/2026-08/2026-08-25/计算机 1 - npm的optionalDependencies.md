这里是跨平台 Node 模块**「多包发布」**与**「按需匹配安装」**的全生命周期 Mermaid 流程图：

```mermaid
flowchart TD
    %% ================= 第一部分：发布阶段 =================
    subgraph 发布阶段 ["阶段一：开发者 CI/CD 并行编译与多包发布"]
        style 发布阶段 fill:#f0f7ff,stroke:#0366d6,stroke-width:2px

        DEV[开发者提交代码 / Tag] --> CI[CI/CD 矩阵构建 Matrix Build]
        
        CI --> M1["macOS 编译器<br/>(编译 arm64 二进制)"]
        CI --> M2["Linux 编译器<br/>(编译 x64 二进制)"]
        CI --> M3["Windows 编译器<br/>(编译 x64 exe)"]
        
        M1 --> P1["发布子包 1: <br/><b>@pkg/darwin-arm64</b><br/><i>(声明 os: darwin, cpu: arm64)</i>"]
        M2 --> P2["发布子包 2: <br/><b>@pkg/linux-x64</b><br/><i>(声明 os: linux, cpu: x64)</i>"]
        M3 --> P3["发布子包 3: <br/><b>@pkg/win32-x64</b><br/><i>(声明 os: win32, cpu: x64)</i>"]
        
        P1 & P2 & P3 --> P_MAIN["所有子包就绪后<br/>发布主包: <b>pkg (仅含 JS 胶水代码)</b><br/><i>(在 optionalDependencies 中引用所有子包)</i>"]
    end

    %% 发布到 NPM
    P1 & P2 & P3 & P_MAIN --> REGISTRY[("📦 NPM Registry 官方仓库<br/>(存放 1 个主包 + N 个独立平台子包)")]

    %% ================= 第二部分：安装阶段 =================
    subgraph 安装阶段 ["阶段二：用户端 npm install 按需匹配与过滤"]
        style 安装阶段 fill:#f6ffed,stroke:#52c41a,stroke-width:2px

        USER["用户执行: npm install pkg<br/>(假设当前为: Mac M系列芯片)"] --> GET_MAIN["1. 下载主包 pkg<br/>并解析 optionalDependencies 列表"]
        REGISTRY -. 下载主包 .-> GET_MAIN
        
        GET_MAIN --> CHECK_META["2. 获取各子包元数据 (纯 JSON，不含二进制)<br/>读取各包的 os / cpu 声明"]
        REGISTRY -. 获取元数据 .-> CHECK_META

        CHECK_META --> MATCH{"3. 与当前机器系统比对<br/>process.platform / process.arch"}
        
        MATCH -- "符合 (darwin + arm64)" --> DL_SUB["✅ <b>命中匹配</b><br/>发起网络下载并解压<br/><b>@pkg/darwin-arm64 (~8MB)</b>"]
        REGISTRY -. 仅下载匹配的子包 .-> DL_SUB

        MATCH -- "不符合 (linux / win32)" --> SKIP["❌ <b>平台不匹配 (EBADPLATFORM)</b><br/>由于属于 optionalDependencies<br/><b>直接静默忽略，不发起下载</b> (0MB)"]

        DL_SUB --> NODE_MODULES["4. 写入 node_modules: <br/>• node_modules/pkg<br/>• node_modules/@pkg/darwin-arm64"]
    end

    %% ================= 第三部分：运行阶段 =================
    subgraph 运行阶段 ["阶段三：运行时动态调用"]
        style 运行阶段 fill:#fffbe6,stroke:#faad14,stroke-width:2px

        NODE_MODULES --> RUN["用户代码执行 require('pkg')"]
        RUN --> DYN["主包根据 process.platform 动态寻址:<br/><code>require('@pkg/darwin-arm64')</code>"]
        DYN --> EXEC["⚡️ 秒级调用本地原生二进制运行"]
    end
```

---

### 核心亮点总结：

1. **发布侧（独立解耦）**：NPM 仓库里实际存在 $N+1$ 个包（1 个入口主包 + $N$ 个各操作系统平台的独立二进制子包）。
2. **安装侧（网络与磁盘极简）**：用户机器只获取元数据做 `os/cpu` 比对，**只产生 1 次对应平台的二进制文件网络下载**，其余平台的几十个包均被静默跳过，保证安装速度与体积最小化。
3. **运行侧（透明无感）**：主包通过 `process.platform` 动态 `require` 唯一定位并调用那个已被成功安装的原生二进制。