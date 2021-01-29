---
title: launch.vs.json スキーマ リファレンス (C++)
description: '`launch.vs.json` ファイルのスキーマ要素について説明します'
ms.date: 12/02/2020
helpviewer_keywords:
- launch.vs.json file [C++]
ms.openlocfilehash: 0338f3c8c6bf9eff73a0464ad6c9aac777afacc2
ms.sourcegitcommit: 3d9cfde85df33002e3b3d7f3509ff6a8dc4c0a21
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/21/2021
ms.locfileid: "98667475"
---
# <a name="launchvsjson-schema-reference-c"></a>launch.vs.json スキーマ リファレンス (C++)

*launch.vs.json* ファイルを使用してデバッグ パラメーターを構成します。 ファイルを作成するには、 **ソリューション エクスプローラー** で実行可能ファイルを右クリックし、 **[デバッグ設定と起動設定]** を選択します。 ご自分のプロジェクトに最も近いオプションを選択した後、次のプロパティを使用して必要に応じて構成を変更します。 CMake プロジェクトのデバッグについて詳しくは、「[CMake デバッグ セッションを構成する](./configure-cmake-debugging-sessions.md)」を参照してください。

## <a name="default-properties"></a>既定のプロパティ

|プロパティ|種類|説明|
|-|-|-|
|`args`|array|起動されたプログラムに渡されるコマンドライン引数を指定します。|
|`buildConfigurations`|array| 構成を適用するビルド モードの名前を指定するキーと値のペア。 たとえば、`Debug` または `Release` と、選択したビルド モードに応じて使用する構成などです。
|`currentDir`|string|ビルド ターゲットへの完全なディレクトリ パスを指定します。 これは、このパラメーターが設定されていない限り、自動的に検出されます。|
|`cwd`|string|プログラムが実行されるリモート システム上のディレクトリへの完全なパス。 既定値は  `"${debugInfo.defaultWorkingDirectory}"` です|
|`debugType`|string|コードの種類 (ネイティブ、マネージド、または混合) に応じてデバッグ モードを指定します。 これは、このパラメーターが設定されていない限り、自動的に検出されます。 使用できる値: `"native"`、`"managed"`、`"mixed"`。|
|`env`|array| カスタム環境変数のキーと値のリストを指定します。 たとえば、 `env:{"myEnv":"myVal"}`と指定します。|
|`inheritEnvironments`|array|複数のソースから継承された環境変数のセットを指定します。 *`CMakeSettings.json`* や *`CppProperties.json`* などのファイルでいくつかの変数を定義し、デバッグ コンテキストで使用できるようにすることができます。  **Visual Studio 16.4:** `env.VARIABLE_NAME` 構文を使用して、ターゲットごとに環境変数を指定します。 変数を設定解除するには、これを `"null"` に設定します。|
|`name`|string|**[スタートアップ アイテム]** ドロップダウンでエントリの名前を指定します。|
|`noDebug`|boolean|起動されたプログラムをデバッグするかどうかを指定します。 指定されない場合、このパラメーターの既定値は **`false`** です。|
|`portName`|string|実行中のプロセスにアタッチするときのポートの名前を指定します。|
| `program`|string|実行するデバッグ コマンドです。 既定値は  `"${debugInfo.fullTargetPath}"` です。|
|`project`|string|プロジェクト ファイルへの相対パスを指定します。 通常、CMake プロジェクトをデバッグするときに、この値を変更する必要はありません。 |
|`projectTarget`|string|`project` のビルド時に呼び出される省略可能なターゲットを指定します。 ターゲットは、 **[スタートアップ アイテム]** ドロップダウンの名前と一致している必要があります。|
|`stopOnEntry`|boolean|プロセスが起動され、デバッガーがアタッチされたらすぐに中断するかどうかを指定します。 このパラメーターの既定値は **`false`** です。|
|`remoteMachine`|string|プログラムが起動されたリモート マシンの名前を指定します。|
|`type`|string|プロジェクトが `dll` であるか `exe` であるかを指定します。既定値は .exe です。|

## <a name="c-linux-properties"></a>C++ Linux のプロパティ

