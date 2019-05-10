## Cythonの調査（Numpyで実際的な画像を想定した離散畳み込み処理で比較）

### 目的
画像解析の色味の変化の検出などで頻繁に行われる　畳み込み処理　を想定して、Pure Python と、Cython（厳密な型指定を施す）のパフォーマンスを比較する。  
画像とフィルタは、Numpy上に構築したデータを利用する。  
<br>

### Pure Python
``` Jpyter Notebook
import numpy as np
```

Pureな関数。  
``` Jpyter Notebook
def pure_filter(f, g):
    vupper = img.shape[0]
    wupper = img.shape[1]
    supper = filter.shape[0]
    tupper = filter.shape[1]
    smidium = supper
    tmidium = tupper
    xupper = vupper + 2*smidium
    yupper = wupper + 2*tmidium
    # 出力画像を想定した配列
    h = np.zeros([xupper, yupper], dtype=f.dtype)
    # 畳み込み処理
    for x in range(xupper):
        for y in range(yupper):
            # filter の各ピクセルに対する加算
            s_from = max(smidum - x, -smidum)
            s_to = min((xupper - x) - smidum, smidum + 1)
            t_from = max(tmidum - y, -tmidum)
            t_to = min((ymidum - y) - tmidum, tmidum + 1)
            value = 0
            for s in range(s_from, s_to):
                for t in range(t_from, t_to):
                    v = x - smidum + s
                    w = y - tmidum + t
                    value += filter[smidum - s, tmidum - t] * img[v, w]
            h[x, y] = value
    return h
```

実行
``` Jpyter Notebook
N = 100
img = np.arange(N*N, dtype=np.int).reshape((N,N))
filter = np.arange(81, dtype=np.int).reshape((9, 9))
%timeit -n2 -r3 pure_filer(img, filter)
```

結果は ...
``` Jpyter Notebook
414 ms ± 1.57 ms per loop (mean ± std. dev. of 3 runs, 2 loops each)
```
<br>

### 厳密な型指定を施した Cython
``` Jpyter Notebook
%%cython
import numpy as np
cimport numpy as np
DTYPE = np.int
ctypedef np.int_t DTYPE_t

def cython_filter(np.ndarray img, np.ndarray filter):
    cdef int vupper = img.shape[0]
    cdef int wupper = img.shape[1]
    cdef int supper = filter.shape[0]
    cdef int tupper = filter.shape[1]
    cdef int smidium = supper // 2
    cdef int tmidium = tupper // 2
    cdef int xupper = vupper + 2*smidium
    cdef int yupper = wupper + 2*tmidium
    cdef np.ndarray h = np.zeros([xupper, yupper], dtype=DTYPE)
    cdef int x, y, s, t, v, w
    cdef int s_from, s_to, t_from, t_to
    cdef DTYPE_t value
    for x in range(xupper):
        for y in range(yupper):
            s_from = max(smidium - x, -smidium)
            s_to = min((xupper - x) - smidium, smidium + 1)
            t_from = max(tmidium - y, -tmidium)
            t_to = min((yupper - y) - tmidium, tmidium + 1)
            value = 0
            for s in range(s_from, s_to):
                for t in range(t_from, t_to):
                    v = x - smidium + s
                    w = y - tmidium + t
                    value += filter[smidium - s, tmidium - t] * img[v, w]
            h[x, y] = value
    return h
```

結果は ...
``` Jpyter Notebook
N = 100
img = np.arange(N*N, dtype=np.int).reshape((N,N))
filter = np.arange(81, dtype=np.int).reshape((9, 9))
%timeit -n2 -r3 cython_filter(img, filter)

372 ms ± 301 µs per loop (mean ± std. dev. of 3 runs, 2 loops each)
```
<br>

### 所見
比較結果  
|条件  |内容  |
|:---|:---|
|pure Python  |414 ms ± 1.57 ms per loop (mean ± std. dev. of 3 runs, 2 loops each)  |
|Cython すべての型に型指定  |372 ms ± 301 µs per loop (mean ± std. dev. of 3 runs, 2 loops each)  |  

この比較ではあまり効果があがっていない。  
インデックスの取り方とか工夫が必要かもしれない。今後の課題。  

更新履歴：  
2019.05.10 作成
