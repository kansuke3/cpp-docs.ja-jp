---
description: '詳細については、「チュートリアル: C++ AMP アプリケーションのデバッグ」を参照してください。'
title: 'チュートリアル: C++ AMP アプリケーションのデバッグ'
ms.date: 03/07/2021
helpviewer_keywords:
- debugging, C++ Accelerated Massive Parallelism
- C++ AMP, debugging
- C++ Accelerated Massive Parallelism, debugging
- debugging, C++ AMP
ms.openlocfilehash: 7073de40a3b985da89d4f0bb03046696f35c8f02
ms.sourcegitcommit: 6ed44d9c3fb32e965e363b9c69686739a90a2117
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/08/2021
ms.locfileid: "102465587"
---
# <a name="walkthrough-debugging-a-c-amp-application"></a>チュートリアル: C++ AMP アプリケーションのデバッグ

この記事では、C++ Accelerated Massive Parallelism (C++ AMP) を使用してグラフィックス処理装置 (GPU) を活用するアプリケーションをデバッグする方法について説明します。 これは、整数の大きな配列を合計する並列圧縮プログラムを使用します。 このチュートリアルでは、次の作業について説明します。

- GPU デバッガーを起動しています。

- GPU スレッドウィンドウで GPU スレッドを検査しています。

- [ **並列スタック** ] ウィンドウを使用して、複数の GPU スレッドの呼び出し履歴を同時に観察します。

- [ **並列ウォッチ** ] ウィンドウを使用して、複数のスレッド間で同時に1つの式の値を検査します。

- GPU スレッドのフラグの解除、凍結、凍結解除、およびグループ化を行います。

- タイルのすべてのスレッドをコード内の特定の位置に実行します。

## <a name="prerequisites"></a>前提条件

このチュートリアルを開始する前に:

- [「C++ AMP の概要」](../../parallel/amp/cpp-amp-overview.md)を参照してください。

- 行番号がテキストエディターに表示されていることを確認します。 詳細については、「 [方法: エディターで行番号を表示する](/visualstudio/ide/reference/how-to-display-line-numbers-in-the-editor)」を参照してください。

- ソフトウェアエミュレーターでのデバッグをサポートするために、少なくとも Windows 8 または Windows Server 2012 を実行していることを確認します。

[!INCLUDE[note_settings_general](../../mfc/includes/note_settings_general_md.md)]

### <a name="to-create-the-sample-project"></a>サンプル プロジェクトを作成するには

プロジェクトを作成する手順は、使用している Visual Studio のバージョンによって異なります。 このページの目次の上に、正しいドキュメントバージョンが選択されていることを確認します。

::: moniker range="msvc-160"

### <a name="to-create-the-sample-project-in-visual-studio-2019"></a>Visual Studio 2019 でサンプルプロジェクトを作成するには

1. メニュー バーで、 **[ファイル]** > **[新規作成]** > **[プロジェクト]** の順に選択して、 **[新しいプロジェクトの作成]** ダイアログ ボックスを開きます。

1. ダイアログの上部で、 **[言語]** を **[C++]** に、 **[プラットフォーム]** を **[Windows]** に、 **[プロジェクト タイプ]** を **[コンソール]** に設定します。

1. フィルター処理されたプロジェクト タイプの一覧から、 **[コンソール アプリ]** を選択して、 **[次へ]** を選択します。 次のページで、[ `AMPMapReduce` **名前** ] ボックスに「」と入力し、プロジェクトの名前を指定します。別のプロジェクトの場所を指定する場合は、プロジェクトの場所を指定します。

   ![プロジェクトに名前を指定する](../../build/media/mathclient-project-name-2019.png "プロジェクトに名前を指定する")

1. **[作成]** ボタンを選択してクライアント プロジェクトを作成します。

::: moniker-end

::: moniker range="<=msvc-150"

### <a name="to-create-the-sample-project-in-visual-studio-2017-or-visual-studio-2015"></a>Visual Studio 2017 または Visual Studio 2015 でサンプルプロジェクトを作成するには

1. Visual Studio を起動します。

1. メニュー バーで、 **[ファイル]** > **[新規作成]** > **[プロジェクト]** を選択します。

1. [テンプレート] ウィンドウの [ **インストール済み** ] で、[ **Visual C++**] を選択します。

1. [ **Win32 コンソールアプリケーション**] を選択し、[ `AMPMapReduce` **名前** ] ボックスに「 **」** と入力して、[OK] をクリックします。

1. **[次へ]** ボタンをクリックします。

