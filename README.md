# 1 pwn で利用できるツール
## (1) objdumpによる逆アセンブル
```
objdump -D -M intel
```
* 参考になるURL：https://qiita.com/maskot1977/items/f4eddd90e4f3d8deb00f
  
## (2) gdb(peda)


## (3) pythonを用いたエクスプロイトコードの作成
### ⓪ pythonをコマンドラインで実行
```
python -c "$command"
```
また、「`\``」を利用することでコマンドに対する引数にも標準出力を送ることができる。
```
./overflow1 `python -c "print('A'*16 + '\xce\xfa\xde\xc0')"`
```

### ① 文字列の数化(オーバーフローの基本に利用できる)
pythonでは、「"a" * 20」のようにして任意の数文字列を出力できる。
```
print("a" * 20)
```
### ② 通常の文字ではないバイナリデータの送信(returnアドレスの変更やペイロードを入れたりするときに利用できる)
pythonでは、「"\x03"」とすることで任意のバイナリデータを標準出力で出すことができる。
```
print('A'*16 + '\xce\xfa\xde\xc0')
```
※ エンディアン変換による順番の変化には気を付けよう
## (4) pwntools(python)によるエクスプロイトの送信
https://qiita.com/8ayac/items/12a3523394080e56ad5a

### ◇ プロセスの獲得
```
io = process('test_program')
```

### ◇ リモートプロセスの獲得
```
io = remote('192.168.0.1', port)
```

### ◇ プロセスからのデータ受信
```
ret = p.recv() # test_programの出力をEOFまでを受けとる
ret = p.recvline() # 改行までを受けとる、改行が送られないとここで止まるので注意
ret = p.recvline(timeout=0.01) # recv系はtimeoutを設定できる、単位は秒。ハングするのが嫌なら設定しておくといい
ret = p.recvuntil('some output') # 引数の文字列までを受けとる。
```

### ◇ ログとして値を出力する
context.log_levelに値を設定することで、debug, infoなど出力するログも操作できる
```
log.info(ret) 
```

### 
## (5) checksec(実行ファイルにどんなセキュリティ機構が備わっているか確認)
### ◇ インストール
```
git clone https://github.com/slimm609/checksec.sh
```

### ◇ 実行ファイルの検査
```
./checksec --file=<file name>
```


# 2 pwnの参考になるサイト
## (1) pwnの入門
https://gist.github.com/matsubara0507/72dc50c89200a09f7c61

