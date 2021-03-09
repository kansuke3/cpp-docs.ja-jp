---
description: '詳細については、次を参照してください: _fcvt_s'
title: _fcvt_s
ms.date: 4/2/2020
api_name:
- _fcvt_s
- _o__fcvt_s
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
- api-ms-win-crt-convert-l1-1-0.dll
- api-ms-win-crt-private-l1-1-0.dll
api_type:
- DLLExport
topic_type:
- apiref
f1_keywords:
- fcvt_s
- _fcvt_s
helpviewer_keywords:
- fcvt_s function
- converting floating point, to strings
- floating-point functions, converting number to string
- _fcvt_s function
ms.assetid: 48671197-1d29-4c2b-a5d8-d2368f5f68a1
ms.openlocfilehash: 62b72a9f71f967077169086371ebf36542b154ed
ms.sourcegitcommit: 90c300b74f6556cb5d989802d2e80d79542f55e7
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/09/2021
ms.locfileid: "102514448"
---
# <a name="_fcvt_s"></a>_fcvt_s

浮動小数点数を文字列に変換します。 これは、「[CRT のセキュリティ機能](../../c-runtime-library/security-features-in-the-crt.md)」の説明にあるとおり、セキュリティが強化されたバージョンの [_fcvt](fcvt.md) です。

## <a name="syntax"></a>構文

```C
errno_t _fcvt_s(
   char* buffer,
   size_t sizeInBytes,
   double value,
   int count,
   int *dec,
   int *sign
);
template <size_t size>
errno_t _fcvt_s(
   char (&buffer)[size],
   double value,
   int count,
   int *dec,
   int *sign
); // C++ only
```

### <a name="parameters"></a>パラメーター

*格納*<br/>
変換の結果を保持する指定されたバッファー。

*sizeInBytes*<br/>
バッファーのサイズ (バイト単位)。

*value*<br/>
変換される数値。

*count*<br/>
小数点以下の桁数。

*alpha*<br/>
格納された小数点位置を指すポインター。

*sign*<br/>
格納された符号インジケーターを指すポインター。

## <a name="return-value"></a>戻り値

正常終了した場合は 0 を返します。 障害が発生した場合、戻り値はエラー コードを示します。 エラー コードは、Errno.h で定義されています。 これらのエラーの一覧については、「[errno、_doserrno、_sys_errlist、および _sys_nerr](../../c-runtime-library/errno-doserrno-sys-errlist-and-sys-nerr.md)」を参照してください。

パラメーターが次の表の無効な値の場合は、「[パラメーターの検証](../../c-runtime-library/parameter-validation.md)」で説明されているように、この関数は無効なパラメーター ハンドラーを呼び出します。 実行の継続が許可された場合、この関数は **errno** を **einval** に設定し、 **einval** を返します。

### <a name="error-conditions"></a>エラー条件

|*格納*|*sizeInBytes*|value|count|dec|sign|戻り値|*バッファー* 内の値|
|--------------|-------------------|-----------|-----------|---------|----------|------------|-----------------------|
|**NULL**|any|any|any|any|any|**EINVAL**|変更されません。|
|Not **NULL** (有効なメモリを指す)|<=0|any|any|any|any|**EINVAL**|変更されません。|
|any|any|any|any|**NULL**|any|**EINVAL**|変更されません。|
|any|any|any|any|any|**NULL**|**EINVAL**|変更されません。|

## <a name="security-issues"></a>セキュリティの問題

*バッファー* が有効なメモリを指しておらず、が **NULL** でない場合、 **_fcvt_s** はアクセス違反を生成する可能性があります。

## <a name="remarks"></a>注釈

**_Fcvt_s** 関数は、浮動小数点数を null で終わる文字列に変換します。 *値* パラメーターは、変換される浮動小数点数です。 **_fcvt_s** は、 *値* の数字を文字列として格納し、null 文字 (' \ 0 ') を追加します。 *Count* パラメーターは、小数点の後に格納する桁数を指定します。 余分な数字は、 *カウント* するために丸められます。 有効桁数が *カウント* よりも小さい場合は、文字列に0が埋め込まれます。

文字列には数字だけが格納されます。 小数点の位置と *値* の符号は *dec* から取得でき、呼び出しの後に *符号* を付けます。 *Dec* パラメーターは整数値を指します。この整数値は、文字列の先頭に対する小数点の位置を示します。 0 または負の整数値は、小数点が文字列の先頭より左にあることを示します。 パラメーター *sign* は、 *値* の符号を示す整数を指します。 Value が正で、*値* が負の場合は0以外の値に設定されて *いる場合、* 整数は0に設定されます。

長さ **_CVTBUFSIZE** のバッファーは、任意の浮動小数点値に対して十分です。

**_Ecvt_s** と **_fcvt_s** の違いは、 *count* パラメーターの解釈です。 **_ecvt_s** は、 *カウント* を出力文字列の合計桁数として解釈し、 **_fcvt_s** は小数点の後の桁数 *として解釈し* ます。

C++ では、テンプレートのオーバーロードによってこの関数を簡単に使用できます。オーバーロードでは、バッファー長を自動的に推論できるため、サイズ引数を指定する必要がなくなります。 詳細については、「[セキュリティ保護されたテンプレート オーバーロード](../../c-runtime-library/secure-template-overloads.md)」を参照してください。

この関数のデバッグバージョンは、最初にバッファーを0xFE で埋めます。 この動作を無効にするには、[_CrtSetDebugFillThreshold](crtsetdebugfillthreshold.md) を使用します。

既定では、この関数のグローバル状態はアプリケーションにスコープが設定されています。 これを変更するには、「 [CRT でのグローバル状態](../global-state.md)」を参照してください。

## <a name="requirements"></a>必要条件

|機能|必須ヘッダー|オプション ヘッダー|
|--------------|---------------------|---------------------|
|**_fcvt_s**|\<stdlib.h>|\<errno.h>|

互換性について詳しくは、「 [Compatibility](../../c-runtime-library/compatibility.md)」をご覧ください。

**ライブラリ:**[C ランタイムライブラリ](../../c-runtime-library/crt-library-features.md)のすべてのバージョン。

## <a name="example"></a>例

```C
// fcvt_s.c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

int main()
{
    char * buf = 0;
    int decimal;
    int sign;
    int err;

    buf = (char*) malloc(_CVTBUFSIZE);
    err = _fcvt_s(buf, _CVTBUFSIZE, 1.2, 5, &decimal, &sign);

    if (err != 0)
    {
        printf("_fcvt_s failed with error code %d\n", err);
        exit(1);
    }

    printf("Converted value: %s\n", buf);
}
```

```Output
Converted value: 120000
```

## <a name="see-also"></a>関連項目

[データ変換](../../c-runtime-library/data-conversion.md)<br/>
[浮動小数点のサポート](../../c-runtime-library/floating-point-support.md)<br/>
[atof、_atof_l、_wtof、_wtof_l](atof-atof-l-wtof-wtof-l.md)<br/>
[_ecvt_s](ecvt-s.md)<br/>
[_gcvt_s](gcvt-s.md)<br/>
[_fcvt](fcvt.md)<br/>