1. [ **プリコンパイル済みヘッダー** ] チェックボックスをオフにし、[ **完了** ] をクリックします。

1. **ソリューションエクスプローラー** で、プロジェクトから *stdafx.h*、 *targetver .h*、および *stdafx.h* を削除します。

::: moniker-end

次:

1. AMPMapReduce を開き、内容を次のコードに置き換えます。

```cpp
    // AMPMapReduce.cpp defines the entry point for the program.
    // The program performs a parallel-sum reduction that computes the sum of an array of integers.

    #include <stdio.h>
    #include <tchar.h>
    #include <amp.h>

    const int BLOCK_DIM = 32;

    using namespace concurrency;

    void sum_kernel_tiled(tiled_index<BLOCK_DIM> t_idx, array<int, 1> &A, int stride_size) restrict(amp)
    {
        tile_static int localA[BLOCK_DIM];

        index<1> globalIdx = t_idx.global * stride_size;
        index<1> localIdx = t_idx.local;

        localA[localIdx[0]] =  A[globalIdx];

        t_idx.barrier.wait();

        // Aggregate all elements in one tile into the first element.
        for (int i = BLOCK_DIM / 2; i > 0; i /= 2)
        {
            if (localIdx[0] < i)
            {

                localA[localIdx[0]] += localA[localIdx[0] + i];
            }

            t_idx.barrier.wait();
        }

        if (localIdx[0] == 0)
        {
            A[globalIdx] = localA[0];
        }
    }

    int size_after_padding(int n)
    {
        // The extent might have to be slightly bigger than num_stride to
        // be evenly divisible by BLOCK_DIM. You can do this by padding with zeros.
        // The calculation to do this is BLOCK_DIM * ceil(n / BLOCK_DIM)
        return ((n - 1) / BLOCK_DIM + 1) * BLOCK_DIM;
    }

    int reduction_sum_gpu_kernel(array<int, 1> input)
    {
        int len = input.extent[0];

        //Tree-based reduction control that uses the CPU.
        for (int stride_size = 1; stride_size < len; stride_size *= BLOCK_DIM)
        {
            // Number of useful values in the array, given the current
            // stride size.
            int num_strides = len / stride_size;

            extent<1> e(size_after_padding(num_strides));

            // The sum kernel that uses the GPU.
            parallel_for_each(extent<1>(e).tile<BLOCK_DIM>(), [&input, stride_size] (tiled_index<BLOCK_DIM> idx) restrict(amp)
            {
                sum_kernel_tiled(idx, input, stride_size);
            });
        }

        array_view<int, 1> output = input.section(extent<1>(1));
        return output[0];
    }

    int cpu_sum(const std::vector<int> &arr) {
        int sum = 0;
        for (size_t i = 0; i < arr.size(); i++) {
            sum += arr[i];
        }
        return sum;
    }

    std::vector<int> rand_vector(unsigned int size) {
        srand(2011);

        std::vector<int> vec(size);
        for (size_t i = 0; i < size; i++) {
            vec[i] = rand();
        }
        return vec;
    }

    array<int, 1> vector_to_array(const std::vector<int> &vec) {
        array<int, 1> arr(vec.size());
        copy(vec.begin(), vec.end(), arr);
        return arr;
    }

    int _tmain(int argc, _TCHAR* argv[])
    {
        std::vector<int> vec = rand_vector(10000);
        array<int, 1> arr = vector_to_array(vec);

        int expected = cpu_sum(vec);
        int actual = reduction_sum_gpu_kernel(arr);

        bool passed = (expected == actual);
        if (!passed) {
            printf("Actual (GPU): %d, Expected (CPU): %d", actual, expected);
        }
        printf("sum: %s\n", passed ? "Passed!" : "Failed!");

        getchar();

        return 0;
    }
```

1. メニュー バーで、 **[ファイル]**  >  **[すべてを保存]** の順に選択します。

1. **ソリューションエクスプローラー** で、 **AMPMapReduce** のショートカットメニューを開き、[**プロパティ**] を選択します。

1. [**プロパティページ**] ダイアログボックスの [**構成プロパティ**] で、[ **C/c + +**  >  **プリコンパイル済みヘッダー**] を選択します。

1. **プリコンパイル済みヘッダー** プロパティの場合は、[**プリコンパイル済みヘッダーを使用しない**] を選択し、[ **OK** ] をクリックします。

1. メニュー バーで、 **[ビルド]**  >  **[ソリューションのビルド]** の順にクリックします。

