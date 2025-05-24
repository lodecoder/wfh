# 概要

output-repomix-dot-env.xml はタスク管理ワークフローです。
これをサポートするためのコマンドラインツールを作成しようとしています。
以下はそのツールの仕様書です。

# workflow-helper (wfh) 開発環境

- dotnet core (.NET 8)
- Win32 API を使うので下記の設定
	```csproj
	  <PropertyGroup>
	    <OutputType>WinExe</OutputType>
	    <TargetFramework>net8.0</TargetFramework>
	    <ImplicitUsings>enable</ImplicitUsings>
	    <Nullable>enable</Nullable>
	    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
	  </PropertyGroup>
	```

# 仕様

## 1. まず、ワークスペースを指定する。

`{working directory}/.dev/workspace-[0-9]+/` にマッチするフォルダー (`workspace-template[^/]*/`, `workspace-example[^/]*/` は除外) を探す。
見つからなければフォルダ階層が上の `{working directory}/../.dev/workspace-[0-9]+/` にマッチするフォルダーを探す。親フォルダを再帰的に探索。
見つかったら下記のように選択肢を出して Console.ReadLine() で数字を入力して指定 (選択肢にないものを入力した場合は、再度選択肢を表示して再入力を繰り返す。最後まで見つからなかった場合はワークスペース新規作成orパスを入力)
```sh
1. root/.dev/workspace-1
2. root/.dev/workspace-2
3. (新規作成)
4. (入力して指定...)
```
"(新規作成)" の場合は working directory 直下に ".dev/" がある場合、または working directory が ".dev/" の場合は ".dev/workspace-template/" を ".dev/workspace-${sequence}/" にコピーしてそこをワークスペースとする (${sequence} の部分は 1 から win32api で作成して例外が起こらないパスをインクリメントして自動で決定する)。また、コピー後に、コピーしたワークスペースの `.dev/workspace-${sequence}/misc/.gitignore` を `.dev/workspace-${sequence}/.gitignore` に移動する。
"(入力して指定...)" を選んだ場合は ConsoleReadLine() でパスを入力し、そのパスが (`/\.dev\/workspace-[0-9]+\/?$/`) にマッチする場合はそれをワークスペースとして選ぶ
	
※ `wfh auto` または `wfh a` で起動した場合は、 見つかった `workspace-*/` が1つの場合はそれを自動で選択する。
※ `wfh force` または `wfh f` で起動した場合は、見つかった `workspace-*/` のうち ${sequence} が一番大きいものを選択。

ワークスペースが決定したら
- ワークスペースを working directory としてセット
- コンソールタイトルに `wfh - ワークスペース: ${path}` をセット。

## 2. 選択したワークスペース上でコマンドを実行する

1. コマンドリストをコンソール出力する
2. 現在のワークスペースを出力
3. コマンドの入力を要求。 Console.ReadLine() でコマンドと引数を入力 (入力の前後の空白は trim)
3. コマンドを実行して (コマンドが見つからなければ何もせず)  1. に戻る (無限ループ)

というシンプルなもの。

# コマンドリスト

1. diff-order
  - 動作: 英語と日本語の order md を `"%ProgramFiles%/WinMergeU.exe" "<path1>" "<path2>"` で開く
  - 引数: なし
  - コマンドリストでの表示: `1. diff-order  # 指示書の日英比較`
  - コマンドリストに表示する条件: `order-.*(?<!\.ja)\.md` と と、このファイル名の `.md` を `.ja.md` に置換したパス に2つともファイルが存在する場合
2. diff-plan
  - 動作: 英語と日本語の plan md を `"%ProgramFiles%/WinMergeU.exe" "<path1>" "<path2>"` で開く
  - 引数: なし
  - コマンドリストでの表示: `2. diff-plan  # 計画の日英比較`
  - コマンドリストに表示する条件: `plan-.*(?<!\.ja)\.md` と、このファイル名の `.md` を `.ja.md` に置換したパス に2つともファイルが存在する場合