|プロパティ|Type|説明|
|-|-|-|
|`program`|string|リモート マシン上のプログラム実行可能ファイルへの完全なパス。 CMake を使用する場合、このフィールドの値としてマクロ `${debugInfo.fullTargetPath}` を使用できます。|
|`processId`|整数|デバッガーをアタッチする省略可能なプロセス ID。|
|`sourceFileMap`|object|デバッグ エンジンに渡される省略可能なソース ファイル マッピング。 形式: `{ "\<Compiler source location>": "\<Editor source location>" }` または `{ "\<Compiler source location>": { "editorPath": "\<Editor source location>", "useForBreakpoints": true } }`。 例: `{ "/home/user/foo": "C:\\foo" }` または `{ "/home/user/foo": { "editorPath": "c:\\foo", "useForBreakpoints": true } }`。 「[ソース ファイル マップ オプション](#source_file_map_options)」を参照してください。|
|`additionalProperties`|string|sourceFileMapOptions の 1 つ。 (以下を参照してください。)|
|`MIMode`|string|MIDebugEngine の接続先となる MI 対応のコンソール デバッガーの種類を示します。 使用できる値は `"gdb"`、`"lldb"` です。|
|`args`|array|プログラムに渡すコマンド ライン引数。|
|`environment`|array|プログラムの環境に追加する環境変数。 例 : `[ { "name": "squid", "value": "clam" } ]`。|
|`targetArchitecture`|string|デバッグ対象のアーキテクチャ。 これは、このパラメーターが設定されていない限り、自動的に検出されます。 使用できる値は `x86`、`arm`、`arm64`、`mips`、`x64`、`amd64`、`x86_64` です。|
|`visualizerFile`|string|このプロセスをデバッグするときに使用する .natvis ファイル。 このオプションは GDB の再フォーマットと互換性がありません。 この設定を使用する場合は、「`"showDisplayString"`」を参照してください。|
|`showDisplayString`|boolean|visualizerFile を指定すると、`showDisplayString` により表示文字列が有効になります。 このオプションをオンにすると、デバッグ中にパフォーマンスが低下するおそれがあります。|
|`remoteMachineName`|string|gdb とデバッグ対象のプログラムをホストする、Linux リモート マシン。 新しい Linux マシンを追加するには、接続マネージャーを使用します。 CMake を使用する場合、このフィールドの値としてマクロ `${debugInfo.remoteMachineName}` を使用できます。|
|`miDebuggerPath`|string|MI 対応のデバッガー (gdb など) へのパス。 これを指定しないと、最初に PATH からデバッガーを検索します。|
|`miDebuggerServerAddress`|string|接続先の MI 対応デバッガー サーバーのネットワーク アドレス。 例: localhost:1234。|
|`setupCommands`|array|基になるデバッガーを設定するために実行する、1 つ以上の GDB または LLDB コマンド。 例 : `"setupCommands": [ { "text": "-enable-pretty-printing", "description": "Enable GDB pretty printing", "ignoreFailures": true }]`。 「[セットアップ コマンドを起動する](#launch_setup_commands)」を参照してください。|
|`customLaunchSetupCommands`|array|指定した場合、ターゲットを起動するために使われる既定のコマンドが、他のいくつかのコマンドに置き換えられます。 たとえば、ターゲット プロセスにアタッチする場合は、"-target-attach" になります。 空のコマンド リストを指定すると、起動コマンドは何とも置き換えられません。これは、デバッガーにコマンド ライン オプションとして起動オプションが提供されている場合に便利です。 例 : `"customLaunchSetupCommands": [ { "text": "target-run", "description": "run target", "ignoreFailures": false }]`。|
|`launchCompleteCommand`|string|デバッガーが完全にセットアップされた後に、ターゲット プロセスを実行するために実行するコマンド。 使用できる値は、"exec-run"、"exec-continue"、"None" です。 既定値は "exec-run" です。|
|`debugServerPath`|string|起動するデバッグ サーバーの完全なパス (省略可能)。 既定値は null です。|
|`debugServerArgs`|string|デバッグ サーバー引数 (省略可能)。 既定値は null です。|
|`filterStderr`|boolean|サーバー開始のパターンを stderr ストリームから検索し、stderr をデバッグ出力にログ記録します。 既定値は **`false`** です。|
|`coreDumpPath`|string|指定したプログラムのコア ダンプ ファイルへの完全なパス (省略可能)。 既定値は null です。|
externalConsole|boolean|true の場合、デバッグ対象のためにコンソールが起動します。 **`false`** の場合、コンソールは起動しません。 この設定の既定値は **`false`** です。 このオプションは、場合によっては技術的な理由で無視されます。|
|`pipeTransport`|string|これを指定した場合、デバッガーは、Visual Studio と MI 対応デバッガー (gdb など) の間で標準入力/出力をリレーするパイプとして別の実行可能ファイルを利用し、リモート コンピューターに接続します。 使用できる値: 1 つ以上の[パイプ トランスポート オプション](#pipe_transport_options)。|

## <a name="c-windows-remote-debug-and-deploy-properties"></a><a name="remote_deploy_debug"></a> C++ Windows リモート デバッグと配置のプロパティ

リモート マシンでアプリをデバッグおよび配置するときに使用します。

|プロパティ|Type|説明|
|-|-|-|
|`cwd`|string|リモート マシン上のターゲットの作業ディレクトリ。 CMake を使用する場合、このフィールドの値としてマクロ `${debugInfo.defaultWorkingDirectory}` を使用できます。 既定値は、デバッグ プログラムまたはコマンドのディレクトリです。|
|`deploy`|string|配置する追加のファイルまたはディレクトリを指定します。 次に例を示します。<br> `"deploy": {"sourcePath":"<Full path to source file/directory on host machine>", "targetPath":"<Full destination path to file/directory on target machine>"}` |
|`deployDirectory`|string|プロジェクト出力が自動的に配置されるリモート マシン上の場所。 既定値は "`C:\Windows Default Deploy Directory\<name of app>` です|
|`deployDebugRuntimeLibraries`|string|アクティブなプラットフォーム用のデバッグ ランタイム ライブラリを配置するかどうかを指定します。 アクティブな configurationType が `"Debug"` の場合、既定値は `"true"` です|
|`deployRuntimeLibraries`|string|アクティブなプラットフォーム用のランタイム ライブラリを配置するかどうかを指定します。 アクティブな configurationType が `"MinSizeRel"`、`"RelWithDebInfo"`、または `"Release"` の場合、既定値は `"true"` です。|
|`disableDeploy` | boolean | ファイルを配置するかどうかを指定します。 |
|`remoteMachineName`|string|プログラムが起動されたリモートの ARM64 Windows マシンの名前を指定します。 サーバー名またはリモート マシンの IP アドレスにすることができます。 |
|`authenticationType`|string|リモート接続の種類を指定します。 設定可能な値は `"windows"` および `"none"` です。 既定では、 `"windows"`です。 これは、リモート マシンで実行しているリモート デバッガーで指定されている認証設定と一致している必要があります。|

## <a name="launch-setup-commands"></a><a name="launch_setup_commands"></a> セットアップ コマンドを起動する

`setupCommands` プロパティと共に使用します。

|プロパティ|Type|説明|
|-|-|-|
|`text`|string|実行するデバッガー コマンド。|
|`description`|string|コマンドの説明 (省略可能)。|
|`ignoreFailures`|boolean|true に設定すると、コマンドからの失敗が無視されます。 既定値は **`false`** です。|

## <a name="pipe-transport-options"></a><a name = "pipe_transport_options"></a> パイプ トランスポート オプション

`pipeTransport` プロパティと共に使用します。

|プロパティ|Type|説明|
|-|-|-|
|`pipeCwd`|string|パイプ プログラムに渡す作業ディレクトリの完全修飾パス。|
|`pipeProgram`|string|実行するパイプ コマンドの完全修飾パス。|
|`pipeArgs`|array|接続を構成するためにパイプ プログラムに渡すコマンド ライン引数。|
|`debuggerPath`|string|ターゲット マシン上のデバッガーへの完全なパス。例: /usr/bin/gdb。|
|`pipeEnv`|object|パイプ プログラムに渡す環境変数。|
|`quoteArgs`|boolean|個々の引数に (スペースやタブなどの) 文字が含まれる場合に引用符で囲むかどうか。 **`false`** に設定すると、デバッガー コマンドを自動的に引用符で囲むことはなくなります。 既定値は **`true`** です。|

## <a name="source-file-map-options"></a><a name="source_file_map_options"></a> ソース ファイル マップ オプション

`sourceFileMap` プロパティと共に使用します。

|プロパティ|Type|説明|
|-|-|-|
|`editorPath`|string|エディターが配置するソース コードの場所。|
|`useForBreakpoints`|boolean|ブレークポイントを設定する場合は、このソース マッピングを使用する必要があります。 **`false`** に設定すると、ブレークポイントを設定するのにファイル名と行番号のみが使用されます。 **`true`** に設定すると、ブレークポイントは、このソース マッピングが使用されている場合にのみ、ファイルへの完全パスと行番号で設定されます。 それ以外の場合は、ファイル名と行番号のみがブレークポイントの設定に使用されます。 既定値は **`true`** です。|
