---
title: "bevy基础"
date: 2024-06-06 22:59:02
categories: "OS"
tags: 
- "Rust"
- "bevy"
---

# Bevy 基础

## Bevy设置

### 加快编译优化

在`Cargo.toml`或`.cargo/config.toml`:

```toml
# 对依赖项启用最大优化，但不对我们的代码启用最大优化：
[profile.dev.package."*"]
opt-level = 3

# 在调试/开发模式下仅启用少量优化
[profile.dev]
opt-level = 1

# 实际发布模式下最积极的优化配置：
[profile.release]
lto = true # 设置为 true 时编译器会进行链接时间优化，但会增加编译时间。
opt-level = 3 # 设置优化级别为最高。
codegen-units = 1 # 控制了代码生成的单元数量。当设置为 1 时，整个 crate 会被编译成一个单独的代码生成单元。
incremental = false # 不启用增量编译：增量编译允许 Cargo 只重新编译自上次构建以来已更改的文件和依赖项，从而加快编译速度。
debug = false # 当设置为false时，编译器不会生成调试信息。
```

**警告！**如果您使用调试器（如`gdb`或`lldb`）来逐步执行代码，任何程度的编译器优化都会影响体验。您的断点可能会被跳过，并且代码流可能会以意想不到的方式跳来跳去。如果您想调试/逐步执行代码，您可能需要 `opt-level = 0`。

### 加速 Rust-Analyzer

*Powershell* 中键入 **rustup shwo** 来显示你的Rust构建工具链

```powershell
PS C:\Users\cnkay> rustup show
Default host: x86_64-pc-windows-msvc
rustup home:  D:\tools\SDK\rust\.rustup

stable-x86_64-pc-windows-msvc (default)
rustc 1.78.0 (9b00956e5 2024-04-29)
```

如果你使用的是 *MSVC* 工具链那么连接器是`link.exe`。

为了实现快速编译，需要设置非默认链接器。在 *VSCode* 中设置`settings.json` 添加：

```json
Windows:
"rust-analyzer.cargo.extraEnv": {
    "RUSTFLAGS": "-Clinker=rust-lld.exe"
}

Linux(mold):
"rust-analyzer.cargo.extraEnv": {
    "RUSTFLAGS": "-Clinker=clang -Clink-arg=-fuse-ld=mold"
}

Linux (lld):
"rust-analyzer.cargo.extraEnv": {
    "RUSTFLAGS": "-Clinker=clang -Clink-arg=-fuse-ld=lld"
}

```

## CI设置

`CI`是**GitHub Actions**的简称

## ECS 模式

ECS 是*实体(Entity)*、*组件(Component)*、*系统(System)*的简称，是一种将**数据**和**行为**分开的编程范式。

**系统(System)**

系统是处理实体和组件的逻辑单元，负责执行特定的功能或行为。

系统是基于组件的存在于否，以及它们的状态来执行逻辑单元。

系统通常是独立于特定实体的，可以处理多个具有相似组件结构的实体。

**实体(Entity)**

实体是系统中的基本对象，可以是游戏中的角色、物体或者其他有意义的实体。

实体本身通常只是一个标识符，没有行为或状态。

**组件(Component)**

组件是实体的属性或数据单元，描述了实体的特征和状态。

不同的组件可以包含不同类型的数据，例如位置、渲染信息、健康状态等。

一个实体可以关联多个组件，组件之间是相互独立的。

**总结**

**实体只是指向组件的唯一ID或引用，本质上他将所有的组件连接在一起，构成游戏中的单个对象。**

**组件只是存粹的数据结构，如坦克对象的大小和速度。**

**系统只是对具有特定组件功能的实体进行操作的功能。**

在Bevy中**实体(Entity)** 是指向一群**组件(Component)** 的唯一 **事物(things)**, 然后使用 **系统(System)** 处理其过程.

例如，一个实体可能有`位置(Position)`和`速度(Velocity)`组件，而另一个实体可能有`位置(Position)`和`UI`组件。系统是在一组特定组件上运行的逻辑， 你可能有一个运行在所有带有`位置(Position)`和`速度(Velocity)`组件的实体上的`移动`系统。

## ECS 中的数据 

### World

定义：*World*是**系统**和**实体**的集合。在`ECS`架构中，*World*是最高级别的组织单位，它包含了**游戏中的所有实体（Entity）以及处理这些实体的系统（System）**。

当程序启动时，默认会生成一个World,可以存在多个World对象。

### Entities/Components

概念上类似于在数据库或电子表格。

每一行数据就是一个对象，`(Entity)`与`ID`或者行号类似，`(Components)`类似于 表的*列*，可以有任意多个*行*。

```rust
// 实体
#[derive(Component)]
struct Xp(u32);

// 组件
#[derive(Component)]
struct Health {
    current: u32,
    max: u32,
}

// 系统
fn level_up(mut query: Query<(&mut Xp, &mut Health)>,) {
    // process all relevant entities
    for (mut xp, mut health) in query.iter_mut() {
        if xp.0 > 1000 {
            xp.0 -= 1000;
            health.max += 25;
            health.current = health.max;
        }
    }
}
```

### Resources

如果某物只有一个全局实例（单例），并且它是独立（不与其他数据关联），那么你就可以创建一个`Resources`

例如，您可以创建一个资源来存储游戏的图形 设置，或指向非 Bevy 库的接口。

这是一种存储数据的简单方法，当你不需要`实体/组件`的灵活性的时候。

```rust
#[derive(Resource)]
struct GameSettings {
    current_level: u32,
    difficulty: u32,
    max_time_seconds: u32,
}

fn setup_game(
    mut commands: Commands,
) {
    // Add the GameSettings resource to the ECS
    // (if one already exists, it will be overwritten)
    commands.insert_resource(GameSettings {
        current_level: 1,
        difficulty: 100,
        max_time_seconds: 60,
    });
}

fn spawn_extra_enemies(
    mut commands: Commands,
    // we can easily access our resource from any system
    game_settings: Res<GameSettings>,
) {
    if game_settings.difficulty > 50 {
        commands.spawn((
            // ...
        ));
    }
}
```
