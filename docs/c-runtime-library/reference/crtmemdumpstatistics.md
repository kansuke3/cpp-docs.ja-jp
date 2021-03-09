---
description: '詳細については、次を参照してください: _CrtMemDumpStatistics'
title: _CrtMemDumpStatistics
ms.date: 11/04/2016
api_name:
- _CrtMemDumpStatistics
api_location:
- msvcrt.dll
- msvcr80.dll
- msvcr90.dll
- msvcr100.dll
- msvcr100_clr0400.dll
- msvcr110.dll
- msvcr110_clr0400.dll
- msvcr120.dll
- msvcr120_clr0400.dll
- ucrtbase.dll
api_type:
- DLLExport
topic_type:
- apiref
f1_keywords:
- CrtMemDumpStatistics
- _CrtMemDumpStatistics
helpviewer_keywords:
- _CrtMemDumpStatistics function
- CrtMemDumpStatistics function
ms.assetid: 27b9d731-3184-4a2d-b9a7-6566ab28a9fe
ms.openlocfilehash: 684b5ac50af453dbe2800ded595c5747370717a6
ms.sourcegitcommit: 90c300b74f6556cb5d989802d2e80d79542f55e7
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/09/2021
ms.locfileid: "102514591"
---
# <a name="_crtmemdumpstatistics"></a>_CrtMemDumpStatistics

指定されたヒープ状態のデバッグ ヘッダー情報をユーザーが判読できる形式でダンプします (デバッグ バージョンのみ)。

## <a name="syntax"></a>構文

```C
void _CrtMemDumpStatistics(
   const _CrtMemState *state
);
```

### <a name="parameters"></a>パラメーター

*状態*<br/>
ダンプするヒープ状態へのポインター。

## <a name="remarks"></a>注釈

**_CrtMemDumpStatistics** 関数は、指定されたヒープ状態のデバッグヘッダー情報をユーザーが判読できる形式でダンプします。 ダンプ統計情報は、割り当ての追跡とメモリの問題の検出のためにアプリケーションで使用できます。 メモリ状態には、特定のヒープ状態、または 2 つの状態の相違点を含めることができます。 [_DEBUG](../../c-runtime-library/debug.md)が定義されていない場合、 **_CrtMemDumpStatistics** の呼び出しはプリプロセス中に削除されます。

*State* パラメーターは、 **_CrtMemDumpStatistics** が呼び出される前に [_CrtMemCheckpoint](crtmemcheckpoint.md)によって入力された、または [_CrtMemDifference](crtmemdifference.md)によって返された **_CrtMemState** 構造体へのポインターである必要があります。 *State* が **NULL** の場合、「[パラメーターの検証](../../c-runtime-library/parameter-validation.md)」で説明されているように、無効なパラメーターハンドラーが呼び出されます。 実行の継続が許可された場合、 **errno** は **EINVAL** に設定され、アクションは実行されません。 詳細については、「[errno、_doserrno、_sys_errlist、および _sys_nerr](../../c-runtime-library/errno-doserrno-sys-errlist-and-sys-nerr.md)」を参照してください。

ヒープ状態関数と **_CrtMemState** 構造体の詳細については、「 [ヒープ状態レポート関数](/visualstudio/debugger/crt-debug-heap-details)」を参照してください。 デバッグ バージョンのベース ヒープでのメモリ ブロックの割り当て、初期化、管理の方法について詳しくは、「 [CRT Debug Heap Details](/visualstudio/debugger/crt-debug-heap-details)」をご覧ください。

## <a name="requirements"></a>必要条件

|ルーチンによって返される値|必須ヘッダー|省略可能なヘッダー|
|-------------|---------------------|----------------------|
|**_CrtMemDumpStatistics**|\<crtdbg.h>|\<errno.h>|

互換性について詳しくは、「 [Compatibility](../../c-runtime-library/compatibility.md)」をご覧ください。

**ライブラリ:**[C ランタイムライブラリ](../../c-runtime-library/crt-library-features.md)のデバッグバージョンのみ。

## <a name="see-also"></a>こちらもご覧ください

[デバッグルーチン](../../c-runtime-library/debug-routines.md)<br/>