3. make-order
  - 動作: `.dev/rules/make-order-instruction.md` を指示するためのプロンプト `make-order-prompt.md` をクリップボードにコピー
    - 「make-order-prompt をクリップボードにコピーしました」とメッセージ表示
  - 引数: なし
  - コマンドリストでの表示: `3. make-order  # make-order-prompt をクリップボードにコピー`
  - コマンドリストに表示する条件: `draft-.*(?<!\.ja)\.md` のファイルが存在する かつ `order-.*(?<!\.ja)\.md` のファイルが存在しない場合
4. make-plan
  - 動作: `.dev/rules/make-plan-instruction.md` を指示するためのプロンプト `make-plan-prompt.md` をクリップボードにコピー
    - 「make-plan-prompt をクリップボードにコピーしました」とメッセージ表示
  - 引数: なし
  - コマンドリストでの表示: `4. make-plan  # make-plan-prompt をクリップボードにコピー`
  - コマンドリストに表示する条件: `order-.*(?<!\.ja)\.md` のファイルが存在する かつ `plan-.*(?<!\.ja)\.md` のファイルが存在しない場合
5. step
  - 動作: `.dev/rules/step-instruction.md` を指示するためのプロンプト `step-prompt.md` をクリップボードにコピー
    - 「step-prompt をクリップボードにコピーしました」とメッセージ表示
  - 引数: なし
  - コマンドリストでの表示: `5. step  # step-prompt をクリップボードにコピー`
  - コマンドリストに表示する条件: `plan-.*(?<!\.ja)\.md` のファイルが存在する場合
6. tidy
  - 動作: retrospective.md を作成する指示のプロンプト `tidy-prompt.md` をクリップボードにコピー
    - 「tidy-prompt をクリップボードにコピーしました」とメッセージ表示
  - 引数: なし
  - コマンドリストでの表示: `6. tidy  # tidy-prompt をクリップボードにコピー`
  - コマンドリストに表示する条件: `step-.+\.md` にマッチするファイルが1つでも存在する場合
7. breakdown-todo
  - 動作: `.dev/rules/breakdown-todo-instruction.md` を実行するためのプロンプト `breakdown-todo-prompt.md` をクリップボードにコピー
    - 「breakdown-todo-prompt をクリップボードにコピーしました」とメッセージ表示
  - 引数: なし
  - コマンドリストでの表示: `7. breakdown-todo  # タスク分割プロンプトのコピー`
8. repomix
  - 動作: `cd "workspace-*/" && repomix` をコマンド実行
  - コマンドリストでの表示: `8. repomix  # ワークスペースでrepomixを実行`
9. tree
  - 動作: ワークスペースをツリー表示 (最大4階層)
  - 引数(オプション): `tree --parent` または `tree -p` で ".dev/" をツリー表示
  - コマンドリストでの表示: `9. tree [--parent|-p]  # ワークスペースをツリー表示`
10. open-workspace
  - 動作: ワークスペースをエクスプローラーで開く
  - 引数: なし
  - コマンドリストでの表示: `10. open-workspace # ワークスペースをエクスプローラーで開く`
11. status
  - 動作: plan-*.md から steps の todo list を抽出してコンソールに表示
  - 引数: なし
  - コマンドリストでの表示: `11. status  # plan から todo list を表示`
  - コマンドリストに表示する条件: `plan-.*(?<!\.ja)\.md` のファイルが存在する場合
12. complete-workspace
  - 動作: ワークスペースを `.dev/done/${yyyy}/${yyyy-MM-dd}-${title}/` に移動 (移動先ディレクトリをwin32apiで作成、失敗時は中止)。移動成功したら再びワークスペースを決定する最初の処理へ戻る
  - 引数: なし
  - コマンドリストでの表示: `12. complete-workspace [title]  # ワークスペースを done フォルダへ移動して完了する`
  - コマンドリストに表示する条件: `step-.+\.md` にマッチするファイルが1つでも存在する場合
  - 引数(オプション): 作成するディレクトリの${title}部分の値 (ディレクトリ名に利用できない文字が含まれる場合は中止)
    - 引数を指定しなかった場合は Console.ReadLine で入力