## <a name="debugging-the-cpu-code"></a>CPU コードのデバッグ

この手順では、ローカルの Windows デバッガーを使用して、このアプリケーションの CPU コードが正しいことを確認します。 このアプリケーションで特に興味深い CPU コードのセグメントは、 **`for`** 関数内のループです `reduction_sum_gpu_kernel` 。 GPU で実行されるツリーベースの並列リダクションを制御します。

### <a name="to-debug-the-cpu-code"></a>CPU コードをデバッグするには

1. **ソリューションエクスプローラー** で、 **AMPMapReduce** のショートカットメニューを開き、[**プロパティ**] を選択します。

2. [ **プロパティページ** ] ダイアログボックスの [ **構成プロパティ**] で、[ **デバッグ**] を選択します。 [**起動するデバッガー** ] ボックスの一覧で [**ローカル Windows デバッガー** ] が選択されていることを確認します。

3. **コードエディター** に戻ります。

4. 次の図に示すコード行にブレークポイントを設定します (約67行 70)。

   ![エディターのコード行の横にマークされている CPU ブレークポイント。](../../parallel/amp/media/campcpubreakpoints.png "CPU ブレークポイント") <br/>
   CPU ブレークポイント

5. メニュー バーで、 **[デバッグ]**  >  **[デバッグ開始]** の順に選択します。

6. [ **ローカル** ] ウィンドウで、 `stride_size` 70 行目のブレークポイントに達するまでの値を確認します。

7. メニュー バーで、 **[デバッグ]**  >  **[デバッグの停止]** の順に選択します。

## <a name="debugging-the-gpu-code"></a>GPU コードのデバッグ

このセクションでは、GPU コードをデバッグする方法について説明します。これは、関数に含まれているコードです `sum_kernel_tiled` 。 GPU コードは、各 "ブロック" の整数の合計を並列で計算します。

### <a name="to-debug-the-gpu-code"></a>GPU コードをデバッグするには

1. **ソリューションエクスプローラー** で、 **AMPMapReduce** のショートカットメニューを開き、[**プロパティ**] を選択します。

2. [ **プロパティページ** ] ダイアログボックスの [ **構成プロパティ**] で、[ **デバッグ**] を選択します。

3. **[起動するデバッガー]** の一覧で、 **[ローカル Windows デバッガー]** を選択します。

4. [ **デバッガーの種類** ] ボックスの一覧で、[ **自動** ] が選択されていることを確認します。

    **Auto** は既定値です。 Windows 10 より前のバージョンでは、 **GPU** は **自動** ではなく必須の値です。

5. **[OK]** を選択します。

6. 次の図に示すように、30行目にブレークポイントを設定します。

   ![エディターのコード行の横にマークされている GPU ブレークポイント。](../../parallel/amp/media/campgpubreakpoints.png "GPU ブレークポイント") <br/>
   GPU ブレークポイント

7. メニュー バーで、 **[デバッグ]**  >  **[デバッグ開始]** の順に選択します。 Cpu コード内のコード行は CPU 上で実行されるため、67行と70行の CPU コードのブレークポイントは、GPU のデバッグ中には実行されません。

### <a name="to-use-the-gpu-threads-window"></a>[GPU スレッド] ウィンドウを使用するには

1. [ **Gpu スレッド**] ウィンドウを開くには、メニューバーで[  >  **Windows**  >  **GPU スレッド** のデバッグ] を選択します。

   表示される [ **Gpu スレッド** ] ウィンドウで gpu スレッドの状態を調べることができます。

