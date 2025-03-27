
# アプリ設計書: Hiveを使ったデータ管理

## 1. はじめに
本アプリは、Flutterを使い、ローカルデータベースにHiveを用いてタスク管理を行います。タスク情報を保存し、後で簡単に取得して表示する機能を提供します。

---

## 2. 依存関係の設定

### 2.1 必要なパッケージ
以下のパッケージを`pubspec.yaml`ファイルに追加し、依存関係をインストールします。

```yaml
dependencies:
  hive: ^2.0.4
  hive_flutter: ^1.1.0
  path_provider: ^2.0.8
```

- **hive**: ローカルデータベース（キー・バリュー型データベース）
- **hive_flutter**: Flutter専用のHiveパッケージ
- **path_provider**: アプリのローカルストレージパスを取得

---

## 3. Hiveのセットアップ

### 3.1 初期化
`main.dart`内でHiveを初期化します。`path_provider`を使ってローカルストレージのパスを取得し、Hiveのデータベースをそのパスに保存します。

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  var appDocumentDir = await getApplicationDocumentsDirectory();
  Hive.init(appDocumentDir.path);
  runApp(MyApp());
}
```

- **getApplicationDocumentsDirectory**: デバイスのローカルストレージ内にアプリデータを保存するためのディレクトリパスを取得します。
- **Hive.init**: Hiveの初期化を行い、指定したパスにデータを保存する準備をします。

---

## 4. データモデルの作成

### 4.1 タスクデータモデル
タスクを保存するためのデータモデル`Task`を作成します。このモデルは、Hiveで保存できるように`TypeAdapter`を使用して定義します。

```dart
@HiveType(typeId: 0)
class Task {
  @HiveField(0)
  final String title;

  @HiveField(1)
  final int priority;

  @HiveField(2)
  final DateTime deadline;

  Task({required this.title, required this.priority, required this.deadline});
}
```

- **@HiveType(typeId: 0)**: `HiveType`アノテーションで、データモデルをHiveで管理できるようにする。`typeId`はデータモデルに一意のIDを与えます。
- **@HiveField(x)**: 各フィールドに対応する`HiveField`アノテーション。`x`はフィールドのインデックス番号です。

---

## 5. データの保存と取得

### 5.1 データの保存
タスクをHiveデータベースに保存するには、以下のようにします。まずは、タスク用のボックスを開き、データを追加します。

```dart
var box = await Hive.openBox<Task>('taskBox');
var task = Task(title: 'Sample Task', priority: 1, deadline: DateTime.now());
await box.add(task);
```

- **Hive.openBox**: 指定した名前のボックス（テーブル）を開きます。データはそのボックスに保存されます。
- **box.add**: 新しいタスクをボックスに追加します。

### 5.2 データの取得
保存したタスクを取得する方法です。タスクをインデックスで取得することができます。

```dart
var task = box.getAt(0); // 0番目のタスクを取得
```

- **box.getAt**: ボックス内の指定したインデックス番号のデータを取得します。

---

## 6. UIの実装

### 6.1 タスクの表示
タスクのリストを表示するために、`ListView`を使用して、保存したタスクをリストとして表示します。

```dart
ListView.builder(
  itemCount: box.length,
  itemBuilder: (context, index) {
    var task = box.getAt(index);
    return ListTile(
      title: Text(task?.title ?? ''),
      subtitle: Text('Priority: ${task?.priority}'),
    );
  },
);
```

- **ListView.builder**: データのリストを表示するウィジェット。`itemCount`でデータの数を指定し、`itemBuilder`で各アイテムを作成します。

---

## 7. 注意事項

- **データ型の変更**: タスクデータモデルを更新する際は、`typeId`と`HiveField`のインデックス番号が一致していることを確認してください。
- **データの整合性**: Hiveでは、データの整合性は手動で管理する必要があるため、タスクの追加・更新・削除時に注意してください。

---

## 8. 今後の拡張

- **タスクの編集・削除機能**: UIに編集ボタンや削除ボタンを追加し、タスクの変更を可能にする。
- **データの並び替え**: タスクを優先度や期限で並び替える機能を追加する。
- **クラウド同期**: 今後、Hiveデータをクラウドで同期させる機能を追加することも考えられる。

---