13. help
  - 動作: 各コマンドのヘルプを表示する
  - 引数(オプション): ヘルプを表示するコマンドを引数に受ける(インクリメンタルサーチでコマンド検索)
  - コマンドリストでの表示: `13. help [command]  # コマンドのヘルプを表示`

※ 非表示のコマンドがあっても番号をそのぶん詰めないこと。
※ インクリメンタルサーチ(例: "diff-order" であれば /^di?f?f?-*or?d?e?r?$/i )でマッチするコマンド または コマンド番号のコマンド を実行 (複数のコマンドがマッチしたら処理はせず、マッチしたコマンドをリスト表示して、再度入力を促すようにする)

# その他の仕様

## コマンドの順番

IWorkspaceCommand を継承したコマンドクラスを表示したい順番で配列で定義する。コマンド番号は 配列のインデックス + 1 の値となる。

## エラー時の処理

kuri.exe, WinMerge.exe, repomix がインストールされていない場合、コマンド実行でエラーが発生した場合は、
 エラーメッセージ表示してください

## コマンドリストの表示形式

```sh
番号  コマンド名           説明
----  ----------------    ----------------------------------
1.    diff-order          # 指示書の日英比較
2.    diff-plan           # 計画の日英比較
3.    make-order          # make-order-prompt.md をクリップボードにコピー
```

番号は Cyan, 説明は Green でansiエスケープシーケンスで色をつける。

## statusコマンドの出力例
```
Plan: ログイン機能の実装
Progress: 3/7 steps completed

✓ Step 1: Login Interface Foundation
✓ Step 2: Client-Side Validation Implementation  
✓ Step 3: Backend API Integration Planning
   Step 4: Authentication Service Implementation
   Step 5: Session Management and Security
```

## プレースホルダーの仕様

- 置換できないプレースホルダーがある場合は警告を黄色の字で表示
- プレースホルダーにマッチする正規表現は `[GeneratedRegex(@"\$\{([0-9a-zA-Z_]*)\}")]`

### ${xxx} のプレースホルダーについて、使用可能な変数のリスト
- ${sequence} - ワークスペース番号 (`workspace-${sequence}` のように利用される)
- ${order_title}: 指示書のタイトル (`order-${order_title}.md` のように利用される)
- ${plan_title}: 計画のタイトル (`plan-${order_title}.md` のように利用される)

## ファイル名のマッチ

case insensitive でマッチするか判定。ディレクトリを含む処理の場合 `Path.GetFullPath` で正規化して、ワークスペースと前方一致する箇所は取り除く。

## コマンドの実装に利用するもの

- kuri.exe (`kuri --mode texts "aa a" bbb ccc` のように引数を指定すると引数の内容を改行で結合して ("aa a\nbbb\nccc") クリップボードに貼り付けるツール)
	- クリップボードに貼り付ける機能はこれを利用
	- プロンプトは `${workspace}/rules/prompts/*-prompt.md` から取得し、プレースホルダー `${xxx}` を置換して kuri でクリップボードに保存

## ワークスペース作成時の注意点

TOCTOU 競合を避けるため Win32 API を使用すること。下記は参考コード。
IOException がスローされた場合はディレクトリ作成失敗とみなす

````cs
using System.Runtime.InteropServices;

public static partial class DirectoryHelper
{
    [LibraryImport("kernel32.dll", SetLastError = true, StringMarshalling = StringMarshalling.Utf16)]
    private static partial bool CreateDirectoryW(string lpPathName, IntPtr lpSecurityAttributes);

    [LibraryImport("kernel32.dll")]
    private static partial uint GetLastError();

    private const uint ERROR_ALREADY_EXISTS = 183;
    private const uint ERROR_PATH_NOT_FOUND = 3;
    private const uint ERROR_ACCESS_DENIED = 5;

