---
description: '詳細情報: `/X` (標準インクルードパスの無視)'
title: /X (標準インクルード パスの無視)
ms.date: 01/21/2021
f1_keywords:
- /x
- VC.Project.VCCLCompilerTool.OVERWRITEStandardIncludePath
- VC.Project.VCCLWCECompilerTool.OVERWRITEStandardIncludePath
- VC.Project.VCCLCompilerTool.IgnoreStandardIncludePath
helpviewer_keywords:
- /X compiler option [C++]
- include files, ignore standard path
- -X compiler option [C++]
- include directories, ignore standard
- X compiler option
- Ignore Standard Include Paths compiler option
ms.assetid: 16bdf2cc-c8dc-46e4-bdcc-f3caeba5e1ef
ms.openlocfilehash: 97d19027c41df7db9103c1c21d2f2d7b8d398e7e
ms.sourcegitcommit: a26a66a3cf479e0e827d549a9b850fad99b108d1
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/22/2021
ms.locfileid: "98712805"
---
# <a name="x-ignore-standard-include-paths"></a>`/X` (標準インクルードパスを無視します)

および環境変数で指定されたディレクトリ内のインクルードファイルをコンパイラが検索できないようにし `PATH` `INCLUDE` ます。

## <a name="syntax"></a>構文

> **`/X`**

## <a name="remarks"></a>解説

このオプションを ([追加の[ `/I` インクルードディレクトリ])](i-additional-include-directories.md)オプションと共に使用すると、標準のインクルードファイルへの代替パスを指定できます。

### <a name="to-set-this-compiler-option-in-the-visual-studio-development-environment"></a>Visual Studio 開発環境でこのコンパイラ オプションを設定するには

1. プロジェクトの **[プロパティ ページ]** ダイアログ ボックスを開きます。 詳細については、[Visual Studio での C++ コンパイラとビルド プロパティの設定](../working-with-project-properties.md)に関するページを参照してください。

1. [**構成プロパティ**] [  >  **C/c + +**] [  >  **プリプロセッサ**] プロパティページを選択します。

1. " **標準インクルードパスを無視** する" プロパティを変更します。

### <a name="to-set-this-compiler-option-programmatically"></a>このコンパイラ オプションをコードから設定するには

- 以下を参照してください。<xref:Microsoft.VisualStudio.VCProjectEngine.VCCLCompilerTool.IgnoreStandardIncludePath%2A>

## <a name="example"></a>例

次のコマンドで、は、 **`/X`** および環境変数で指定された場所を無視 `PATH` し、 `INCLUDE` **`/I`** インクルードファイルを検索するディレクトリを指定します。

```cmd
CL /X /I \ALT\INCLUDE MAIN.C
```

## <a name="see-also"></a>関連項目

[MSVC コンパイラ オプション](compiler-options.md)<br/>
[MSVC コンパイラ コマンド ラインの構文](compiler-command-line-syntax.md)
