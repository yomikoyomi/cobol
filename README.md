# COBOLの勉強
* COBOL
 * 導入
 * IO
 * サブルーチン
 * GO TO  
 
-----
## 2016/12/26
### 参考サイト
[COBOLプログラミング基礎 (http://tallercolibri.com/)](http://tallercolibri.com/)
### COBOL基礎
```cobol
IDENTIFICATION DIVISION.
PROGRAM-ID. sample.
ENVIRONMENT DIVISION.
INPUT-OUTPUT SECTION.
FD F1.
~~~
DATA DIVISION.
01 CNT PIC 9(04).
~~~
PROCEDURE DIVISION.
MAIN.
    ~~~
    STOP RUN.
```
#### 導入
* 部
 * 見出し部（IDENTIFICATION DIVISION）
   * PROGRAM-ID  
     →他プログラムにて参照される名前
   * 著者名
 * 環境部（ENVIRONMENT DIVISION）
   * IOで使うファイルパスなどの宣言
     * INPUT-OUTPUT SECTION.
 * データ部（DATA DIVISION)
   * 変数の宣言、初期化
     * WORKING-STORGE SECTION.
   * サブルーチン外部プログラムでのセクション
     * LINKAGE SECTION.
   * IOに関係するセクション
     * FILE SECTION.
   * 手続き部（PROCEDURE DIVISION）
     * メインロジック


* 変数定数
```cobol
DATA DIVISION.
WORKING-STORGE SECTION.
01 CNT PIC 9(04)
~~~
```
 * 定義はデータ部で行なう
 * PIC ( PICTURE ) -宣言
 * 9(02) -数値2桁(ゼロパディング)
 * X(02) -英数2桁
 * VALUE 初期化  
 `01 CNT PIC9(04) VALUE 1.`
 * 入れ子
 ```cobol
 ** こうして宣言するとメモリ上に並べて確保される
 01 DATE.
      03 HH PIC 9(02).
      03 MM PIC 9(02).
      03 SS PIC 9(02).
 ```
* 条件式
 * IF ELSE  
   * ELSE IFが無い  
   * IF ( 条件式) THEN
 * EVALUATE (switch)  
   * WHEN ( case )  
   * WHEN OTHER
   * switchと異なり、明示的に処理を終えなくても  
     WHENブロック内のみで処理が終わる
* 繰り返し
 * PERFORM  
   * WITH TEST (BEFORE/AFTER)  
   →未設定ではBEFORE  
   * VARYING ( 変数名 ) FROM ( 初期値 ) BY ( 増値 ) UNTIL (条件式)
   * PERFORM 10 TIMES  
   →10回繰り返す  
   * EXIT PERFORMでループ途中終了
* 関数
 * ( 関数名 ) SECTION.

 ```cobol
 ** 呼出
 PROCEDURE DIVISION.
     PERFORM INIT-PROC.
     STOP RUN.

 ** 宣言・処理
 INIT-PROC SECTION.
     DISPLAY "TEST".
 INIT-PROC-EXIT.
     EXIT.
 ```

#### IO
```cobol
 ENVIRONMENT DIVISION.
 INPUT-OUTPUT SECTION.
 FILE-CONTROL.
     SELECT F1 ASSIGN TO
        "out2.txt" STATUS FST.
 DATA DIVISION.
 FILE SECTION.
     FD F1.
 ```
 * INPUT  
   * OPEN INPUT ( ファイル変数名 )
 * OUTPUT  
   * OPEN OUTPUT ( ファイル変数名 )  
   →上書き
   * OPEN EXTEND ( ファイル変数名 )  
   →追記( 改行無し )
   * WRITE ( ファイル変数名 )  
   →書き込み命令  
   →AFTER ( 改行回数 )  書き込み後改行  
   `WRITE F1 AFTER 2` 2行改行する
 * STATUS
   * ファイルの読み込みに成功した  
     AND 読み込んだ行がEOMではない
       →"00"を返す

-----

#### サブルーチン
コンパイルは、メインルーチンとサブルーチンを一緒にしないとコンパイルエラーになる
`cobc -x MainLogic.cbl SubLogic.cbl`

MainLogic.cbl
```cobol
IDENTIFICATION DIVISION.
PROGRAM-ID. mainLogic.
DATA DIVISION.
WORKING-STORAGE SECTION.
01 ws-arg pic X(04).
PROCEDURE DIVISION.
MAIN.
    DISPLAY "Program Start...".
    MOVE "TEST" TO ws-arg.
    DISPLAY "Init arg: " ws-arg UPON CONSOLE.
    CALL "subLogic" USING ws-arg.
    DISPLAY "After arg: " ws-arg UPON CONSOLE.

    STOP RUN.
```

SubLogic.cbl

```cobol
IDENTIFICATION DIVISION.
PROGRAM-ID. subLogic.
DATA DIVISION.
LINKAGE SECTION.
01 ln-arg PIC X(04).
PROCEDURE DIVISION USING ln-arg.
MAIN.
    DISPLAY "SubLogic called..." UPON CONSOLE.
    DISPLAY "Get arg: " ln-arg UPON CONSOLE.
    MOVE "ESTT" TO ln-arg.

    EXIT PROGRAM.
```

Log
```
Program Start...
Init arg: TEST
SubLogic called...
Get arg: TEST
After arg: ESTT
```

#### GOTO
メソッドと異なり、指定した手続名以降を全て処理してしまう。  
セクションを作ってあげるか
サブルーチンの方が良い気がする。。。  

Goto.cbl
```COBOL
IDENTIFICATION DIVISION.
PROGRAM-ID. go-to.
DATA DIVISION.
WORKING-STORAGE SECTION.
01 CHAR PIC X(01).
01 WTIME.
    03 HH PIC 9(02).
    03 MM PIC 9(02).
    03 SS PIC 9(02).
01 CAL PIC 9(01).
PROCEDURE DIVISION.
MAIN.
    DISPLAY "SELECT GO TO POINT" UPON CONSOLE.
    DISPLAY "a) SHOW NOW DATE" UPON CONSOLE.
    DISPLAY "b) CALC 1 + 3" UPON CONSOLE.
    DISPLAY "q) QUIT THIS PROGRAM." UPON CONSOLE.

    ACCEPT CHAR FROM CONSOLE.

    EVALUATE CHAR
        WHEN "a"
            GO TO A
        WHEN "b"
            GO TO B
        WHEN "q"
            GO TO Q
        WHEN OTHER
            GO TO MAIN
    END-EVALUATE.
    GO TO Q.

A.
    ACCEPT WTIME FROM TIME.
    DISPLAY "NOW DATE :" HH ":" MM ":" SS UPON CONSOLE.
    GO TO Q.

B.
    COMPUTE CAL = 1 + 3.
    DISPLAY "1 + 3 = " CAL UPON CONSOLE.
    GO TO Q.

Q.
    STOP RUN.
```
