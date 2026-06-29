# LocalSend 贡献笔记

## 项目结构

Flutter 桌面/移动端文件传输应用，底层用 Rust (flutter_rust_bridge) 处理网络和加密。

```
app/          — Flutter 主应用
common/       — 共享模型和工具
app/macos/    — macOS 原生工程 (Xcode)
app/lib/gen/  — 生成代码 (.g.dart, .mapper.dart)，不可手改
```

## 构建

```bash
cd app
flutter pub get
dart run build_runner build -d   # 重新生成 .g.dart / .mapper.dart
flutter run -d macos
```

FVM 配置在 `.fvm/fvm_config.json`，指定 Flutter 版本。

## 设置 (Settings) 模式

- `persistence_provider.dart` — SharedPreferences 读写封装
- `settings_state.dart` + `.mapper.dart` — 由 `dart_mappable` 生成的不可变状态类 + copyWith
- `settings_provider.dart` — PureNotifier，封装 state 变更

**关键教训：mapper 文件是生成的，绝对不要手改。** 手改导致 copyWith 参数不对齐，会引发数据错乱。如需不碰 mapper 的简单设置，用独立 PureNotifier Provider（参考 `auto_copy_provider.dart`）。

## i18n

- 新 key 只加到 `app/assets/i18n/en.json`
- `app/lib/gen/strings_en.g.dart` 是生成的，但无 build_runner 时可手补一两个 getter（临时方案）
- 其他语言由翻译者通过 `_missing_translations_*.json` 补齐

## CI

```yaml
on:
  push: [main]
  pull_request: [main]
```

- format job：`rm -rf lib/gen` 后 `dart format --set-exit-if-changed`
- test job：`flutter analyze` + `flutter test`
- 上游 `favorite_dialog.dart` 和 `favorite_edit_dialog.dart` 有历史格式问题

## 测试

CONTRIBUTING.md 要求 "All changes should be covered by tests." 用 `package:test/test.dart`。

## PR 规范

- fork → feature 分支 → PR 到 `localsend/localsend:main`
- `Closes #issue号` 自动关闭 issue
- 首次贡献者需维护者批准 CI