2. [ **GPU スレッド** ] ウィンドウを Visual Studio の下部にドッキングします。 [ **スレッドの切り替え** ] をクリックすると、タイルとスレッドのテキストボックスが表示されます。 [ **Gpu スレッド** ] ウィンドウには、次の図に示すように、アクティブおよびブロックされている GPU スレッドの合計数が表示されます。

   ![4つのアクティブスレッドがある GPU スレッドウィンドウ。](../../parallel/amp/media/campc.png "4 つのアクティブ スレッドがある [GPU スレッド] ウィンドウ") <br/>
   [GPU スレッド] ウィンドウ

   313この計算には、タイルが割り当てられます。 各タイルには32のスレッドが含まれています。 ローカル GPU のデバッグはソフトウェアエミュレーターで行われるため、4つのアクティブな GPU スレッドがあります。 4つのスレッドが命令を同時に実行し、次の命令にまとめて移動します。

   [ **Gpu スレッド** ] ウィンドウには、4つの gpu スレッドがアクティブになっており、28個の gpu スレッドがブロックされています。これには、約21行目 () で定義された [tile_barrier:: wait](reference/tile-barrier-class.md#wait) ステートメントがあります `t_idx.barrier.wait();` 。 すべての 32 GPU スレッドは、最初のタイルであるに属 `tile[0]` します。 矢印は、現在のスレッドを含む行を指します。 別のスレッドに切り替えるには、次のいずれかの方法を使用します。

    - [ **GPU スレッド** ] ウィンドウで、切り替え先のスレッドの行のショートカットメニューを開き、[ **スレッドに切り替え**] を選択します。 行が複数のスレッドを表している場合は、スレッドの座標に従って最初のスレッドに切り替えます。

    - 対応するテキストボックスにスレッドのタイルとスレッドの値を入力し、[ **スレッドの切り替え** ] ボタンをクリックします。

   [ **呼び出し履歴** ] ウィンドウには、現在の GPU スレッドの呼び出し履歴が表示されます。

### <a name="to-use-the-parallel-stacks-window"></a>[並列スタック] ウィンドウを使用するには

1. [**並列スタック**] ウィンドウを開くには、メニューバーで[  >  **Windows**  >  **並列スタック** のデバッグ] を選択します。

   [ **並列スタック** ] ウィンドウを使用すると、複数の GPU スレッドのスタックフレームを同時に調べることができます。

2. [ **並列スタック** ] ウィンドウを Visual Studio の下部にドッキングします。

3. 左上隅の一覧で [ **スレッド** ] が選択されていることを確認します。 次の図の [ **並列スタック** ] ウィンドウには、[ **gpu スレッド** ] ウィンドウに表示された gpu スレッドの呼び出し履歴がフォーカスされたビューが表示されます。

   ![4つのアクティブスレッドがある並列スタックウィンドウ。](../../parallel/amp/media/campd.png "4 つのアクティブ スレッドがある [並列スタック] ウィンドウ") <br/>
   [並列スタック] ウィンドウ

   32スレッドは、から `_kernel_stub` 関数呼び出しのラムダステートメントに、その後、並列処理が減少するように `parallel_for_each` 関数に移動しました `sum_kernel_tiled` 。 32スレッドのうち28個がステートメントに進められ、 [`tile_barrier::wait`](reference/tile-barrier-class.md#wait) 22 行目でブロックされたままになります。一方、他の4つのスレッドは、30行目の関数でアクティブなままに `sum_kernel_tiled` なります。

   GPU スレッドのプロパティを調べることができます。 これらは、[**並列スタック**] ウィンドウの豊富な DataTip の [ **GPU スレッド**] ウィンドウで使用できます。 これらを表示するには、のスタックフレーム上にポインターを置き `sum_kernel_tiled` ます。 次の図は、DataTip を示しています。

   ![並列スタックウィンドウのための DataTip です。](../../parallel/amp/media/campe.png "[並列スタック] ウィンドウのデータヒント") <br/>
   GPU スレッドの DataTip

   [ **並列スタック** ] ウィンドウの詳細については、「 [[並列スタック] ウィンドウの使用](/visualstudio/debugger/using-the-parallel-stacks-window)」を参照してください。

### <a name="to-use-the-parallel-watch-window"></a>並列ウォッチウィンドウを使用するには

1. [**並列ウォッチ**] ウィンドウを開くには、メニューバーで [**デバッグ**] [  >  **Windows**  >  **並列ウォッチ**-  >  **並列ウォッチ 1**] の順に選択します。

   [ **並列ウォッチ** ] ウィンドウを使用すると、複数のスレッドにわたる式の値を調べることができます。

2. [ **並列ウォッチ 1** ] ウィンドウを Visual Studio の下部にドッキングします。 [ **並列ウォッチ** ] ウィンドウのテーブルには32行あります。 各は、[GPU スレッド] ウィンドウと [ **並列スタック** ] ウィンドウの両方に表示される gpu スレッドに対応します。 ここで、すべての 32 GPU スレッドで検査する値を持つ式を入力できます。

3. [ウォッチ列の **追加** ] ヘッダーを選択し、「」と入力して、 `localIdx` **enter** キーを押します。

4. [ウォッチ列の **追加** ] ヘッダーをもう一度選択し、「」と入力して、 `globalIdx` **enter** キーを押します。

5. [ウォッチ列の **追加** ] ヘッダーをもう一度選択し、「」と入力して、 `localA[localIdx[0]]` **enter** キーを押します。

   対応する列ヘッダーを選択すると、指定した式で並べ替えることができます。

   列を並べ替えるには、 **Locala [Locala [0]]** 列ヘッダーを選択します。 次の図は、 **Locala [Locala [0]]** による並べ替えの結果を示しています。

   ![並べ替えられた結果を使用した並列ウォッチウィンドウ。](../../parallel/amp/media/campf.png "結果が並べ替えられている [並列ウォッチ] ウィンドウ") <br/>
   並べ替えの結果

   [ **並列ウォッチ** ] ウィンドウのコンテンツを excel にエクスポートするには、 **excel のボタンを** クリックし、[ **excel で開く**] を選択します。 開発用コンピューターに Excel がインストールされている場合、このボタンをクリックすると、そのコンテンツを含む Excel ワークシートが開きます。

6. [ **並列ウォッチ** ] ウィンドウの右上隅にあるフィルターコントロールを使用して、ブール式を使用してコンテンツをフィルター処理できます。 `localA[localIdx[0]] > 20000`[フィルターコントロール] テキストボックスに「」と入力し、 **enter** キーを押します。

   このウィンドウに `localA[localIdx[0]]` は、値が2万より大きいスレッドだけが含まれるようになりました。 コンテンツは、前に選択した並べ替えアクションである列によって並べ替えられたままになり `localA[localIdx[0]]` ます。

## <a name="flagging-gpu-threads"></a>GPU スレッドのフラグを行う

特定の GPU スレッドをマークするには、[ **Gpu スレッド** ] ウィンドウ、[ **並列ウォッチ** ] ウィンドウ、または [ **並列スタック** ] ウィンドウの [DataTip] でフラグを設定します。 [GPU スレッド] ウィンドウの行に複数のスレッドが含まれている場合は、その行にフラグを追加すると、その行に含まれるすべてのスレッドがフラグになります。

### <a name="to-flag-gpu-threads"></a>GPU スレッドにフラグを付けるには

1. [**並列ウォッチ 1** ] ウィンドウで **[thread]** 列ヘッダーを選択して、タイルインデックスとスレッドインデックスで並べ替えます。

2. メニューバーで [**デバッグ** 続行] を選択すると、  >  アクティブだった4つのスレッドが次のバリアに進行します (AMPMapReduce の行32で定義)。

3. 現在アクティブになっている4つのスレッドを含む行の左側にあるフラグシンボルを選択します。

   次の図は、[ **GPU スレッド** ] ウィンドウのアクティブなフラグが設定されたスレッド4つを示しています。

   ![フラグが設定されたスレッドを含む GPU スレッドウィンドウ。](../../parallel/amp/media/campg.png "スレッドにフラグが設定されている [GPU スレッド] ウィンドウ") <br/>
   [GPU スレッド] ウィンドウのアクティブなスレッド

   [並列 **ウォッチ** ] ウィンドウと [ **並列スタック** ] ウィンドウの [DataTip] は、両方ともフラグが設定されたスレッドを示します。

4. フラグが設定された4つのスレッドにフォーカスを設定する場合は、フラグが設定されたスレッドのみを表示するように選択できます。 **GPU スレッド**、**並列ウォッチ**、および **並列スタック** のウィンドウに表示されるものを制限します。

   任意のウィンドウまたは [**デバッグの場所**] ツールバーの [**フラグ付きののみ表示**] ボタンをクリックします。 次の図は、[**デバッグの場所**] ツールバーの [**フラグ付きののみ表示**] ボタンを示しています。

   ![フラグ付きアイコンが表示されたデバッグ場所ツールバー。](../../parallel/amp/media/camph.png "[フラグが設定されたもののみを表示] アイコンがある [デバッグの場所] ツール バー") <br/>
   [**フラグ付きののみ表示**] ボタン

   これで、[ **GPU スレッド**]、[ **並列ウォッチ**]、[ **並列スタック** ] の各ウィンドウに、フラグが設定されたスレッドのみが表示されます。

## <a name="freezing-and-thawing-gpu-threads"></a>GPU スレッドの凍結と凍結解除

Gpu スレッドの凍結 (中断) と凍結解除 (再開) は、[ **Gpu スレッド** ] ウィンドウまたは [ **並列ウォッチ** ] ウィンドウから行うことができます。 CPU スレッドの凍結と凍結解除は、同じように行うことができます。詳細については、「 [方法: [スレッド] ウィンドウを使用する](/visualstudio/debugger/how-to-use-the-threads-window)」を参照してください。

### <a name="to-freeze-and-thaw-gpu-threads"></a>GPU スレッドを凍結および凍結解除するには

1. [ **フラグ付きののみ表示** ] ボタンをクリックして、すべてのスレッドを表示します。

2. メニューバーで、[**デバッグ**] [続行] の順に選択し  >  ます。

3. アクティブな行のショートカットメニューを開き、[ **フリーズ**] を選択します。

   [ **GPU スレッド** ] ウィンドウの次の図は、4つのすべてのスレッドが固定されていることを示しています。

   ![凍結されたスレッドを示す GPU スレッドウィンドウ。](../../parallel/amp/media/campk.png "凍結されたスレッドを示す [GPU スレッド] ウィンドウ") <br/>
   [ **GPU スレッド** ] ウィンドウの凍結されたスレッド

   同様に、[ **並列ウォッチ** ] ウィンドウには、4つのスレッドすべてが固定されていることが示されます。

4. メニューバーで [**デバッグ**] [続行] を選択  >  すると、次の4つの GPU スレッドが22行目のバリアを超えて進行し、30行目のブレークポイントに移動できるようになります。 [ **GPU スレッド** ] ウィンドウには、以前に凍結された4つのスレッドが固定され、アクティブな状態のままであることが示されます。

5. メニューバーで、[ **デバッグ**]、[ **続行**] の順に選択します。

6. [ **並列ウォッチ** ] ウィンドウでは、個別または複数の GPU スレッドを凍結解除することもできます。

### <a name="to-group-gpu-threads"></a>GPU スレッドをグループ化するには

1. [ **GPU スレッド** ] ウィンドウのスレッドの1つのショートカットメニューで、[ **グループ化**]、[ **アドレス**] の順に選択します。

   [ **GPU スレッド** ] ウィンドウのスレッドは、アドレスによってグループ化されます。 アドレスは、スレッドの各グループが配置されている逆アセンブリの命令に対応します。 24のスレッドは22行にあり、 [tile_barrier:: Wait メソッド](reference/tile-barrier-class.md#wait) が実行されます。 12のスレッドは、32行目のバリアの命令にあります。 これらのスレッドの4つにはフラグが設定されています。 8つのスレッドは、30行目のブレークポイントにあります。 これらのスレッドの4つは固定されています。 次の図は、[ **GPU スレッド** ] ウィンドウのグループ化されたスレッドを示しています。

   ![スレッドがアドレス別にグループ化された GPU スレッドウィンドウ。](../../parallel/amp/media/campl.png "スレッドがアドレスごとにグループ化されている [GPU スレッド] ウィンドウ") <br/>
   [ **GPU スレッド** ] ウィンドウのグループ化されたスレッド

2. [**並列ウォッチ**] ウィンドウのデータグリッドのショートカットメニューを開いて、 **Group by** 操作を行うこともできます。 [ **グループ化**] を選択し、スレッドをグループ化する方法に対応するメニュー項目を選択します。

## <a name="running-all-threads-to-a-specific-location-in-code"></a>コード内の特定の場所へのすべてのスレッドの実行

特定のタイル内のすべてのスレッドを、カーソルを含む行に実行するには、[ **現在のタイルをカーソルまで実行**] を使用します。

### <a name="to-run-all-threads-to-the-location-marked-by-the-cursor"></a>カーソルによってマークされた場所にすべてのスレッドを実行するには

1. 凍結されたスレッドのショートカットメニューで、[ **凍結** 解除] を選択します。

2. **コードエディター** で、カーソルを30行目に置きます。

3. **コードエディター** のショートカットメニューで、[**現在のタイルをカーソル行の前まで実行**] を選択します。

   以前に21行目のバリアでブロックされていた24個のスレッドが32行目に進んでいます。 [ **GPU スレッド** ] ウィンドウに表示されます。

## <a name="see-also"></a>関連項目

[C++ AMP の概要](../../parallel/amp/cpp-amp-overview.md)\
[GPU コードのデバッグ](/visualstudio/debugger/debugging-gpu-code)\
[方法: GPU スレッドウィンドウを使用する](/visualstudio/debugger/how-to-use-the-gpu-threads-window)\
[方法: 並列ウォッチウィンドウを使用する](/visualstudio/debugger/how-to-use-the-parallel-watch-window)\
[同時実行ビジュアライザーを使用した C++ AMP コードの分析](/archive/blogs/nativeconcurrency/analyzing-c-amp-code-with-the-concurrency-visualizer)
