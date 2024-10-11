# study-go


 
 <!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
 
 <!-- code_chunk_output -->
 
- [study-go](#study-go)
  - [Install for osx](#install-for-osx)
    - [パッケージを作る](#パッケージを作る)
  - [実行アプリを書いてビルドする](#実行アプリを書いてビルドする)
  - [for statement](#for-statement)
  - [Goの基本文法](#goの基本文法)
  - [Goの関数](#goの関数)
  - [可変長引数](#可変長引数)
  - [値渡しと参照渡し (Pythonは参照渡し。がintとかstring(immutable)はコピーされる)](#値渡しと参照渡し-pythonは参照渡しがintとかstringimmutableはコピーされる)
  - [値、型としての関数(interface)](#値型としての関数interface)
  - [PanicとRecover](#panicとrecover)
  - [main関数とinit関数](#main関数とinit関数)
  - [struct](#struct)
  - [structの匿名フィールド　(Goに継承ってないんじゃなかったっけ？)](#structの匿名フィールドgoに継承ってないんじゃなかったっけ)
  - [method継承](#method継承)
  - [interfaceとは何か](#interfaceとは何か)
    - [空のinterface](#空のinterface)
    - [interface関数の引数](#interface関数の引数)
  - [Goで簡単なWebサーバを立てる](#goで簡単なwebサーバを立てる)
  - [そのほか](#そのほか)
    - [Linux/macOSなど](#linuxmacosなど)
    - [Windows](#windows)
    - [バイナリサイズ小さくする](#バイナリサイズ小さくする)
 
 <!-- /code_chunk_output -->
 
 ## Install for osx
 ```sh
 brew install go
 export GOPATH=$HOME/go
 #Go 1.11以降では、go.modファイルを使用するモジュールモードが標準になり、GOPATHは必須ではなくなりました。しかし、ツールや依存関係のためにGOPATH/binは依然として有用です。
 go env #で確認できる
 
 mkdir $GOPATH/src
 # src にはソースコードを保存します（例えば：.go .c .h .s等）これから開発するPGにとってメインとなるディレクトリです。全てのソースコードはこのディレクトリに置く
 # pkg にはコンパイル後に生成されるファイル（例えば：.a）コードが依存する可能性のあるサードパーティの Go 依存関係など、さまざまなパッケージオブジェクトが格納されています。
 # bin にはコンパイル後に生成される実行可能ファイル
 
 $GOPATH
 ├─bin/
 ├─pkg/
 │  └─darwin_amd64/
 │    ├─github.com/
 │    │  └─GitHubユーザ名
 │    │    ├─`*.a`ファイル[^1]
 │    │    └─GitHubレポジトリ名/`*.a`ファイル[^2]
 │    └─pkg.in/
 │      ├─パッケージ名/
 │      └─`*.a`ファイル
 └─src/
   ├─gopkg.in/
   │  └─パッケージ名/
   │    └─LICENSEとか`*.go`とかREADMEとか
   └─github.com/
     ├─GitHubユーザ名
     │  └─GitHubレポジトリ名/
     │    └─LICENSEとか`*.go`とかREADMEとか
     └─<あなたのGitHubユーザ名>
       └─GitHubレポジトリ名/
         ├─glide.yaml
         ├─main.go
         ├─その他、あなたが開発中のソフトウェアのコード
         └─vendor/依存先パッケージのコード(glideでとってきたやつ)
 ```
 
 ### パッケージを作る
 
 ```sh
 cd $GOPATH/src
 mkdir mymodule
 vim mymodule/sqrt.go
 ```
 ```go
 // $GOPATH/src/mymodule/sqrt.goコードは以下の通り：
 // 一般的にpackageの名前とディレクトリ名は一致させるべきです。
 package mymodule
 
 func Sqrt(x float64) float64 {
 	z := 0.0
 	for i := 0; i < 1000; i++ {
 		z -= (z*z - x) / (2 * x)
 	}
 	return z
 }
 ```
 以下でコンパイル/installできる。(とりあえずmina.goをbuildする時リンクされるのでやらなくてもOK)  
 * 対応するアプリケーションパッケージディレクトリに入り、go installを実行すればインストールできます。  
 * 任意のディレクトリで以下のコードgo install mymoduleを実行。
 ```sh
 go install {mymodule}
 # インストールが終われば、以下のディレクトリにmymodule.aのファイルが現れるはずです。
 cd $GOPATH/pkg/${GOOS}_${GOARCH}
 mymodule.a
 
 ```
 ## 実行アプリを書いてビルドする
 
 .aファイルはアプリケーションパッケージです。どのように実行できるでしょうか？
 次にアプリケーション・プログラムを作成してこのアプリケーションパッケージをコールします。
 ```sh
 cd $GOPATH/src
 mkdir myapp
 cd myapp
 vim main.go
 ```
 ```go
 package main
 // package <pkgName>（我々の例ではpackage main）の1行は現在のファイルがどのパッケージの属しているかを表しています
 // またパッケージmainはこれ自体が独立して実行できるパッケージであることを示しています。コンパイル後実行可能ファイルが生成されます。mainパッケージを除いて、他のパッケージは最後には*.aというファイルが生成され（パッケージファイルとも呼ばれます。）
 // それぞれ独立して実行できるGoプログラムは必ずpackage mainの中に含まれています。このmainパッケージには必ずインターフェースとなるmain関数が含まれます。この関数には引数がなく、戻り値もありません。
 import (
     "mymodule"
 	  "fmt"
 )
 // パッケージ名とパッケージが存在するディレクトリは異なっていてもかまいません。ここでは<pkgName>がディレクトリ名ではなくpackage <pkgName>で宣言されるパッケージ名とします。
 
 func main() {
 	  fmt.Printf("Hello, world.  Sqrt(2) = %v\n", mymodule.Sqrt(2))
 }
 // importにおいてコールするパッケージはmymoduleであり、これが$GOPATH/srcのパスに対応します。もしネストしたディレクトリであれば、importの中でネストしたディレクトリをインポートします。例えばいくつものGOPATHがあった場合も同じで、Goは自動的に複数の$GOPATH/srcの中から探し出します。
 ```
 `go run`でコンパイルと実行もやってくれる。ファイルを実行しましたが、スタンドアロンのバイナリファイルは生成しません  
 `go build`を実行すれば、このディレクトリの下にmyappの実行可能ファイルが生成されます。
 結果をインストールせずに、パッケージとその依存関係を含めてアプリケーションをコンパイルします。ディスク上にバイナリファイルを作成しますが、プログラムは実行されません。-o出力コマンドラインオプションを使用して作成されたバイナリファイルの名前を変更可能。
 ```sh
 go build
 ./myapp
 # Hello, world.  Sqrt(2) = 1.414213562373095
 # デフォルトでは、生成されるバイナリファイルにはデバッグ情報とシンボルテーブルが含まれています
 # バイナリのサイズを約 30% 削減することができます。
 go build -ldflags "-w -s"
 #linuxバイナリでクロスコンパイル
 GOOS="linux" GOARCH="amd64" go build main.go
 ```
 どのようにアプリケーションをインストールするのでしょうか。このディレクトリに入り、go installを実行すると、$GOPATH/bin/の下に実行可能ファイルmyappが作成されます。$GOPATH/binが我々のPATHに追加されていることを思い出して下さい、コマンドラインから以下のように入力することで実行することができます。
 ```sh
 go install
 myapp
 # Hello, world.  Sqrt(2) = 1.414213562373095
 ```
 まとめると
 * binディレクトリの下にコンパイル後の実行可能ファイルが保存され、  
 * pkgの下に関数パッケージが保存され、
 * srcの下にアプリケーションのソースコードが保存されているということです。
 
 ## for statement
 ```go
 aa := [3]string{"taro", "jiro", "hanako"}
 // aa := [3]int{5,8,100}
 for i, v := range aa{
     fmt.Println(i, v)
 }
 
 ```
 
 
 ## Goの基本文法
 文法を通してGoがいかに簡単かご覧いただけたかと思います。たった２５個のキーワードです。
 ```go
 break    default      func    interface    select
 case     defer        go      map          struct
 chan     else         goto    package      switch
 const    fallthrough  if      range        type
 continue for          import  return       var
 ```
 これらキーワードが何に使われるのか見てみることにしましょう。
 * varとconstは2.2のGo言語の基礎に出てくる変数と定数の宣言を参考にしてください。
 * packageとimportにはすでに少し触れました。
 * func は関数とメソッドの定義に用いられます。
 * return は関数から返るために用いられます。
 * defer はデストラクタのようなものです。
 * go はマルチスレッドに用いられます。
 * select は異なる型の通信を選択するために用いられます。
 * interface はインターフェースを定義するために用いられます。2.6章をご参考ください。
 * struct は抽象データ型の定義に用いられます。2.5章をご参考ください。
 * break、case、continue、for、fallthrough、else、if、switch、goto、defaultは2.3のフロー紹介をご参考ください。
 * chanはchannel通信に用いられます。
 * typeはカスタム定義型の宣言に用いられます。
 * mapはmap型のデータの宣言に用いられます。
 * rangeはslice、map、channelデータの読み込みに用いられます。
 この２５個のキーワードを覚えれば、Goは既に殆ど学び終わったも同然です。
 
 
 ## Goの関数
 関数はGoの中心的な設計です。キーワードfuncによって宣言します。形式は以下の通り：
 ```go
 func funcName(i1 type1, i2 type2) (out1 type1, out2 type2) {
 	//ここはロジック処理のコードです。
 	//複数の値を戻り値とします。
 	return value1, value2
 }
 ```
 * 関数はひとつまたは複数の引数をとることができ、各引数の後には型が続きます。,をデリミタとします。
 * 関数は複数の戻り値を持ってかまいません。
 * 上の戻り値は２つの変数out1とout2であると宣言されています。もし宣言したくないというのであればそれでもかみません。直接２つの型です。
 * もしひとつの戻り値しか存在せず、また戻り値の変数が宣言されていなかった場合、戻り値の括弧を省略することができます。
 * もし戻り値が無ければ、最後の戻り値の情報も省略することができます。
 * もし戻り値があれば、関数の中でreturn文を追加する必要があります。
 
 ```go
 func (f *File) Write(b []byte) (n int, err error) {
     if f == nil {
         return 0, ErrInvalid
     }
     n, e := f.write(b)
     :
 }
 // このように名前の前に型を付けて定義されている関数は、その型に対する メソッド です。つまりこの定義からは、 Write が File という型のデータに対するメソッドであることがわかります。
 // その File は、通常のファイルを表すような型で、os パッケージで定義されている 構造体 です。Go言語における構造体は、複数のデータ型を集めたデータ型で、いわば「データの塊」だといえます。
 // つまりこの Write は、f が指し示すファイルに対してバイト列 b を書き込み、書き込んだバイト数 n と、エラーが起きた場合はそのエラー error を返す、といえます。
 
 // ex.2
 func max(a, b int) int {
 	if a > b {
 		return a
 	}
 	return b
 }
 // 第一引数の型は省略することができます（つまり、a int, b intと同意）、デフォルトは直近の型です。２つ以上の同じ型の変数または戻り値も同じです。同時に戻り値がひとつであることに注意してください。これは省略記法です。
 
 // Go言語はCに比べ先進的な特徴を持っています。関数が複数の戻り値を持てるのもその一つです。
 //A+B と A*B を返します
 func SumAndProduct(A, B int) (int, int) {
 	return A+B, A*B
 }
 
 // 上の例では直接２つの引数を返しました。当然引数を返す変数に命名してもかまいません。この例では２つの型のみ使っていますが、下のように定義することもできます。値が返る際は変数名を付けなくてかまいません。なぜなら関数の中で直接初期化されているからです。しかしもしあなたの関数がエクスポートされるのであれば（大文字からはじまります）オフィシャルではなるべく戻り値に名前をつけるようお勧めしています。なぜなら名前のわからない戻り値はコードをより簡潔なものにしますが、生成されるドキュメントの可読性がひどくなるからです。
 
 func SumAndProduct(A, B int) (add int, Multiplied int) {
 	add = A+B
 	Multiplied = A*B
 	return
 }
 ```
 
 ## 可変長引数
 Goの関数は可変長引数をサポートしています。可変長引数を受け付ける関数は不特定多数の引数があります。これを実現するために、関数が可変長引数を受け取れるよう定義する必要があります：
 ```go
 func myfunc(arg ...int) {}
 // arg ...intはGoにこの関数が不特定多数の引数を受け付けることを伝えます。ご注意ください。この引数の型はすべてintです。関数ブロックの中で変数argはintのsliceとなります。
 for _, n := range arg {
 	fmt.Printf("And the number is: %d\n", n)
 }
 ```
 
 ## 値渡しと参照渡し (Pythonは参照渡し。がintとかstring(immutable)はコピーされる)
 引数をコールされる関数の中に渡すとき、実際にはこの値のコピーが渡されます。コールされる関数の中で引数に修正をくわえても、関数をコールした実引き数には何の変化もありません。数値の変化はコピーの上で行われるだけだからです。
 
 この内容を検証するために、ひとつ例を見てみましょう
 ```go
 package main
 import "fmt"
 
 //引数+1を行う、簡単な関数
 func add1(a int) int {
 	a = a+1 // aの値を変更します。
 	return a //新しい値を返します。
 }
 
 func main() {
 	x := 3
 
 	fmt.Println("x = ", x)  // "x = 3"と出力するはずです。
 
 	x1 := add1(x)  //add1(x) をコールします。
 
 	fmt.Println("x+1 = ", x1) // "x+1 = 4"　と出力するはずです。
 	fmt.Println("x = ", x)    // "x = 3" と出力するはずです。
 }
 ```
 add1関数をコールし、add1のなかでa = a+1の操作を実行したとしても、上述のx変数には何の変化も発生しません。
 
 理由はとても簡単です：add1がコールされた際、add1が受け取る引数はxそのものではなく、xのコピーだからです。
 
 もし本当にこのxそのものを渡したくなったらどうするの？と疑問に思うかもしれません。
 
 この場合いわゆるポインタにまで話がつながります。我々は変数がメモリの中のある特定の位置に存在していることを知っています。変数を修正するということはとどのつまり変数のアドレスにあるメモリを修正していることになります。add1関数がx変数のアドレスを知ってさえいれば、x変数の値を変更することが可能です。そのため、我々はxの存在するアドレスである`&x`を関数に渡し、関数の変数の型をintからポインタ変数である*intに変更します。これで関数の中でxの値を変更することができるようになりました。この時関数は依然としてコピーにより引数を受け渡しますが、コピーしているのはポインタになります。以下の例をご覧ください。
 ```go
 package main
 import "fmt"
 
 //引数に+1を行う簡単な関数
 func add1(a *int) int { // ご注意ください。
 	*a = *a+1 // aの値を修正しています。
 	return *a // 新しい値を返します。
 }
 
 func main() {
 	x := 3
 
 	fmt.Println("x = ", x)  // "x = 3"と出力するはずです。
 
 	x1 := add1(&x)  // add1(&x) をコールしてxのアドレスを渡します。
 
 	fmt.Println("x+1 = ", x1) // "x+1 = 4"を出力するはずです。
 	fmt.Println("x = ", x)    // "x = 4"を出力するはずです。
 }
 ```
 このようにxを修正するという目的に到達しました。では、ポインタを渡す長所はなんなのでしょうか？
 
 * ポインタを渡すことで複数の関数が同じオブジェクトに対して操作を行うことができます。
 * インスタンスはメモリ上に確保されているデータの本体です。100バイトの文字列であれば100バイトぶんのメモリを消費しています。一方、ポインタはインスタンスの場所情報です。64ビット機であれば8バイトです。
 * ポインタ渡しは比較的軽いです（8バイト）、ただのメモリのアドレスです。ポインタを使って大きな構造体を渡すことができます。もし値渡しを行なっていたら、相対的にもっと多くのシステムリソース（メモリと時間）を毎回のコピーで消費することになります。そのため大きな構造体を渡す際は、ポインタを使うのが賢い選択というものです。
 * インスタンスがメモリ上にあれば、そのメモリのアドレスはかならず1つあるので、インスタンスからポインタを作ることができます。また、ポインタは特定のアドレスを指しているので、ポインタからインスタンスを取り出すこともできます。相互に変換できる、というのは大切な特性です。
 * &は、インスタンスからポインタを取り出す操作です。これはインスタンスをメモリ上に作った後にポインタを取り出して変数に入れています。
 * Go言語のchannel、slice、mapの３つの型はメカニズムを実現するポインタのようなものです。ですので、直接渡すことができますので、アドレスを取得してポインタを渡す必要はありません。（注：もし関数がsliceの長さを変更する場合はアドレスを取得し、ポインタを渡す必要があります。）
 * もしポインタの引数を渡さなければ、関数が受け取るのは実は引数で与えられた変数のコピーになってしまいます。つまり、メソッド内で色の変更を行うと、元の変数のコピーを操作しているだけで、本当の変数ではないのです。そのため、値を変更したい場合は、ポインタを渡す必要があります。
 * Goではコールしているポインタのメソッドがポインタのメソッドであるかどうかは気にする必要がありません。Goはあなたが行おうとしているすべてのことを知っているのです。C/C++でプログラムを経験されてこられた方にとっては、とてもとても大きな苦痛が解決されることでしょう。
 
 
 ## 値、型としての関数(interface)
 Goでは関数も変数の一種です。typeを通して定義します。これは全て同じ引数と同じ戻り値を持つ一つの型です。
 `type typeName func(input1 inputType1 , input2 inputType2 [, ...]) (result1 resultType1 [, ...])`
 関数を型として扱うことにメリットはあるのでしょうか？ではこの型の関数を値として渡してみましょう。以下の例をご覧ください。
 ```go
 package main
 import "fmt"
 
 type testInt func(int) bool // 関数の型を宣言します。
 
 func isOdd(integer int) bool {
 	if integer%2 == 0 {
 		return false
 	}
 	return true
 }
 
 func isEven(integer int) bool {
 	if integer%2 == 0 {
 		return true
 	}
 	return false
 }
 
 // ここでは宣言する関数の型を引数のひとつとみなします。
 func filter(slice []int, f testInt) []int {
 	var result []int
 	for _, value := range slice {
 		if f(value) {
 			result = append(result, value)
 		}
 	}
 	return result
 }
 
 func main(){
 	slice := []int {1, 2, 3, 4, 5, 7}
 	fmt.Println("slice = ", slice)
 	odd := filter(slice, isOdd)    // 関数の値渡し
 	fmt.Println("Odd elements of slice are: ", odd)
 	even := filter(slice, isEven)  // 関数の値渡し
 	fmt.Println("Even elements of slice are: ", even)
 }
 ```
 共有のインターフェースを書くときに関数を値と型にみなすのは非常に便利です。上の例でtestIntという型は関数の型の一つでした。
 ふたつのfilter関数の引数と戻り値はtestIntの型と同じですが、より多くのロジックを実現することができ、プログラムをより優れたものにすることができます。  
 
 構造体が「データの塊」であるのに対し、インタフェースは「メソッド宣言の塊」であるような型だといえるでしょう。そして、上記のような仕様の Write メソッドが宣言されているインタフェースが io.Writer なのです。実際に io.Writer インタフェースの定義を見てみましょう。インタフェースは、構造体と違って何かしら実体を持つものを表すのではなく、「どんなことができるか」を宣言しているだけです。
 ```go
 
 type Writer interface {
     Write(p []byte) (n int, err error)
 }
 // os/file.go では、まさにこの形式で、 Write が *File のメソッドとして定義されていましたね。os/file.go における Write() の定義を再掲します。
 func (f *File) Write(b []byte) (n int, err error) {
     if f == nil {
         return 0, ErrInvalid
     }
     n, e := f.write(b)
     :
 }
 // インタフェースで宣言されているすべてのメソッドが、データ型に対して定義されている場合、そのデータ型は「インタフェースを満たす」と表現します。いま見たように、 io.Writer は Write() メソッドの仕様が宣言されているインタフェースです。そして *File というデータ型には、その仕様通りに定義された Write() メソッドがあります。したがって、「*File は io.Writer インタフェースを満たす」といえます 
 ```
 
 ## PanicとRecover
 https://github.com/astaxie/build-web-application-with-golang/blob/master/ja/02.3.md#panic%E3%81%A8recover
 
 ## main関数とinit関数
 Goでは２つの関数が予約されています：init関数（すべてのpackageで使用できます）とmain関数（package mainでしか使用できません）です。
 この２つの関数はいかなる引数と戻り値も持ちません。packageの中では各ファイルに一つだけのinit関数を書くよう強くおすすめします。
 packageのなかで複数のinit関数を書けるが、可読性か後々のメンテナンス性がよくない。
 
 Goのプログラムは自動でinit()とmain()をコールしますので、どこかでこの２つの関数をコールする必要はありません。
 各packageのinit関数はオプションです。しかしpackage mainは必ず一つmain関数を含まなければなりません。
 
 プログラムの初期化と実行はすべてmainパッケージから始まります。もしmainパッケージが他のパッケージをインポートしていたら、コンパイル時にその依存パッケージがインポートされます。あるパッケージが複数のパッケージに同時にインポートされている場合は、先にその他のパッケージがインポートされ、その後このパッケージの中にあるパッケージクラス定数と変数が初期化されます。
 次にinit関数が（もしあれば）実行され、最後にmain関数が実行されます。
 
 ## struct
 Go言語では、Cや他の言語と同じように、他の型の属性やフィールドのコンテナとして新しい型を宣言することができます。例えば、一個人のエンティティを表しているperson型を作成することができます。このエンティティは属性を持っています：性別と年齢です。このような型はstructと呼ばれます。以下にコードを示します：
 ```go
 type person struct {
 	name string
 	age int
 }
 ```
 どのようにstructは使用されるのでしょうか？下のコードをご覧ください。
 ```go
 type person struct {
 	name string
 	age int
 }
 
 var P person  // Pは現在person型の変数です。
 
 P.name = "Astaxie"  // "Astaxie"を変数Pのnameプロパティに代入します。
 P.age = 25  // "25"を変数Pのageプロパティに代入します。
 fmt.Printf("The person's name is %s", P.name)  // Pのnameプロパティにアクセスします。
 
 // 上のようなPの宣言以外に他にもいくつかの宣言方法があります。
 // 1.順序にしたがって初期化する。
 P := person{"Tom", 25}
 
 // 2.field:valueの方法によって初期化します。この場合は順序は任意でかまいません。
 P := person{age:24, name:"Tom"}
 
 // 3.もちろんnew関数を通してポインタを作ることもできます。このPの型は*personです。
 P := new(person)
 
 
 // 以下ではひと通りのstructの使用例をご説明します。
 package main
 import "fmt"
 
 // 新しい型を宣言します。
 type person struct {
 	name string
 	age int
 }
 
 // 二人の年齢を比較します。年齢が大きい方の人を返し、また年齢差も返します。
 // structも値渡しです。
 func Older(p1, p2 person) (person, int) {
 	if p1.age>p2.age {  // p1とp2の二人の年齢を比較します。
 		return p1, p1.age-p2.age
 	}
 	return p2, p2.age-p1.age
 }
 
 func main() {
 	var tom person
 
 	// 初期値を代入します。
 	tom.name, tom.age = "Tom", 18
 
 	// ２つのフィールドを明確に初期化します。
 	bob := person{age:25, name:"Bob"}
 
 	// structの定義の順番に従って初期化します。
 	paul := person{"Paul", 43}
 
 	tb_Older, tb_diff := Older(tom, bob)
 	tp_Older, tp_diff := Older(tom, paul)
 	bp_Older, bp_diff := Older(bob, paul)
 
 	fmt.Printf("Of %s and %s, %s is older by %d years\n",
 		tom.name, bob.name, tb_Older.name, tb_diff)
 
 	fmt.Printf("Of %s and %s, %s is older by %d years\n",
 		tom.name, paul.name, tp_Older.name, tp_diff)
 
 	fmt.Printf("Of %s and %s, %s is older by %d years\n",
 		bob.name, paul.name, bp_Older.name, bp_diff)
 }
 ```
 ## [structの匿名フィールド　(Goに継承ってないんじゃなかったっけ？)]( https://github.com/astaxie/build-web-application-with-golang/blob/master/ja/02.4.md#struct%E3%81%AE%E5%8C%BF%E5%90%8D%E3%83%95%E3%82%A3%E3%83%BC%E3%83%AB%E3%83%89)
 
 上でstructをどのように定義するか紹介しました。定義する際はフィールド名とその型が一つ一つ対応しています。実はGoは型だけの定義もサポートしています。これはフィールド名を書かない方法ではなく、匿名フィールドです。組み込みフィールドとも呼ばれます。
 
 匿名フィールドがstructである場合、このstructがもつすべてのフィールドは隠されたまま現在定義しているこのstructに導入されます。
 
 ひとつ具体的な例をお見せしましょう。
 ```go
 package main
 import "fmt"
 
 type Human struct {
 	name string
 	age int
 	weight int
 }
 
 type Student struct {
 	Human  // 匿名フィールド、デフォルトでStudentはHumanのすべてのフィールドを含むことになります。
 	speciality string
 }
 
 func main() {
 	// 学生を一人初期化します。
 	mark := Student{Human{"Mark", 25, 120}, "Computer Science"}
 
 	// 対応するフィールドにアクセスします。
 	fmt.Println("His name is ", mark.name)
 	fmt.Println("His age is ", mark.age)
 	fmt.Println("His weight is ", mark.weight)
 	fmt.Println("His speciality is ", mark.speciality)
 	// 対応するメモ情報を修正します。
 	mark.speciality = "AI"
 	fmt.Println("Mark changed his speciality")
 	fmt.Println("His speciality is ", mark.speciality)
 	// 彼の年齢情報を修正します。
 	fmt.Println("Mark become old")
 	mark.age = 46
 	fmt.Println("His age is", mark.age)
 	// 体重情報も修正します。
 	fmt.Println("Mark is not an athlet anymore")
 	mark.weight += 60
 	fmt.Println("His weight is", mark.weight)
 }
 ```
 
 
  ![Qiita](https://github.com/astaxie/build-web-application-with-golang/raw/master/ja/images/2.4.student_struct.png?raw=true)
 *点線のところが匿名*
 
 Studentがageとnameの属性にアクセスする際、あたかも自分のフィールドであるかのようにアクセスしたのをご覧いただけるかと思います。匿名フィールドというのはこういうものです。フィールドの継承を実現できるのです。もっとクールにする方法もありますよ。studentはHumanのフィールド名でアクセスできます。下のコードを御覧ください。ほら、とってもクールでしょ？
 ```go
 mark.Human = Human{"Marcus", 55, 220}
 mark.Human.age -= 1
 ```
 匿名によるアクセスとフィールドの修正はとても便利です。でも単なるstructのフィールドですから、すべてのビルトイン型と自分で定義した型をすべて匿名フィールドとみなすことができます。下の例をご覧ください。
 ```go
 package main
 import "fmt"
 
 type Skills []string
 
 type Human struct {
 	name string
 	age int
 	weight int
 }
 
 type Student struct {
 	Human  // 匿名フィールド、struct
 	Skills // 匿名フィールド、自分で定義した型。string slice
 	int    // ビルトイン型を匿名フィールドとします。
 	speciality string
 }
 
 func main() {
 	// 学生Jannを初期化します。
 	jane := Student{Human:Human{"Jane", 35, 100}, speciality:"Biology"}
 	// ここで対応するフィールドにアクセスしてみます。
 	fmt.Println("Her name is ", jane.name)
 	fmt.Println("Her age is ", jane.age)
 	fmt.Println("Her weight is ", jane.weight)
 	fmt.Println("Her speciality is ", jane.speciality)
 	// 彼のskill技能フィールドを修正します。
 	jane.Skills = []string{"anatomy"}
 	fmt.Println("Her skills are ", jane.Skills)
 	fmt.Println("She acquired two new ones ")
 	jane.Skills = append(jane.Skills, "physics", "golang")
 	fmt.Println("Her skills now are ", jane.Skills)
 	// 匿名ビルトイン型のフィールドを修正します。
 	jane.int = 3
 	fmt.Println("Her preferred number is", jane.int)
 }
 ```
 上の例のとおり、structはstructを匿名フィールドとするだけでなく、自分で定義した型やビルトイン型も匿名フィールドとすることができます。また、対応するフィールド上で関数操作を行うこともできます（例えば例の中のappendです）。
 
 ここで一つ疑問がでてきます：もしhumanにphoneというフィールドがあったとすると、studentもphoneと呼ばれるフィールドができます。これはどうすべきでしょうか？
 
 Goでは簡単にこの問題を解決することができます。外側が優先的にアクセスされますので、student.phoneとアクセスした場合studentの中のフィールドにアクセスし、humanのフィールドにはアクセスしません。
 
 このように匿名フィールドを通じてフィールドを継承することができます。当然もしあなたが対応する匿名型のフィールドにアクセスしたくなったら、匿名フィールドの名前からアクセスすることができます。下の例をご覧ください。
 ```go
 package main
 import "fmt"
 
 type Human struct {
 	name string
 	age int
 	phone string  // Human型がもつフィールド
 }
 
 type Employee struct {
 	Human  // 匿名フィールドHuman
 	speciality string
 	phone string  // 社員のphoneフィールド
 }
 
 func main() {
 	Bob := Employee{Human{"Bob", 34, "777-444-XXXX"}, "Designer", "333-222"}
 	fmt.Println("Bob's work phone is:", Bob.phone)
 	// もし我々がHumanのphoneフィールドにアクセスする場合は
 	fmt.Println("Bob's personal phone is:", Bob.Human.phone)
 }
 ```
 カスタム定義型って？カスタム定義型はstructじゃないのか。そういうわけではありません。structはカスタム定義型のなかでも比較的特殊な型であるだけです。下のような宣言で実現します。
 ```go
 type typeName typeLiteral
 // 以下のカスタム定義型の宣言のコードをご覧ください。
 
 type ages int
 
 type money float32
 
 type months map[string]int
 
 m := months {
 	"January":31,
 	"February":28,
 	...
 	"December":31,
 }
 ```
 このように自分のコードの中に意味のある型を定義することができるのです。実際はエイリアスを定義しているだけです。Cのtypedefに似たようなもので、例えば上のagesはintの代わりになっています。
 
 ## method継承
 フィールドだけでなくmethodも継承できるのです。もし匿名フィールドが一つのメソッドを実現している場合、この匿名フィールドを含むsturctもこのメソッドをコールすることができるのです。例をお見せします。
 ```go
 package main
 import "fmt"
 
 type Human struct {
 	name string
 	age int
 	phone string
 }
 
 type Student struct {
 	Human //匿名フィールド
 	school string
 }
 
 type Employee struct {
 	Human //匿名フィールド
 	company string
 }
 
 //Humanでmethodを定義
 func (h *Human) SayHi() {
 	fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
 }
 
 //EmployeeのmethodでHumanのmethodを書き直す。
 func (e *Employee) SayHi() {
 	fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,
 		e.company, e.phone) //Yes you can split into 2 lines here.
 }
 
 func main() {
 	mark := Student{Human{"Mark", 25, "222-222-YYYY"}, "MIT"}
 	sam := Employee{Human{"Sam", 45, "111-888-XXXX"}, "Golang Inc"}
 
 	mark.SayHi()
 	sam.SayHi()
 }
 ```
 上のコードのデザインはこのように絶妙です。Goのデザインに驚くことでしょう。
 このように、基本的なオブジェクト指向のプログラムを設計することができます。ですが、Goのオブジェクト指向はこのように簡単です。プライベートやパブリックといったキーワードは出てきません。大文字と小文字によって実現しているのです（大文字で始まるものはパブリック、小文字で始まるものはプライベートです）、メソッドにも同じルールが適用されます。
 
 
 ## interfaceとは何か
 Goではとても繊細なinterfaceと呼ぶべき設計があります。これはオブジェクト指向と内容構成にとって非常に便利。
 簡単にいえば、interfaceはmethodの組み合わせです。interfaceを通してオブジェクトの振る舞いを定義することができます。
 struct、StudentとEmployeeで他のメソッドSingを実現します。その後StudentはBorrowMoneyメソッドを追加してEmployeeはSpendSalaryを追加しましょう。
 * Studentには３つのメソッドがあることになります：SayHi、Sing、BorrowMoneyです。EmployeeはSayHi、Sing、SpendSalaryです。このようなメソッドの組み合わせはinterfaceと呼ばれます。そして、それらはStudentとEmployeeで実装されます。StudentとEmployeeはinterfaceのSayHiとSingを実装します。同時にEmployeeはBorrowMoneyを実装しません。そして、StudentはSpendSalaryを実装しません。なぜなら、EmployeeはBorrowMoneyメソッドを持っていません。また、StudentはSpendSalaryメソッドを持っていないからです。
 * もし我々がinterfaceの変数を定義すると、この変数にはこのinterfaceの任意の型のオブジェクトを保存することができます。
 
 以下のコードで、interfaceはメソッドの集合を抽象化したものだとお分かりいただけるとおもいます。他のinterfaceでない型によって実装されなければならず、自分自身では実装することができません。Goはinterfaceを通してduck-typingを実現できます。すなわち、"鳥の走る様子も泳ぐ様子も鳴く声もカモのようであれば、この鳥をカモであると呼ぶことができる"わけです。
 ```go
 package main
 import "fmt"
 
 type Human struct {
 	name string
 	age int
 	phone string
 }
 
 type Student struct {
 	Human //匿名フィールド
 	school string
 	loan float32
 }
 
 type Employee struct {
 	Human //匿名フィールド
 	company string
 	money float32
 }
 
 //HumanにSayHiメソッドを実装します。
 func (h Human) SayHi() {
 	fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
 }
 
 //HumanにSingメソッドを実装します。
 func (h Human) Sing(lyrics string) {
 	fmt.Println("La la la la...", lyrics)
 }
 
 //EmployeeはHumanのSayHiメソッドをオーバーロードします。
 func (e Employee) SayHi() {
 	fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,
 		e.company, e.phone)
 	}
 
 // Interface MenはHuman,StudentおよびEmployeeに実装されます。
 // この３つの型はこの２つのメソッドを実装するからです。
 type Men interface {
 	SayHi()
 	Sing(lyrics string)
 }
 
 func main() {
 	mike := Student{Human{"Mike", 25, "222-222-XXX"}, "MIT", 0.00}
 	paul := Student{Human{"Paul", 26, "111-222-XXX"}, "Harvard", 100}
 	sam := Employee{Human{"Sam", 36, "444-222-XXX"}, "Golang Inc.", 1000}
 	tom := Employee{Human{"Tom", 37, "222-444-XXX"}, "Things Ltd.", 5000}
 
 	//Men型の変数iを定義します。
 	var i Men
 
 	//iにはStudentを保存できます。
 	i = mike
 	fmt.Println("This is Mike, a Student:")
 	i.SayHi()
 	i.Sing("November rain")
 
 	//iにはEmployeeを保存することもできます。
 	i = tom
 	fmt.Println("This is Tom, an Employee:")
 	i.SayHi()
 	i.Sing("Born to be wild")
 
 	//sliceのMenを定義します。
 	fmt.Println("Let's use a slice of Men and see what happens")
 	x := make([]Men, 3)
 	//この３つはどれも異なる要素ですが、同じインターフェースを実装しています。
 	x[0], x[1], x[2] = paul, sam, mike
 
 	for _, value := range x{
 		value.SayHi()
 	}
 }
 ```
 ### 空のinterface
 空のinterface(interface{})にはなんのメソッドも含まれていません。この通り、すべての型は空のinterfaceを実装しています。空のinterfaceはそれ自体はなんの意味もありません（何のメソッドも含まれていませんから）が、任意の型の数値を保存する際にはかなり役にたちます。これはあらゆる型の数値を保存することができるため、C言語のvoid*型に似ています。
 ```go
 // aを空のインターフェースとして定義
 var a interface{}
 var i int = 5
 s := "Hello world"
 // aは任意の型の数値を保存できます。
 a = i
 a = s
 ```
 ある関数がinterface{}を引数にとると、任意の型の値を引数にとることができます。もし関数がinterface{}を返せば、任意の型の値を返すことができるのです。とても便利ですね！
 
 
 ### interface関数の引数
 interfaceの変数はこのinterface型のオブジェクトを持つことができます。これにより、関数（メソッドを含む）を書く場合思いもよらない思考を与えてくれます。interface引数を定義することで、関数にあらゆる型の引数を受けさせることができるです。
 
 例をあげましょう：`fmt.Println`は私達がもっともよく使う関数です。ですが、任意の型のデータを受けることができる点に気づきましたか。fmtのソースファイルを開くとこのような定義が書かれています：
 ```go
 type Stringer interface {
 	 String() string
 }
 ```
 つまり、Stringメソッドを持つ全ての型が`fmt.Println`によってコールされることができるのです。ためしてみましょう。
 ```go
 package main
 import (
 	"fmt"
 	"strconv"
 )
 
 type Human struct {
 	name string
 	age int
 	phone string
 }
 
 // このメソッドを使ってHumanにfmt.Stringerを実装します。
 func (h Human) String() string {
 	return "❰"+h.name+" - "+strconv.Itoa(h.age)+" years -  ✆ " +h.phone+"❱"
 }
 
 func main() {
 	Bob := Human{"Bob", 39, "000-7777-XXX"}
 	fmt.Println("This Human is : ", Bob)
 }
 ```
 さらに詳しくは、、 [https://github.com/astaxie/build-web-application-with-golang/blob/master/ja/02.6.md]
 
 ## Goで簡単なWebサーバを立てる
 https://github.com/astaxie/build-web-application-with-golang/blob/master/ja/03.2.md#32-go%E3%81%A7%E7%B0%A1%E5%8D%98%E3%81%AAweb%E3%82%B5%E3%83%BC%E3%83%90%E3%82%92%E7%AB%8B%E3%81%A6%E3%82%8B  
 でWebサーバを書くためにはhttpパッケージの２つの関数を呼ぶだけで良いことがわかります。
 
 >もしあなたが以前PHPプログラマであれば。こう問うかもしれません。我々のnginx、apacheサーバは必要ないのですかと？なぜならこいつは直接tcpポートを監視しますので、nginxがやることをやってくれます。またsayhelloNameは実は我々が書いたロジック関数ですので、phpの中のコントローラ（controller）関数に近いものです。
 >もしあなたがPythonプログラマであったのなら、tornadoを聞いたことがあると思います。このコードはそれとよく似ていませんか？ええ、その通りです。GoはPythonのような動的な言語によく似た特性を持っています。Webアプリケーションを書くにはとても便利です。
 >もしあなたがRubyプログラマであったのなら、RORの/script/serverを起動したのと少し似ている事に気づいたかもしれません。
 
 ## そのほか
 [Goのコマンド https://github.com/astaxie/build-web-application-with-golang/blob/master/ja/01.3.md]
 
 `go clean` でお掃除。
 `go doc` でパッケージ、関数、メソッド、変数に関するドキュメントを調べることができます
 ```sh
 go doc fmt.Println
 ```
 
 このツールは、golang.orgのパッケージのドキュメントをローカルでも見られるようにしてくれるツールです。ウェブサーバとして起動してブラウザで見るのが典型的な使い方ですが、起動時に次のように --analysis type を付けることでインタフェースの分析を行ってくれます。
 
 `godoc -http ":6060" -analysis type`
 
 GOPATH以下にパッケージがたくさん入っている場合は、解析にものすごい時間がかかってしまいます。GOPATHを一時的に変更して実行するなどしてください。
 
 ### Linux/macOSなど
 $ GOPATH=/ godoc -http ":6060" -analysis type
  
 ### Windows
 > set GOPATH=C:\
 > godoc -http ":6060" -analysis type
 　ブラウザで http://localhost:6060 を開くと、次のような golang.org にそっくりのページが表示されます。このようにgodocはオフラインでドキュメントを見ることができて便利
 
 パッケージのソースコードを取得するには、`go get` コマンドを使用します
 ```sh
 go get github.com/astaxie/beedb
 $GOPATH
   src
    |--github.com
 		  |-astaxie
 			  |-beedb
    pkg
 	|--対応プラットフォーム
 		 |-github.com
 			   |--astaxie
 					|beedb.a
 # 実際のパッケージ名 ldapauth の前に github.com/xxxxx を使うことで、パッケージ名が一意であることが保証されます。コードをsrcの下にcloneします。その後go installを実行します。
 # コードの中でリモートパッケージが使用される場合、単純にローカルのパッケージと同じように頭のimportに対応するパスを添えるだけです。
 ```
 Go の開発者は伝統的に go get を使って依存パッケージをインストールしていますが、依存パッケージがアップデートされて下位互換性が失われた場合に問題が発生することがあります。Go では、下位互換性の問題を防ぐために、依存関係をロックするために-dep と mod の 2 つのツールを導入しました   
 
 `go fmt ./path/to/package`でフォーマット整形してくれる  
 golintでsyntax チェック `go vet`でもできる
 ```sh
 go get -u golang.org/x/lint/golint
 ```
 
 ここが基本的チュート良さげ
 https://github.com/astaxie/build-web-application-with-golang/blob/master/ja/01.2.md
 
 一般的にpackageの名前とディレクトリ名は一致させるべきです。
 
 ### バイナリサイズ小さくする
 go build -ldflags "-s -w"を使う。   
 upx/goupxを使ってもいいけど、起動時間が伸びる   
 https://blog.filippo.io/shrink-your-go-binaries-with-this-one-weird-trick/
 
 
 -------
 参考：https://qiita.com/mumoshu/items/0d2f2a13c6e9fc8da2a4#%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E6%9B%B8%E3%81%8D%E5%A7%8B%E3%82%81%E3%82%8B%E3%81%BE%E3%81%A7%E3%81%AB%E7%9F%A5%E3%82%8A%E3%81%9F%E3%81%8B%E3%81%A3%E3%81%9F%E3%81%93%E3%81%A8