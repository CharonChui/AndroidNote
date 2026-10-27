# 02 Dart 语言速成（Kotlin 对照）

Dart 的语法对 Kotlin/Java 开发者非常友好，1 天足够上手。本章只讲写 Flutter 必须掌握的部分。

在线练习：[dartpad.dev](https://dartpad.dev)，或在工程里新建 `bin/test.dart` 后执行 `dart run bin/test.dart`。

## 2.1 变量与类型

```dart
// Kotlin: val name = "Tom"     → Dart: final
final name = 'Tom';
// Kotlin: const val MAX = 10   → Dart: const（编译期常量）
const max = 10;
// Kotlin: var age = 18         → Dart: var
var age = 18;
// 显式类型
String city = 'Beijing';
int count = 0;           // 64 位整数（Web 上是 JS number）
double price = 9.9;
num any = 1;             // int 和 double 的父类
bool ok = true;
dynamic d = 1; d = 'x';  // 关闭类型检查，尽量不用
Object o = 1;            // ≈ Kotlin Any

// 延迟初始化，≈ lateinit var
late String token;
// 懒加载，≈ by lazy
late final config = loadConfig();
```

`final` 与 `const` 的区别：`final` 是运行期只赋值一次；`const` 是编译期常量，且 `const` 对象是规范化（canonicalized）的，相同的 `const Text('a')` 在内存中只有一份。Flutter 中对 Widget 使用 `const` 能跳过重建。

## 2.2 空安全（与 Kotlin 几乎一致）

```dart
String? nickname;                    // 可空
int len = nickname?.length ?? 0;     // ?. 和 ?:（Dart 写作 ??）
nickname ??= 'guest';                // 为空时赋值
String s = nickname!;                // 断言非空，≈ !!

// 类型提升（≈ Kotlin smart cast），只对局部变量生效
void f(String? s) {
  if (s == null) return;
  print(s.length);                   // 这里 s 已被提升为 String
}
```

坑：类的字段不能被智能提升（因为可能被子类 getter 覆盖），需要先赋给局部变量：

```dart
class A {
  String? name;
  void p() {
    final n = name;          // 先取局部变量
    if (n != null) print(n.length);
  }
}
```

## 2.3 字符串与集合

```dart
final s = 'Hello $name, ${user.age + 1} 岁';   // 插值
final raw = r'C:\path';                          // 原始字符串
final multi = '''
多行
''';

// List ≈ MutableList，字面量 []
final list = <int>[1, 2, 3];
final fixed = const [1, 2];          // 不可变
// Map
final map = <String, int>{'a': 1};
// Set
final set = <String>{'a', 'b'};

// 集合操作（命名与 Kotlin 不同）
list.map((e) => e * 2).toList();     // map 返回惰性 Iterable，记得 toList()
list.where((e) => e > 1);            // ≈ filter
list.firstWhere((e) => e > 1, orElse: () => -1);
list.fold<int>(0, (acc, e) => acc + e);
list.any((e) => e > 2);  list.every((e) => e > 0);
list.expand((e) => [e, e]);          // ≈ flatMap
// Dart 3 起 package:collection 提供 firstOrNull 等，也可直接用 list.firstOrNull

// collection if / for / spread（构建 Widget 列表时非常常用）
final items = [
  'header',
  if (isLogin) 'profile',
  for (final i in list) 'item$i',
  ...?extraList,
];
```

## 2.4 函数

```dart
// 普通函数
int add(int a, int b) => a + b;

// 命名参数（Flutter 大量使用），用 {} 包裹；required 表示必传
void request({required String url, int timeout = 30, Map<String, String>? headers}) {}
request(url: 'https://x', timeout: 10);

// 可选位置参数 []
String greet(String name, [String? title]) => '${title ?? ''}$name';

// 函数是一等公民
final square = (int x) => x * x;
void run(void Function(int) cb) => cb(1);
typedef OnTap = void Function(int index);   // ≈ Kotlin typealias
```

注意：Dart 没有 Kotlin 那样的尾随 lambda 语法，回调都写在括号里，例如 `onPressed: () {}`。

## 2.5 类

```dart
class User {
  final String name;
  final int age;
  String? _token;              // _ 开头：库（文件）私有，不是类私有

  // 构造函数参数直接给字段赋值
  User(this.name, this.age);

  // 命名构造函数（Dart 没有重载，用命名构造代替）
  User.guest() : name = 'guest', age = 0;

  // 工厂构造：可返回缓存实例或子类
  factory User.fromJson(Map<String, dynamic> json) =>
      User(json['name'] as String, json['age'] as int);

  // getter / setter
  bool get isAdult => age >= 18;
  set token(String v) => _token = v;

  @override
  String toString() => 'User($name, $age)';
}

// const 构造：所有字段必须 final
class Point {
  final double x, y;
  const Point(this.x, this.y);
}
```

Dart 没有 `data class`。`==`/`hashCode`/`copyWith` 需要手写，或用 `freezed` / `equatable` 包生成（见第 5 章）。

### 继承、接口、mixin

```dart
abstract class Animal {          // 抽象类
  void speak();
}

class Dog extends Animal {       // 单继承
  @override
  void speak() => print('wang');
}

// Dart 中每个类都隐式定义了一个接口，可以 implements 任意类
class MockDog implements Dog {
  @override
  void speak() {}
}

// mixin：横向复用代码，≈ 带状态的 Kotlin 接口默认实现
mixin Logger {
  void log(String msg) => print('[${runtimeType}] $msg');
}
class Service with Logger {}

// Flutter 中最常见的 mixin
// class _PageState extends State<Page> with SingleTickerProviderStateMixin {}
```

Dart 3 的类修饰符：`sealed`、`final`、`base`、`interface`、`abstract`。`sealed` 与 Kotlin 的 sealed class 用法相同，配合 `switch` 做穷举。

### 扩展函数

```dart
extension StringX on String {
  bool get isPhone => RegExp(r'^1\d{10}$').hasMatch(this);
}
'13800000000'.isPhone;
```

### 级联操作符（≈ Kotlin apply）

```dart
final paint = Paint()
  ..color = Colors.red
  ..strokeWidth = 2;
```

## 2.6 Dart 3：Records、模式匹配、sealed

```dart
// Records：轻量元组，≈ Kotlin Pair/Triple，但可命名字段
(int, String) pair() => (1, 'a');
({int code, String msg}) result() => (code: 0, msg: 'ok');

final (code, msg) = pair();              // 解构
final r = result();
print(r.code);

// sealed + switch 表达式：编译器检查穷举（≈ Kotlin when + sealed）
sealed class UiState {}
class Loading extends UiState {}
class Success extends UiState { final List<String> data; Success(this.data); }
class Failure extends UiState { final Object error; Failure(this.error); }

String render(UiState s) => switch (s) {
  Loading() => '加载中',
  Success(data: final d) when d.isEmpty => '空',
  Success(:final data) => '共 ${data.length} 条',
  Failure(:final error) => '出错: $error',
};

// 对 JSON 做模式匹配 + 校验
void parse(Map<String, dynamic> json) {
  if (json case {'name': String name, 'age': int age}) {
    print('$name $age');
  }
}
```

## 2.7 异步：Future / async / await / Stream

Dart 是单线程 + 事件循环模型，和 Android 主线程 `Looper` 很像。

```
         ┌─────────────────────────┐
main() → │ 同步代码执行完            │
         └───────────┬─────────────┘
                     ▼
     ┌─────── 事件循环 ───────────────────────────┐
     │ 1. 先清空 microtask 队列（scheduleMicrotask、│
     │    Future 完成后的 then 回调）                │
     │ 2. 再取一个 event（IO、Timer、手势、绘制帧）  │
     │ 3. 回到 1                                    │
     └──────────────────────────────────────────────┘
```

| Kotlin 协程 | Dart |
|---|---|
| `suspend fun` | `Future<T> f() async` |
| `withContext/await` | `await` |
| `Deferred<T>` | `Future<T>` |
| `Flow<T>` | `Stream<T>` |
| `StateFlow` | `ValueNotifier` / `BehaviorSubject`(rxdart) |
| `launch { }` | 直接调用不 await（fire-and-forget），或 `unawaited(f())` |
| `Dispatchers.IO` | 不需要：IO 由引擎在后台线程完成，Dart 只等结果 |
| `Dispatchers.Default` 做 CPU 密集计算 | `Isolate.run()` / `compute()` |
| `awaitAll` | `Future.wait([...])` |
| `try/catch` | `try/catch`，或 `future.catchError` |

```dart
Future<String> fetchUser() async {
  final resp = await dio.get('/user');   // 不阻塞 UI 线程
  return resp.data['name'];
}

Future<void> load() async {
  try {
    final results = await Future.wait([fetchUser(), fetchOrders()]);
  } on DioException catch (e) {
    print('网络错误 ${e.message}');
  } catch (e, stack) {
    print('$e\n$stack');
  } finally {
    // ...
  }
}

// 超时
await fetchUser().timeout(const Duration(seconds: 5));

// Stream
Stream<int> ticker() async* {
  for (var i = 0; ; i++) {
    await Future.delayed(const Duration(seconds: 1));
    yield i;
  }
}
final sub = ticker().listen((v) => print(v));
// 不用时一定要取消，否则泄漏（≈ 取消协程 Job）
await sub.cancel();

await for (final v in ticker().take(3)) { print(v); }
```

重点：`async` 不等于开线程。`await` 一个网络请求不会卡 UI，因为 socket IO 在引擎的 IO 线程完成；但你写一个 `for` 循环解析 5MB JSON，就会实实在在卡住 UI 线程，必须放进 Isolate。

## 2.8 Isolate：Dart 的“线程”

Isolate 有独立的内存堆，彼此**不共享内存**，只能通过消息通信（类似多进程，但开销远小于进程）。因此 Dart 里没有锁、没有 `synchronized`、没有数据竞争。

```dart
import 'dart:isolate';
import 'dart:convert';

// 最简单：Isolate.run（Dart 2.19+）
Future<List<Item>> parseItems(String body) {
  return Isolate.run(() {
    final list = jsonDecode(body) as List;
    return list.map((e) => Item.fromJson(e)).toList();
  });
}

// Flutter 提供的等价封装（Web 上会退化为在主线程执行）
// final items = await compute(parseFn, body);
```

使用规则：

- 传给 Isolate 的闭包不能捕获不可发送的对象（如 `BuildContext`、Socket、带原生句柄的对象）。
- 启动一个 Isolate 约几 ms 到十几 ms，适合大块计算；频繁的小任务考虑常驻 Isolate + `SendPort/ReceivePort`。
- 后台 Isolate 里调用插件（MethodChannel）需要先 `BackgroundIsolateBinaryMessenger.ensureInitialized(token)`（Flutter 3.7+）。

## 2.9 库与包

```dart
import 'package:dio/dio.dart';            // 三方包
import 'package:my_app/utils/log.dart';   // 本工程（推荐用 package: 绝对路径）
import '../model/user.dart';              // 相对路径
import 'dart:math' as math;               // 别名
import 'package:x/x.dart' show A, B;      // 只导入部分
import 'big_lib.dart' deferred as big;    // 延迟加载（Android 上需配合 deferred components）

// 一个库拆分为多个文件
part 'user.g.dart';                       // 代码生成常用
```

可见性只有两种：公开（默认）和以 `_` 开头的库私有。没有 `protected`/`internal`，需要时用注解 `@protected`（来自 `package:meta`，仅 lint 提示）。

## 2.10 常见坑（Kotlin 开发者易踩）

1. `map()` / `where()` 返回惰性 `Iterable`，不 `toList()` 会每次迭代都重新计算。
2. `==` 默认比较引用，自定义类要重写 `==` 和 `hashCode`，否则 Set/Map、状态比较都会出问题。
3. 整数除法：`5 / 2 == 2.5`（结果是 double），整除用 `5 ~/ 2`。
4. `switch` 在 Dart 3 不需要 `break`，也不会贯穿。
5. 没有 `?.let {}`，常用写法：`if (x case final v?) {...}` 或先赋给局部变量判空。
6. 异常：Dart 不强制 checked exception，`async` 函数中未捕获的异常会在 Future 中“悄悄”丢失，必须 await 或配置全局错误处理（见第 12 章）。
7. `late` 变量未初始化就访问会抛 `LateInitializationError`，不要滥用。

## 动手练习

1. 写一个 `sealed class Result<T>`（`Ok` / `Err`），用 switch 表达式处理。
2. 用 `async*` 写一个倒计时 Stream，10 秒结束后关闭。
3. 用 `Isolate.run` 计算 1～1 亿的和，对比直接在主 Isolate 计算时 UI 动画是否卡顿。
