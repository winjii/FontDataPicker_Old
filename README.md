フォント内部のデータを読み、縦書き用グリフ置換を行う処理を実装したもの。
GsubReaderクラス参照。

# FreeTypeをどういじったか(VS2017使用)
GsubReaderクラスを正しく動作させるためのFreeTypeのビルド方法。
VS2017を使用したがほかの環境でも同じように出来るはず？

- プロジェクトファイル"builds/windows/vc2010/freetype.vcxproj"をVSで開く


## FT_OpenType_Validate()を使えるように、該当モジュールを追加する

FreeTypeにはオプションで使えるモジュールなるものが用意されている。
FT_OpenType_Validate()はフォントのGSUBテーブルを参照するために使う。


- ソリューションエクスプローラーから、Source Files/FT_MODULES/ にファイル"src/otvalid/otvalid.c"を追加


- include\freetype\config\ftmodule.h を開き ```FT_USE_MODULE( FT_Module_Class, otv_module_class )``` を追加


## FreeTypeのバグを修正

単独置換タイプでフォーマットが1のLookupテーブルは、置換後のグリフIDを (置換前のグリフID)+(差分) として指定するが、この際暗黙的に2^16=65536でmodを取る仕様になっている。しかしFreeType(2.8.1時点)は、この計算を行う時にそれを考慮しておらず、計算結果がはみ出した途端に無効なデータとして処理してしまいGSUBテーブルを参照させてくれない。
これを修正する。

- src\otvalid\otvgsub.c を開き、74行目あたりをコメントアウト
```
    //if ( idx < 0 )
    //  FT_INVALID_DATA;
    
    //idx = (FT_Long)otv_Coverage_get_last( Coverage ) + DeltaGlyphID;
    //if ( (FT_UInt)idx >= otvalid->glyph_count )
    //  FT_INVALID_DATA;
```

## ビルド
後はビルドをした

# 参考情報
- Microsoft Typography
  - https://www.microsoft.com/en-us/Typography/OpenTypeSpecification.aspx
  - フォント内部の他の情報が欲しい時も、概要を調べた後、これを見ながら頑張ってパースしていく感じになると思います