    public static void CreateDirectoryExclusively(string path)
    {
        // 日本語パスの場合、長いパス名に対応
        string fullPath = Path.GetFullPath(path);
        
        // \\?\ プレフィックスを使用して長いパス名をサポート
        if (fullPath.Length > 248)
        {
            fullPath = @"\\?\" + fullPath;
        }

        if (!CreateDirectoryW(fullPath, IntPtr.Zero))
        {
            uint error = GetLastError();
            string errorMessage = error switch
            {
                ERROR_ALREADY_EXISTS => $"ディレクトリが既に存在します: {path}",
                ERROR_PATH_NOT_FOUND => $"指定されたパスが見つかりません: {path}",
                ERROR_ACCESS_DENIED => $"ディレクトリ作成のアクセス権限がありません: {path}",
                _ => $"ディレクトリの作成に失敗しました: {path}. エラーコード: {error}"
            };

            if (error == ERROR_ALREADY_EXISTS)
            {
                throw new InvalidOperationException(errorMessage);
            }
            throw new IOException(errorMessage);
        }
    }
}
````

## ANSI エスケープシーケンスを有効化してコンソールに色をつけれるようにする

下記のコードは別のプロジェクトから持ってきた参考になりそうなコードです

```cs
    // VT モードを有効化する
    [SupportedOSPlatformGuard("windows")]
    public static void EnableVirtualTerminalProcessing() {
        IntPtr hConsoleHandle = GetConsoleOutputHandle();
        if (hConsoleHandle == IntPtr.Zero) return;
        if (!GetConsoleMode(hConsoleHandle, out var mode)) throw new IOException("コンソールモードの取得に失敗しました。");

        const uint ENABLE_VIRTUAL_TERMINAL_PROCESSING = 0x0004;
        if ((mode & ENABLE_VIRTUAL_TERMINAL_PROCESSING) == 0) {
            if (!SetConsoleMode(hConsoleHandle, mode | ENABLE_VIRTUAL_TERMINAL_PROCESSING)) throw new IOException("ENABLE_VIRTUAL_TERMINAL_PROCESSING の設定に失敗しました。");
        }
    }
```

## プレースホルダーの置換ロジックの参考資料

これは全く別のプロジェクトのものなのでそのまま使えないが、正規表現の使い方などは参考になるので記載しておく

```cs
using System.Text.RegularExpressions;
partial class MessageGenerator(DateTimeOffset Time, DateTimeOffset StartTime, DateTimeOffset EndTime) {
    public string Generate(string message) {
        var ret = MessageTemplateMatcher().Replace(message, m => {
            var keyword = m.Groups[1].Value.ToLowerInvariant();
            var format = m.Groups[2].Value.StartsWith(':') ? m.Groups[2].Value[1..] : null;

            return keyword switch {
                "remaining_seconds" => Math.Round((EndTime - Time).TotalSeconds).ToString(format ?? ""),
                "now" => Time.ToString(format ?? "HH:mm:ss"),
                "from" => StartTime.ToString(format ?? "HH:mm:ss"),
                "to" => EndTime.ToString(format ?? "HH:mm:ss"),
                _ => throw new InvalidOperationException($"Invalid keyword: {m.Value}"),
            };
        });
        ret = TagMatcher().Replace(ret, m => {
            var tag = m.Groups[1].Value.ToLowerInvariant();
            var content = m.Groups[2].Value;

            return tag switch {
                "red" => $"\u001b[31m{content}\u001b[0m",
                "yellow" => $"\u001b[33m{content}\u001b[0m",
                "green" => $"\u001b[32m{content}\u001b[0m",
                "blue" => $"\u001b[34m{content}\u001b[0m",
                "magenta" => $"\u001b[35m{content}\u001b[0m",
                "cyan" => $"\u001b[36m{content}\u001b[0m",
                "white" => $"\u001b[37m{content}\u001b[0m",
                "black" => $"\u001b[30m{content}\u001b[0m",
                _ => throw new InvalidOperationException($"Invalid tag: {m.Value}"),
            };
        });
        return ret;
    }

    [GeneratedRegex(@"{([0-9a-zA-Z_-]+)(|:[^}\r\n]+)}")]
    private static partial Regex MessageTemplateMatcher();

    [GeneratedRegex(@"<([a-z]+)>([^<]*)</\1>")]
    private static partial Regex TagMatcher();
}
```
